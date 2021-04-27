---
layout: post
title:  "【海鸟阁】抹布卡池维护指南"
categories: 海鸟阁
tags:  海鸟阁 抹布
author: 风之筝
---

* content
{:toc}

本文用于介绍如何对抹布抽卡的卡池进行维护，方便群友进行卡池同步维护及群聊娱乐。




## 1. 准备工作

你需要准备：
- 一台能上网的电脑
- 文本文档/word/随便什么能打字的软件
- 如果要新增角色/武器，对应的图片
- 了解需要维护的卡池信息

好了，现在你已经完成了所有的准备工作，现在跟着步骤一起来编写维护卡池所需要的SQL吧！

## 2. 维护步骤

整个维护过程包括**卡池信息**、**新增角色/武器信息**、**卡池内容**三部分，每部分需要提供一条或多条SQL语句，下面将一一介绍如果编写。一个编写示例可以参照3.2节：公子复刻池。

### 2.1 卡池信息

新增一个卡池时，你需要写下如下的一条SQL语句：

```
insert into kotoumi_genshin_wish_event values ({卡池ID}, '{卡池名称}', '{五星说明}', '{四星说明}', {卡池类型}, '{开始时间}', '{结束时间}');
```

例如：

```sql
insert into kotoumi_genshin_wish_event values (105, '角色up', 'up角色：达达利亚，非up角色：常驻池五星角色', 'up角色：罗莎莉亚、芭芭拉、菲谢尔，非up角色：常驻池四星', 3, '2021-04-06 18:00:00', '2021-04-27 14:59:59');
```

各字段对应的含义：

| 字段名称 | 说明                                                         | 示例                                                   |
| -------- | ------------------------------------------------------------ | ------------------------------------------------------ |
| 卡池ID   | 当前卡池的唯一ID，**不可重复**。一般来说角色up池从100开始，武器up池从200开始，自定义池子从300开始。当前已存在序号可参考[当前卡池信息](https://github.com/ghh3809/kotoumi/blob/main/script/genshin_data.sql)或3.1节内容。**这个字段很重要，后续还会用到** | 105<br>204                                             |
| 卡池名称 | 卡池的名称，也就是群里抽卡时“10连xx池”中的**xx**，注意结尾不能加“池”字 | 角色up<br>武器up                                       |
| 五星说明 | 给卡池五星的说明，在群里回复“当前卡池”时呈现                 | up角色：达达利亚，非up角色：常驻池五星角色             |
| 四星说明 | 给卡池四星的说明，在群里回复“当前卡池”时呈现                 | up角色：罗莎莉亚、芭芭拉、菲谢尔，非up角色：常驻池四星 |
| 卡池类型 | 卡池的类型，目前一共3种：<br>1：表示常驻池，90抽保底，无任何up，所有角色/武器共享概率<br>2：表示武器up池，80抽保底，有up且有大保底机制<br>3：表示角色up池，90抽保底，有up且有大保底机制<br>卡池类型不限制up数量的多少 | 1<br>2<br>3                                            |
| 开始时间 | 卡池的开始时间，格式为yyyy-MM-dd HH:mm:ss                    | 2021-04-06 18:00:00                                    |
| 结束时间 | 卡池的结束时间，格式为yyyy-MM-dd HH:mm:ss                    | 2021-04-27 14:59:59                                    |

### 2.2 新增角色/武器信息

如果该卡池中的角色/武器之前未曾出现过，则需要新增对应的对象信息，此时你需要编写如下的一些SQL语句：

```
insert into kotoumi_genshin_unit values ({对象ID}, {对象类型}, '{对象名称}', {对象星级});
```

例如：

```sql
insert into kotoumi_genshin_unit values (1030, 1, '罗莎莉亚', 4);
```

各字段对应的含义：

| 字段名称 | 说明                                                         | 示例                 |
| -------- | ------------------------------------------------------------ | -------------------- |
| 对象ID   | 该角色/武器的唯一ID，**不可重复**。一般来说角色从1000开始，武器从2000开始，自定义对象从3000开始。当前已存在序号可参考[当前卡池信息](https://github.com/ghh3809/kotoumi/blob/main/script/genshin_data.sql)或3.1节内容。**这个字段很重要，后续还会用到** | 1030<br>2059         |
| 对象类型 | 表示是一个角色还是武器。目前1表示角色，2表示武器，3表示群友<br>如果需要增加类型，可以随时和风筝沟通。 | 1<br>2               |
| 对象名称 | 该角色/武器的名称                                            | 罗莎莉亚<br>暗巷闪光 |
| 对象星级 | 对象稀有度                                                   | 3<br>4<br>5          |

另外，在添加4星以上角色/武器时，**<font color='red'>还需要提供一个头像</font>**，该头像的命名方式应为`{对象ID}.png`，例如`1030.png`，大小最好不要超过50KB，以保证抽卡时不会卡住。

### 2.3 添加卡池内容

卡池和角色/武器准备好后，需要往卡池中添加对应的角色/武器，这里以角色up池为例，建议按照以下顺序添加，避免遗漏：

- up五星角色
- 非up五星角色
- up四星角色
- 非up四星角色
- 四星武器
- 三星武器

每添加一项角色/武器，均需要编写如下的SQL语句：

```
insert into kotoumi_genshin_wish_unit_map values (null, {卡池ID}, {对象ID}, {是否up});
```

例如：

```sql
insert into kotoumi_genshin_wish_unit_map values (null, 105, 1022, 1);
```

表示往公子up池中，添加了公子这一up角色。各字段含义如下：

| 字段名称 | 说明                                                         | 示例         |
| -------- | ------------------------------------------------------------ | ------------ |
| 卡池ID   | 2.1中添加的卡池ID，表示要往哪个池子中添加内容                | 105<br>204   |
| 对象ID   | 2.2中添加的对象ID，表示添加的是哪个角色/武器                 | 1022<br>2030 |
| 是否up   | 是否为up，1为up，0为非up<br>允许多up的情况出现，例如武器池会有两个up五星武器、五个up四星武器 | 1<br>0       |

至此，卡池信息维护完毕，将2.1~2.3中写出来的SQL交给风筝录入即可^_^

可以参考3.2节的公子复刻池示例进行维护。

## 3. 其他说明

### 3.1 当前历史卡池维护信息

若要查看抹布之前的维护历史，可以点击如下github链接查看：

[genshin_data.sql](https://github.com/ghh3809/kotoumi/blob/main/script/genshin_data.sql)

为防止部分同学打不开github，下面把一些信息贴出来。

### 3.1.1 卡池信息

| 卡池ID | 卡池名称 | 五星简介 | 四星简介 | 卡池类型 | 开始时间 | 结束时间 |
| ------ | -------- | -------- | -------- | -------- | -------- | -------- |
| 1 | 角色（已废弃） | 截止1.2版本所有上场过的5星角色 | 截止1.2版本所有上场过的4星角色 | 1 | 2021-01-01 00:00:00 | 2021-02-02 23:59:59 |
| 2 | 武器（已废弃） | 截止1.2版本所有上场过的5星武器 | 截止1.2版本所有上场过的4星武器 | 1 | 2021-01-01 00:00:00 | 2021-02-02 23:59:59 |
| 3 | 标准（已废弃） | 截止1.2版本所有上场过的5星角色+5星武器 | 截止1.2版本所有上场过的4星角色+4星武器 | 1 | 2021-01-01 00:00:00 | 2021-02-02 23:59:59 |
| 4 | 常驻 | 常驻池5星角色+武器 | 常驻池4星角色+武器 | 1 | 2021-02-03 00:00:00 | 2025-12-31 23:59:59 |
| 101 | 角色up-魈 | up角色：魈，非up角色：常驻池五星角色 | up角色：迪奥娜、北斗、辛焱，非up角色：常驻池四星 | 3 | 2021-02-03 10:00:00 | 2021-02-17 15:59:59 |
| 102 | 角色up-刻晴 | up角色：刻晴，非up角色：常驻池五星角色 | up角色：凝光、班尼特、芭芭拉，非up角色：常驻池四星 | 3 | 2021-02-17 18:00:00 | 2021-03-02 15:59:59 |
| 103 | 角色up-胡桃 | up角色：胡桃，非up角色：常驻池五星角色 | up角色：香菱、重云、行秋，非up角色：常驻池四星 | 3 | 2021-03-02 18:00:00 | 2021-03-16 14:59:59 |
| 104 | 角色up-温迪 | up角色：温迪，非up角色：常驻池五星角色 | up角色：砂糖、雷泽、诺艾尔，非up角色：常驻池四星 | 3 | 2021-03-17 11:00:00 | 2021-04-06 15:59:59 |
| 105 | 角色up-公子 | up角色：达达利亚，非up角色：常驻池五星角色 | up角色：罗莎莉亚、芭芭拉、菲谢尔，非up角色：常驻池四星 | 3 | 2021-04-06 18:00:00 | 2021-04-27 14:59:59 |
| 201 | 武器up-磐岩结绿&和璞鸢 | up武器：磐岩结绿、和璞鸢，非up武器：常驻池五星武器 | up武器：笛剑、祭礼大剑、弓藏、昭心、西风长枪，非up武器：常驻池四星 | 2 | 2021-02-03 10:00:00 | 2021-02-23 15:59:59 |
| 202 | 武器up-狼的末路&护摩之杖 | up武器：狼的末路、护摩之杖，非up武器：常驻池五星武器 | up武器：千岩古剑、千岩长枪、匣里龙吟、祭礼弓、流浪乐章，非up武器：常驻池四星 | 2 | 2021-02-23 10:00:00 | 2021-03-16 14:59:59 |
| 203 | 武器up-终末嗟叹之诗&天空之刃 | up武器：终末嗟叹之诗、天空之刃，非up武器：常驻池五星武器 | up武器：暗巷闪光、暗巷的酒与诗、西风大剑、西风猎弓、匣里灭辰，非up武器：常驻池四星 | 2 | 2021-03-17 11:00:00 | 2021-04-06 15:59:59 |
| 204 | 武器up-天空之翼&四风原典 | up武器：天空之翼、四风原典，非up武器：常驻池五星武器 | up武器：暗巷猎手、西风剑、祭礼大剑、西风秘典、西风长枪，非up武器：常驻池四星 | 2 | 2021-04-06 18:00:00 | 2021-04-27 14:59:59 |

### 3.1.2 当前角色信息

| 角色ID | 角色类型 | 角色名称 | 角色稀有度 |
| ------ | -------- | -------- | ---------- |
| 1001 | 1 | 安柏 | 4 |
| 1002 | 1 | 凯亚 | 4 |
| 1003 | 1 | 丽莎 | 4 |
| 1004 | 1 | 琴 | 5 |
| 1005 | 1 | 可莉 | 5 |
| 1006 | 1 | 诺艾尔 | 4 |
| 1007 | 1 | 芭芭拉 | 4 |
| 1008 | 1 | 温迪 | 5 |
| 1009 | 1 | 雷泽 | 4 |
| 1010 | 1 | 迪卢克 | 5 |
| 1011 | 1 | 班尼特 | 4 |
| 1012 | 1 | 菲谢尔 | 4 |
| 1013 | 1 | 北斗 | 4 |
| 1014 | 1 | 凝光 | 4 |
| 1015 | 1 | 香菱 | 4 |
| 1016 | 1 | 行秋 | 4 |
| 1017 | 1 | 重云 | 4 |
| 1018 | 1 | 砂糖 | 4 |
| 1019 | 1 | 莫娜 | 5 |
| 1020 | 1 | 刻晴 | 5 |
| 1021 | 1 | 七七 | 5 |
| 1022 | 1 | 达达利亚 | 5 |
| 1023 | 1 | 迪奥娜 | 4 |
| 1024 | 1 | 钟离 | 5 |
| 1025 | 1 | 辛焱 | 4 |
| 1026 | 1 | 阿贝多 | 5 |
| 1027 | 1 | 甘雨 | 5 |
| 1028 | 1 | 魈 | 5 |
| 1029 | 1 | 胡桃 | 5 |
| 1030 | 1 | 罗莎莉亚 | 4 |

### 3.1.3 当前武器信息

| 武器ID | 武器类型 | 武器名称 | 武器稀有度 |
| ------ | -------- | -------- | ---------- |
| 2001 | 2 | 飞天御剑 | 3 |
| 2002 | 2 | 黎明神剑 | 3 |
| 2003 | 2 | 冷刃 | 3 |
| 2004 | 2 | 笛剑 | 4 |
| 2005 | 2 | 祭礼剑 | 4 |
| 2006 | 2 | 黑岩长剑 | 4 |
| 2007 | 2 | 西风剑 | 4 |
| 2008 | 2 | 匣里龙吟 | 4 |
| 2009 | 2 | 风鹰剑 | 5 |
| 2010 | 2 | 天空之刃 | 5 |
| 2011 | 2 | 斫峰之刃 | 5 |
| 2012 | 2 | 沐浴龙血的剑 | 3 |
| 2013 | 2 | 以理服人 | 3 |
| 2014 | 2 | 铁影阔剑 | 3 |
| 2015 | 2 | 祭礼大剑 | 4 |
| 2016 | 2 | 雨裁 | 4 |
| 2017 | 2 | 钟剑 | 4 |
| 2018 | 2 | 西风大剑 | 4 |
| 2019 | 2 | 黑岩斩刀 | 4 |
| 2020 | 2 | 狼的末路 | 5 |
| 2021 | 2 | 天空之傲 | 5 |
| 2022 | 2 | 无工之剑 | 5 |
| 2023 | 2 | 神射手之誓 | 3 |
| 2024 | 2 | 鸦羽弓 | 3 |
| 2025 | 2 | 弹弓 | 3 |
| 2026 | 2 | 西风猎弓 | 4 |
| 2027 | 2 | 弓藏 | 4 |
| 2028 | 2 | 祭礼弓 | 4 |
| 2029 | 2 | 绝弦 | 4 |
| 2030 | 2 | 黑岩战弓 | 4 |
| 2031 | 2 | 阿莫斯之弓 | 5 |
| 2032 | 2 | 天空之翼 | 5 |
| 2033 | 2 | 黑缨枪 | 3 |
| 2034 | 2 | 匣里灭辰 | 4 |
| 2035 | 2 | 黑岩刺枪 | 4 |
| 2036 | 2 | 西风长枪 | 4 |
| 2037 | 2 | 天空之脊 | 5 |
| 2038 | 2 | 和璞鸢 | 5 |
| 2039 | 2 | 贯虹之槊 | 5 |
| 2040 | 2 | 魔导绪论 | 3 |
| 2041 | 2 | 讨龙英杰谭 | 3 |
| 2042 | 2 | 翡玉法球 | 3 |
| 2043 | 2 | 祭礼残章 | 4 |
| 2044 | 2 | 流浪乐章 | 4 |
| 2045 | 2 | 西风秘典 | 4 |
| 2046 | 2 | 宗室秘法录 | 4 |
| 2047 | 2 | 黑岩绯玉 | 4 |
| 2048 | 2 | 昭心 | 4 |
| 2049 | 2 | 四风原典 | 5 |
| 2050 | 2 | 天空之卷 | 5 |
| 2051 | 2 | 尘世之锁 | 5 |
| 2052 | 2 | 磐岩结绿 | 5 |
| 2053 | 2 | 护摩之杖 | 5 |
| 2054 | 2 | 千岩古剑 | 4 |
| 2055 | 2 | 千岩长枪 | 4 |
| 2056 | 2 | 终末嗟叹之诗 | 5 |
| 2057 | 2 | 暗巷闪光 | 4 |
| 2058 | 2 | 暗巷的酒与诗 | 4 |
| 2059 | 2 | 暗巷猎手 | 4 |

### 3.2 一次维护示例

#### 3.2.1 公子复刻池

```sql
-- 公子复刻池卡池信息
insert into kotoumi_genshin_wish_event values (105, '角色up', 'up角色：达达利亚，非up角色：常驻池五星角色', 'up角色：罗莎莉亚、芭芭拉、菲谢尔，非up角色：常驻池四星', 3, '2021-04-06 18:00:00', '2021-04-27 14:59:59');

-- 公子复刻池添加的罗莎莉亚新角色
insert into kotoumi_genshin_unit values (1030, 1, '罗莎莉亚', 4);

-- up五星角色：公子
insert into kotoumi_genshin_wish_unit_map values (null, 105, 1022, 1);

-- 非up五星角色：5个常驻五星
insert into kotoumi_genshin_wish_unit_map values (null, 105, 1004, 0);
insert into kotoumi_genshin_wish_unit_map values (null, 105, 1010, 0);
insert into kotoumi_genshin_wish_unit_map values (null, 105, 1019, 0);
insert into kotoumi_genshin_wish_unit_map values (null, 105, 1020, 0);
insert into kotoumi_genshin_wish_unit_map values (null, 105, 1021, 0);

-- up四星角色：芭芭拉、菲谢尔、罗莎莉亚
insert into kotoumi_genshin_wish_unit_map values (null, 105, 1007, 1);
insert into kotoumi_genshin_wish_unit_map values (null, 105, 1012, 1);
insert into kotoumi_genshin_wish_unit_map values (null, 105, 1030, 1);

-- 非up四星角色
insert into kotoumi_genshin_wish_unit_map values (null, 105, 1006, 0);
insert into kotoumi_genshin_wish_unit_map values (null, 105, 1009, 0);
insert into kotoumi_genshin_wish_unit_map values (null, 105, 1011, 0);
insert into kotoumi_genshin_wish_unit_map values (null, 105, 1013, 0);
insert into kotoumi_genshin_wish_unit_map values (null, 105, 1014, 0);
insert into kotoumi_genshin_wish_unit_map values (null, 105, 1015, 0);
insert into kotoumi_genshin_wish_unit_map values (null, 105, 1016, 0);
insert into kotoumi_genshin_wish_unit_map values (null, 105, 1017, 0);
insert into kotoumi_genshin_wish_unit_map values (null, 105, 1018, 0);
insert into kotoumi_genshin_wish_unit_map values (null, 105, 1023, 0);
insert into kotoumi_genshin_wish_unit_map values (null, 105, 1025, 0);

-- 四星武器（这个一般不会变，copy就好）
insert into kotoumi_genshin_wish_unit_map values (null, 105, 2004, 0);
insert into kotoumi_genshin_wish_unit_map values (null, 105, 2005, 0);
insert into kotoumi_genshin_wish_unit_map values (null, 105, 2007, 0);
insert into kotoumi_genshin_wish_unit_map values (null, 105, 2008, 0);
insert into kotoumi_genshin_wish_unit_map values (null, 105, 2015, 0);
insert into kotoumi_genshin_wish_unit_map values (null, 105, 2016, 0);
insert into kotoumi_genshin_wish_unit_map values (null, 105, 2017, 0);
insert into kotoumi_genshin_wish_unit_map values (null, 105, 2018, 0);
insert into kotoumi_genshin_wish_unit_map values (null, 105, 2026, 0);
insert into kotoumi_genshin_wish_unit_map values (null, 105, 2027, 0);
insert into kotoumi_genshin_wish_unit_map values (null, 105, 2028, 0);
insert into kotoumi_genshin_wish_unit_map values (null, 105, 2029, 0);
insert into kotoumi_genshin_wish_unit_map values (null, 105, 2034, 0);
insert into kotoumi_genshin_wish_unit_map values (null, 105, 2036, 0);
insert into kotoumi_genshin_wish_unit_map values (null, 105, 2043, 0);
insert into kotoumi_genshin_wish_unit_map values (null, 105, 2044, 0);
insert into kotoumi_genshin_wish_unit_map values (null, 105, 2045, 0);
insert into kotoumi_genshin_wish_unit_map values (null, 105, 2048, 0);

-- 三星武器（这个一般不会变，copy就好）
insert into kotoumi_genshin_wish_unit_map values (null, 105, 2001, 0);
insert into kotoumi_genshin_wish_unit_map values (null, 105, 2002, 0);
insert into kotoumi_genshin_wish_unit_map values (null, 105, 2003, 0);
insert into kotoumi_genshin_wish_unit_map values (null, 105, 2012, 0);
insert into kotoumi_genshin_wish_unit_map values (null, 105, 2013, 0);
insert into kotoumi_genshin_wish_unit_map values (null, 105, 2014, 0);
insert into kotoumi_genshin_wish_unit_map values (null, 105, 2023, 0);
insert into kotoumi_genshin_wish_unit_map values (null, 105, 2024, 0);
insert into kotoumi_genshin_wish_unit_map values (null, 105, 2025, 0);
insert into kotoumi_genshin_wish_unit_map values (null, 105, 2033, 0);
insert into kotoumi_genshin_wish_unit_map values (null, 105, 2040, 0);
insert into kotoumi_genshin_wish_unit_map values (null, 105, 2041, 0);
insert into kotoumi_genshin_wish_unit_map values (null, 105, 2042, 0);
```

## 4. 致谢

感谢你看到这里，非常感谢你为海鸟阁卡池维护做出的努力，让我们一起让海鸟阁变得更好吧！