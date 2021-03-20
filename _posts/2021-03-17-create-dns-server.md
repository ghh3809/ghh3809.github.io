---
layout: post
title:  "如何搭建一个DNS服务器"
categories: Linux
tags:  DNS
author: 风之筝
---

* content
{:toc}

域名系统（Domain Name System），用于将指定的域名解析到其对应的IP。本文将介绍如何使用自己的服务器搭建一个DNS服务，从而实现特定域名到特定IP地址的解析。




## 1. DNS简介

### 1.1 DNS是什么

DNS，即**域名系统**（**D**omain **N**ame **S**ystem），是计算机网络中的一项基础服务。通常来说，在网络上访问某个主机有两种方式：主机名、或IP地址。我们知道在网络层，通信需要使用IP地址进行定位，因此为了能够将主机名映射为IP地址，DNS服务应运而生。

显然，DNS协议是一个应用层协议（类似于HTTP、FTP、SMPT等），其运行在UDP之上，使用53号端口。另外，DNS辅域名服务器会定时向主域名服务器查询一次数据变动情况，如果数据发生变动，会使用TCP协议进行一次区域传送，进行数据同步。

DNS从设计上像是一个**分层的**、**分布式的**数据库，这和域名分级是类似的。

### 1.2 DNS服务器类型

粗略来说，DNS服务器具有以下几种：

- **根DNS服务器**。根服务器在逻辑上共有13个（即13个IP地址），分布在世界各地。由于UDP查询和响应报文的最大长度为512字节，为了保证根服务器数据能被包含在一个UDP包中，最多只能允许13个根DNS服务器存在，其命名从A到M。
- **顶级域DNS服务器**。这些服务器负责顶级域名的DNS查询，如com、org、net等，以及国家的顶级域名。
- **权威DNS服务器**。特定的机构为了使得自己的域名可以被公共访问，需要维护对应的DNS记录。可以选择自己的权威DNS服务器以保存这些记录，也可以选择支付费用以存储在某个服务提供商的权威DNS服务器中。
- **本地DNS服务器**。对于一个ISP来说，都有一台本地DNS服务器承担代理的作用。它严格上并不属于DNS层级中。

我们本次搭建的DNS服务器，实际上就是一个本地DNS服务器，可以实现DNS代理以及简单的修改。

### 1.3 DNS通信过程

例如访问一个www.google.com的地址，DNS查询过程主要包括**递归查询**与**迭代查询**：

- 查询本地浏览器缓存
- 查询本地hosts文件
- 请求本地DNS服务器进行域名查询（至此的查询为递归查询）
- 若本地DNS服务器没有对应地址的缓存，则请求根DNS服务器，返回com顶级DNS服务器的地址。
- 请求com顶级DNS服务器，获得google.com权威DNS服务器的地址
- 请求google.com权威DNS服务器，获得www.google.com的地址（至此的查询为迭代查询）

下图展示了这一查询过程。

![](https://raw.githubusercontent.com/ghh3809/ghh3809.github.io/master/_posts/_pic/20210317_dns.png)

显然，如果每次查询都经过这样的全部过程，那耗时将是不可接受的，因此**DNS缓存**应运而生。在整个DNS网络中，当某个DNS服务器接受到一个DNS应答时，它将在本地进行缓存，从而在未来的一段时间内（例如2天）对于同样的域名进行直接应答，从而减少请求的次数。

### 1.4 DNS记录类型

DNS服务器实际存储的是一条条**资源记录**（**R**esource **R**ecord，RR），一条资源记录包括以下4个字段：Name、Value、Type和TTL。其中TTL表示该记录应从缓存中删除的时间，其余三个字段在不同的组合下拥有如下的含义：

| Type  | Name           | Value                 | 含义                                     |
| ----- | -------------- | --------------------- | ---------------------------------------- |
| A     | 主机名         | 主机名的IP地址        | 提供一条主机名到IP地址的映射（ipv4）     |
| AAAA  | 主机名         | 主机名的IP地址        | 提供一条主机名到IP地址的映射（ipv6）     |
| NS    | 域             | 该域中权威DNS的主机名 | 路由DNS查询                              |
| CNAME | 主机别名       | 规范主机名            | 提供一个主机别名到规范主机名的映射       |
| MX    | 邮件服务器别名 | 规范主机名            | 提供一个邮件服务器别名到规范主机名的映射 |

需要注意的是，CNAME和MX记录均提供的是主机名间的映射，因此邮件服务器和其他服务器允许使用相同的别名而被映射到不同的服务器。

### 1.5 DNS解析流程实例

我们以www.baidu.com这个域名为例，使用dig工具查看其DNS解析流程。

首先我们在命令行输入

```shell
dig +trace www.baidu.com
```

即可看到这个域名的解析流程。显然，首先需要获得根DNS服务器的列表与对应的IP地址（未显示）：

```
.			188369	IN	NS	k.root-servers.net.
.			188369	IN	NS	d.root-servers.net.
.			188369	IN	NS	b.root-servers.net.
.			188369	IN	NS	f.root-servers.net.
.			188369	IN	NS	h.root-servers.net.
.			188369	IN	NS	a.root-servers.net.
.			188369	IN	NS	e.root-servers.net.
.			188369	IN	NS	j.root-servers.net.
.			188369	IN	NS	l.root-servers.net.
.			188369	IN	NS	g.root-servers.net.
.			188369	IN	NS	m.root-servers.net.
.			188369	IN	NS	c.root-servers.net.
.			188369	IN	NS	i.root-servers.net.
.			188369	IN	RRSIG	NS 8 0 518400 20210329050000 20210316040000 42351 . m9yFSVKnYvynh24y4hOwzmqFr5OjJyekefJ4VdFD/HqDWvyR9O1ckk/b B/WI8Y7Wm0wilTHIccplVJgciG9Uqy2uuEhbGQ/mTmGK85mvx5ZzN7VN RJ4FkwBORC9c3bqjW0vEegbMs4ClzdhDALKM7GbPbolECEqJ0lJWXvVn IJi2KTbvFdYXac63s4Tdy/bQBJS9BM93m4OTeHTTuRnQvL63fRmRnnJQ F2S3JZuYBlE/jKMLLCrV2NE0O4VN0EwjHFginl51AeA5NvIL4jriJdGf xNsntD13R6g5jjgTqQaUZlbeJO/iT1m7XtQXlV9Rh0Xvu1NofxpnXqsJ wtpUIw==
;; Received 1097 bytes from 172.22.1.253#53(172.22.1.253) in 11 ms
```

列表中第一列`.`表示所在域（即记录中的Name），第二列`188369`表示TTL（从响应可以看到，根DNS服务器列表是从局域网地址得到的），第三列`NS`表示记录类型，第四列`x.root-servers.net`即为根DNS服务器的域名。需要注意的是，这里所有的记录都是NS类型的，表示如果想要解析`.`域下的域名，需要请求NS类型中的地址作为其DNS服务器。

接下来，选择一个根DNS服务器（可以看到，选择的是`e.root-servers.net`），获得com顶级域DNS服务器的列表：

```
com.			172800	IN	NS	l.gtld-servers.net.
com.			172800	IN	NS	b.gtld-servers.net.
com.			172800	IN	NS	c.gtld-servers.net.
com.			172800	IN	NS	d.gtld-servers.net.
com.			172800	IN	NS	e.gtld-servers.net.
com.			172800	IN	NS	f.gtld-servers.net.
com.			172800	IN	NS	g.gtld-servers.net.
com.			172800	IN	NS	a.gtld-servers.net.
com.			172800	IN	NS	h.gtld-servers.net.
com.			172800	IN	NS	i.gtld-servers.net.
com.			172800	IN	NS	j.gtld-servers.net.
com.			172800	IN	NS	k.gtld-servers.net.
com.			172800	IN	NS	m.gtld-servers.net.
com.			86400	IN	DS	30909 8 2 E2D3C916F6DEEAC73294E8268FB5885044A833FC5459588F4A9184CF C41A5766
com.			86400	IN	RRSIG	DS 8 1 86400 20210402050000 20210320040000 42351 . opDhdYLhssLRtID78PyEbBW3lQpjv98bF2mK7K9f7otbEYU61LBWXBAa +78W7QWWNun4iWS2O6SnTdr5H1I3HEsRWLascNd5W8OZfNUaqDrWX+qn 9jb3a8Q2SMC420Rccg2bXYm+2T/J3z7VD4kOu4jwHixjryDoCmuarQ6W 7u8A0P0NFAKlFxhd0/LsXItuL8VhCyPfUFxjHQIa7/9AZgjVyF5xapu9 v1LVmETSEVaKzJfzK6/RTPOKo27KYbJbECqvoeYTqNqGL0GFJZC/RDgj uQtCy0GPGlxufbtwuIRptmpRX3qrTjEvzl0SuXNE+yYrLb+Upq2pZeI2 GzTsqg==
;; Received 1173 bytes from 192.203.230.10#53(e.root-servers.net) in 307 ms
```

同理得到baidu.com权威DNS服务器的列表：

```
baidu.com.		172800	IN	NS	ns2.baidu.com.
baidu.com.		172800	IN	NS	ns3.baidu.com.
baidu.com.		172800	IN	NS	ns4.baidu.com.
baidu.com.		172800	IN	NS	ns1.baidu.com.
baidu.com.		172800	IN	NS	ns7.baidu.com.
CK0POJMG874LJREF7EFN8430QVIT8BSM.com. 86400 IN NSEC3 1 1 0 - CK0Q1GIN43N1ARRC9OSM6QPQR81H5M9A  NS SOA RRSIG DNSKEY NSEC3PARAM
CK0POJMG874LJREF7EFN8430QVIT8BSM.com. 86400 IN RRSIG NSEC3 8 2 86400 20210325044039 20210318033039 58540 com. yb26/u3Pyd12e3pkIGjKKoWs+8gYcVu+g4TJYw1igY233mZvg8jK4g7M nhlu8Yr316TB5OyztbH2KGai+kF/chRQ5uVuxA+zJrGZt/bwNH2TswiS TxIoYxl1deMFEmP8CI3SF7G8uIRaWjvt9o+hKWLCcv7N3sLyx2/SHGRP DKsCJAqeVB4+CXMqfUbfSJlbhwpNjR+LpJW38BC0DylM6Q==
HPVUSBDNI26UDNIV6R0SV14GC3KGR4JP.com. 86400 IN NSEC3 1 1 0 - HPVVN3Q5E5GOQP2QFE2LEM4SVB9C0SJ6  NS DS RRSIG
HPVUSBDNI26UDNIV6R0SV14GC3KGR4JP.com. 86400 IN RRSIG NSEC3 8 2 86400 20210325061418 20210318050418 58540 com. lPJtjsCBLl5exfZaZUepnMDdIABxEWCFogkQ7nN0OO4T0UooHdkbBuco PcZAN/Om/bflq8ncHhl0H/I3EkjVuk81Fh1zGIqbIlt/xCo9aevjNujm +katY/vG4EwPd51cBgVsln7qGo4IqBZIJdabf+fsqQ1fRFOO5F7jm/Q/ JQ2EnhUEgrkyp4G9/1G5AjNdQbg1DEvHpfYopdbetwVdFA==
;; Received 761 bytes from 192.48.79.30#53(j.gtld-servers.net) in 293 ms
```

可以发现baidu.com域下有5个域名服务器（`nsx.baidu.com`），最终我们选择一个域名服务器，解析我们最终需要的域名www.baidu.com：

```
www.baidu.com.		1200	IN	CNAME	www.a.shifen.com.
a.shifen.com.		1200	IN	NS	ns4.a.shifen.com.
a.shifen.com.		1200	IN	NS	ns1.a.shifen.com.
a.shifen.com.		1200	IN	NS	ns3.a.shifen.com.
a.shifen.com.		1200	IN	NS	ns5.a.shifen.com.
a.shifen.com.		1200	IN	NS	ns2.a.shifen.com.
;; Received 239 bytes from 112.80.248.64#53(ns3.baidu.com) in 31 ms
```

这里出现了一个CNAME记录，可见www.baidu.com实际是www.a.shifen.com的别名。因此我们需要重新对www.a.shifen.com这个域名进行解析。重复的流程不再赘述，最终可以得到这个域名的解析结果：

```
www.a.shifen.com.	300	IN	A	110.242.68.4
www.a.shifen.com.	300	IN	A	110.242.68.3
a.shifen.com.		1200	IN	NS	ns1.a.shifen.com.
a.shifen.com.		1200	IN	NS	ns2.a.shifen.com.
a.shifen.com.		1200	IN	NS	ns3.a.shifen.com.
a.shifen.com.		1200	IN	NS	ns4.a.shifen.com.
a.shifen.com.		1200	IN	NS	ns5.a.shifen.com.
;; Received 236 bytes from 220.181.33.32#53(ns2.a.shifen.com) in 10 ms
```

终于出现了我们需要的A记录，我们也最终获得了www.baidu.com的IP地址：`110.242.68.3`、`110.242.68.4`。

## 2. 搭建DNS服务

从上述域名解析过程中可以看到，一个本地DNS服务器的作用就是迭代查询DNS网络中的各个服务器，从而获得域名解析结果。因此，如果我们需要在局域网内实现指定域名到IP地址的映射，就可以通过搭建DNS服务器来实现。

### 2.1 准备工作

为了能够顺利搭建DNS服务，我们选择了阿里云的一台CentOS系统的服务器。

```
[root@izbp1hainir6k56v2jjzhmz ~]# cat /proc/version
Linux version 3.10.0-1127.13.1.el7.x86_64 (mockbuild@kbuilder.bsys.centos.org) (gcc version 4.8.5 20150623 (Red Hat 4.8.5-39) (GCC) ) #1 SMP Tue Jun 23 15:46:38 UTC 2020
```

目前最常用的域名服务软件是**BIND**（**B**erkeley **I**nternet **N**ame **D**omain），最早由一名伯克利大学的学生编写，目前最新的版本是9，由**ISC**（**I**nternet **S**ystems **C**onsortium）负责编写和维护。

BIND上DNS服务对应的服务名为`named`，运行在53端口。

首先我们通过yum安装BIND：

```shell
yum -y install bind bind-chroot bind-utils
```

安装完成后，可以看到以下两个目录生成了对应的文件：

```shell
/etc/named.conf # BIND服务的配置文件
/var/named/     # DNS解析需要的zone文件列表
```

### 2.2 BIND配置文件解析

#### 2.2.1 option配置

我们打开`/etc/named.conf`配置文件，可以看到第一部分为配置（option）信息：

```shell
options {
        listen-on port 53 { 127.0.0.1; };
        listen-on-v6 port 53 { ::1; };
        directory       "/var/named"; # zone文件所在目录
        dump-file       "/var/named/data/cache_dump.db";
        statistics-file "/var/named/data/named_stats.txt";
        memstatistics-file "/var/named/data/named_mem_stats.txt";
        recursing-file  "/var/named/data/named.recursing";
        secroots-file   "/var/named/data/named.secroots";
        allow-query     { localhost; }; # 仅允许localhost访问DNS服务

        /*
         - If you are building an AUTHORITATIVE DNS server, do NOT enable recursion.
         - If you are building a RECURSIVE (caching) DNS server, you need to enable
           recursion.
         - If your recursive DNS server has a public IP address, you MUST enable access
           control to limit queries to your legitimate users. Failing to do so will
           cause your server to become part of large scale DNS amplification
           attacks. Implementing BCP38 within your network would greatly
           reduce such attack surface
        */
        recursion yes;

        dnssec-enable yes;
        dnssec-validation yes;

        /* Path to ISC DLV key */
        bindkeys-file "/etc/named.root.key";

        managed-keys-directory "/var/named/dynamic";

        pid-file "/run/named/named.pid";
        session-keyfile "/run/named/session.key";
};
```

摘一段重点字段说明：

```
· <allow-query> — Specifies which hosts are allowed to query this nameserver. By default, all hosts are allowed to query. An access control list, or collection of IP addresses or networks may be used here to only allow particular hosts to query the nameserver.

· <allow-recursion> — Similar to allow-query, this option applies to recursive queries. By default, all hosts are allowed to perform recursive queries on the nameserver.

· <directory> — Specifies the named working directory if different from the default value, /var/named/.

· <forward> — Specifies the forwarding behavior of a forwarders directive.

· <forwarders> — Specifies a list of valid IP addresses for nameservers where requests should be forwarded for resolution.

· <listen-on> — Specifies the network interface on which named listens for queries. By default, all interfaces are used.

· <notify> — Controls whether named notifies the slave servers when a zone is updated. It accepts the following options
```

#### 2.2.2 根区域信息

第二部分为根区域（zone）信息，表示根域名`.`的配置文件。

```shell
zone "." IN {
        type hint;
        file "named.ca";
};
```

我们打开`/var/named/named.ca`，可以看到：

```
; <<>> DiG 9.11.3-RedHat-9.11.3-3.fc27 <<>> +bufsize=1200 +norec @a.root-servers.net
; (2 servers found)
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 46900
;; flags: qr aa; QUERY: 1, ANSWER: 13, AUTHORITY: 0, ADDITIONAL: 27

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 1472
;; QUESTION SECTION:
;.                              IN      NS

;; ANSWER SECTION:
.                       518400  IN      NS      a.root-servers.net.
.                       518400  IN      NS      b.root-servers.net.
.                       518400  IN      NS      c.root-servers.net.
.                       518400  IN      NS      d.root-servers.net.
.                       518400  IN      NS      e.root-servers.net.
.                       518400  IN      NS      f.root-servers.net.
.                       518400  IN      NS      g.root-servers.net.
.                       518400  IN      NS      h.root-servers.net.
.                       518400  IN      NS      i.root-servers.net.
.                       518400  IN      NS      j.root-servers.net.
.                       518400  IN      NS      k.root-servers.net.
.                       518400  IN      NS      l.root-servers.net.
.                       518400  IN      NS      m.root-servers.net.

;; ADDITIONAL SECTION:
a.root-servers.net.     518400  IN      A       198.41.0.4
b.root-servers.net.     518400  IN      A       199.9.14.201
c.root-servers.net.     518400  IN      A       192.33.4.12
d.root-servers.net.     518400  IN      A       199.7.91.13
e.root-servers.net.     518400  IN      A       192.203.230.10
f.root-servers.net.     518400  IN      A       192.5.5.241
g.root-servers.net.     518400  IN      A       192.112.36.4
h.root-servers.net.     518400  IN      A       198.97.190.53
i.root-servers.net.     518400  IN      A       192.36.148.17
j.root-servers.net.     518400  IN      A       192.58.128.30
k.root-servers.net.     518400  IN      A       193.0.14.129
l.root-servers.net.     518400  IN      A       199.7.83.42
m.root-servers.net.     518400  IN      A       202.12.27.33
a.root-servers.net.     518400  IN      AAAA    2001:503:ba3e::2:30
b.root-servers.net.     518400  IN      AAAA    2001:500:200::b
c.root-servers.net.     518400  IN      AAAA    2001:500:2::c
d.root-servers.net.     518400  IN      AAAA    2001:500:2d::d
e.root-servers.net.     518400  IN      AAAA    2001:500:a8::e
f.root-servers.net.     518400  IN      AAAA    2001:500:2f::f
g.root-servers.net.     518400  IN      AAAA    2001:500:12::d0d
h.root-servers.net.     518400  IN      AAAA    2001:500:1::53
i.root-servers.net.     518400  IN      AAAA    2001:7fe::53
j.root-servers.net.     518400  IN      AAAA    2001:503:c27::2:30
k.root-servers.net.     518400  IN      AAAA    2001:7fd::1
l.root-servers.net.     518400  IN      AAAA    2001:500:9f::42
m.root-servers.net.     518400  IN      AAAA    2001:dc3::35

;; Query time: 24 msec
;; SERVER: 198.41.0.4#53(198.41.0.4)
;; WHEN: Thu Apr 05 15:57:34 CEST 2018
;; MSG SIZE  rcvd: 811
```

`ANSWER SECTION`是否觉得很熟悉？没错，就是我们在1.5中得到的根DNS服务器的结果。除此之外，`ADDITIONAL SECTION`部分还对这13个服务器的ipv4地址与ipv6地址进行了配置，这也就是`dig`命令首次请求根域名服务器的结果。

#### 2.2.3 辅助区域文件配置

回过头看到`/etc/named.conf`的第三部分，该部分是辅助区域文件的详细配置。

```shell
include "/etc/named.rfc1912.zones";
include "/etc/named.root.key";
```

在DNS网络中，每个DNS服务器能够直接解析的域名，实际上都存储在**区域文件**中。换句话说，区域文件中的记录越多，该DNS服务器能够回答的**权威应答**也就越多。

我们打开`/etc/named.rfc1912.zones`，查看各区域信息：

```shell
zone "localhost.localdomain" IN {
        type master;
        file "named.localhost";
        allow-update { none; };
};

zone "localhost" IN {
        type master;
        file "named.localhost";
        allow-update { none; };
};

zone "1.0.0.0.0.0.0.0.0.0.0.0.0.0.0.0.0.0.0.0.0.0.0.0.0.0.0.0.0.0.0.0.ip6.arpa" IN {
        type master;
        file "named.loopback";
        allow-update { none; };
};

zone "1.0.0.127.in-addr.arpa" IN {
        type master;
        file "named.loopback";
        allow-update { none; };
};

zone "0.in-addr.arpa" IN {
        type master;
        file "named.empty";
        allow-update { none; };
};
```

该文件为我们默认配置了5个区域。以第一条为例，其各部分的含义是：

- `localhost.localdomain`：该区域的名称。如果区域名称以arpa结尾，表示该zone进行反向解析，即通过ip地址获得域名信息。我们搭建DNS服务器是暂不需要关心反向解析，仅考虑正向解析。

- `type master`：表示这是主DNS。我们在`/etc/named.conf`中见到的`.`域使用的是`type hint`，表示根DNS。同样的还有`type slave`表示从DNS。

- `file "named.localhost"`：表示该zone的配置文件位于`/var/named/named.localhost`中。

- `allow-update { none; };`：是否允许客户端动态更新，none表明不允许。

#### 2.2.4 区域文件记录

最后我们打开`/var/named/named.localhost`，查看该区域的配置：

```shell
$TTL 1D
@       IN SOA  @ rname.invalid. (
                                        0       ; serial
                                        1D      ; refresh
                                        1H      ; retry
                                        1W      ; expire
                                        3H )    ; minimum
        NS      @
        A       127.0.0.1
        AAAA    ::1
```

实际上这里就是该区域的所有资源记录（RR）。这里有一条记录之前没有提到过，就是SOA记录。这种记录是所有区域性文件中的强制性记录，同时它也必须是一个文件中的第一个记录。它的具体格式为：

- `@`：表示区域的根。@表示默认的$ORIGIN。
- `IN SOA`：IN表示互联网，SOA标记记录类型。
- `@`：第二个@表示该域名的主域名服务器，同样这里是默认的$ORIGIN。
- `rname.invalid`：表示区域文件管理员的邮箱地址，这里表示该值不可用。
- `0 ; serial`：区域文件的序列号
- `1D ; refresh`：区域的刷新间隔。这是从服务器向主服务器轮询检查区域文件是否变更之间等待的时间量。
- `1H ; retry`：区域的重试间隔。如果从机在刷新周期结束时无法连接到主机，则它将等待此时间并重试轮询主机。
- `1W ; expire`：到期时间。如果从服务器在此时间内无法与主服务器联系，则它不再作为此区域的权威来源的返回响应。
- `3H ) ; minimum`：名称服务器在此文件中找不到所请求的名称时缓存找不到结果的时间量。

后续的几条记录含义显而易见，在此不再赘述。

### 2.3 搭建自己的DNS服务

我们实现这样的目标：

1、搭建一个DNS服务器，接收所有的DNS请求（ipv4）；

2、将所有的DNS请求转发到`8.8.8.8`进行解析；

3、针对指定域名`mydomain.com`，解析到`localhost`。

为实现转发功能，我们需要在`/etc/named.conf`中做出如下修改：

```shell
options {
    # 监听来自于所有打到53端口的请求
    listen-on port 53 { any; };
    # 允许来自任意host的DNS查询
    allow-query     { any; };
    # 转发逻辑为：服务器将只会请求 forwarders中的DNS主机，请求失败时，将直接应答fail
    # 由于8.8.8.8是一个几乎完全保证可用的DNS，这里forward only和forward first没有功能上的影响
    # 如果不将自己的服务器作为转发服务器，则无需配置forward和forwarders，此时所有的解析将按照之前的迭代查询方式进行查询
    forward only;
    # 转发服务器列表：8.8.8.8
    forwarders  { 8.8.8.8; };
}
```

接下来，我们在辅助区域配置文件`/etc/named.rfc1912.zones`中，添加一条我们自己创建的区域：

```shell
zone "mydomain.com" IN {
        type master;
        file "mydomain.com.zone";
        allow-update { none; };
};
```

最后，我们在`/var/named/`目录下，创建区域文件记录`mydomain.com.zone`：

```shell
$TTL 1D
@       IN SOA  @ rname.invalid. (
                                        0       ; serial
                                        1D      ; refresh
                                        1H      ; retry
                                        1W      ; expire
                                        3H )    ; minimum
        NS      @
        A       ${服务器公网IP}
```

最后，我们启动`named`服务：

```shell
systemctl start named
```

由于我们使用的是阿里云的服务器，因此还需要在控制台开启53端口的权限（注意DNS服务使用的是UDP协议哦）。

![](https://raw.githubusercontent.com/ghh3809/ghh3809.github.io/master/_posts/_pic/20210317_aliyun.png)

最后我们尝试下外部访问：

```shell
# ping mydomain.com
PING mydomain.com (${服务器公网IP}): 56 data bytes
64 bytes from ${服务器公网IP}: icmp_seq=0 ttl=50 time=91.537 ms
```

到此，我们的DNS服务配置成功。相关的日志记录在`/var/named/data/named.run`中。

### 2.4 使用从DNS服务器

有时DNS服务器需要使用多副本的方式，BIND也提供了主-从服务器的机制方便多个服务器之间进行同步。

为了配置从服务器，我们使用如下定义：

```shell
zone "mydomain.com" {
    type slave; # 表明这是个从服务器
    masters { ${主服务器IP}; }; # 配置主服务器的IP
    file "slaves/example.net.zone"; # 文件存储位置
}
```

之后再重启服务时，它就可以自动从主服务器同步`example.net.zone`配置。

