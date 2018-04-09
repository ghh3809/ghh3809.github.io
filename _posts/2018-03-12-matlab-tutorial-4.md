---
layout: post
title:  "同学，你掉了一份MATLAB入门教程(4)"
categories: MATLAB
tags:  MATLAB 流程控制
author: 风之筝
---

* content
{:toc}

通过前面的内容，我们已经对MATLAB中的数据类型和一些简单的命令有所了解。这篇文章将主要介绍一下MATLAB中的流程控制语句，包括顺序执行、条件控制和循环控制。相信经过这篇文章的学习，你已经可以写出像模像样的MATLAB程序了。




## 顺序执行

<script type="text/javascript" src="http://cdn.mathjax.org/mathjax/latest/MathJax.js?config=default"></script>

可能你已经注意到了，我们之前的语句都是在命令行窗口中进行编写的。这样虽然可以及时得到反馈，但是如果我们想要同时执行多个语句则是非常困难的。因此我们将利用MATLAB的脚本m文件，使其从一个“大计算器”变成一个“编程工具”。

顺序执行是程序执行过程中最基本的执行方式，在MATLAB中也不需要对其进行任何的控制，只需要把要执行的命令按顺序写下来就可以了。例如我们需要对下面的方程组进行求解：

$$\left(\begin{array}{ccc}1&2&\\3&4&5\\&6&7\\\end{array}\right)X=\left(\begin{array}{c}0\\1\\2\\\end{array}\right)$$

我们可以在m文件中编写以下内容。需要注意的是，如果语句不以分号结尾，则会将输出发送到命令行窗口；而如果是以分号结尾，程序虽然运行了，但不会产生任何输出：

``` matlab
A = [1,2,0;3,4,5;0,6,7]; % 语句以分号结尾时，结果将不输出到命令行窗口
b = [0,1,2]';
X = A \ b % 语句不以分号结尾时，结果将直接输出到命令行窗口，否则命令行将没有任何输出

% 以下是命令行输出结果
%
% X =
%
%    -0.1364
%     0.0682
%     0.2273
```

## 条件执行

顺序执行显然不能满足我们的需要，如果我们要操作的语句是根据一定的条件进行选择，那么我们就需要用到条件语句。

### if条件语句

最为常见的条件语句即为if条件语句，其使用方法包括以下两种：

``` matlab
% 如果condition条件为TRUE(1)，则执行语句statement，否则不执行
if condition
	statement
end
```

``` matlab
% 如果condition条件TRUE(1)，则执行语句statement1，否则执行语句statement2
if condition
	statement1
else
	statement2
end
```

为了方便对控制条件进行编写，这里介绍常见的关系运算符和逻辑运算符。常用的关系运算符包括以下几个：

- ==（等于），~=（不等于），>（大于），<（小于），>=（大于等于），<=（小于等于）

和其他编程语言类似，注意逻辑判断中的==（等于）和赋值语句中的=（赋值）的区别。

常用的逻辑运算符包括以下几个：

- &&（逻辑与），&#124;&#124;（逻辑或），~（逻辑非）。

注意在MATLAB中，FALSE的值默认为0，TRUE的值默认为1；在进行逻辑运算时，所有的0值被当做FALSE处理，非0值被当做TRUE处理。举例说明：

``` matlab
(3>2) && (2~=3); % 结果为1（TRUE）
~(4<=4); % 结果为0（FALSE）
(3>4) || 5; % 结果为1（TRUE）
~0; % 结果为1（TRUE）
```

> 下面三点写给“久经沙场”的程序员：
> - MATLAB提供位运算符，分别是&（按位与），&#124;（按位或），~（按位非）
> - MATLAB提供的逻辑运算符支持短路操作
> - MATLAB中不提供++、+=等类似的赋值操作符

好了，让我们回归正题。举一个栗子，如果一斤栗子10元，买5斤及以上总价打八折，计算价格的程序可以这样编写：

``` matlab
unitPrice = 10; % 单价10元/斤
weight = 8; % 输入购买的重量（斤）
if weight >= 5
	price = unitPrice * weight * 0.8
else
	price = unitPrice * weight
end

% 以下是命令行输出结果
%
% price =
% 
%     64
```

### if嵌套语句

if语句是可以进行层层嵌套的，例如下面是两种常见的嵌套模式：

``` matlab
% 若condition1满足：同时满足condition2时，执行statement21；不满足时执行statement22。若condition1不满足，则执行statement1
if condition1
	if condition2
		statement21
	else
		statement22
	end
else
	statement1
end
```

``` matlab
% 若condition1满足，则执行statement1；否则，若condition2满足，则执行statement21，不满足时执行statement22
if condition1
	statement1
else
	if condition2
		statement21
	else
		statement22
	end
end
```

通过if语句的嵌套形式，我们可以对运行流程进行更好的控制。再举一个栗子，刚刚的栗子如果买10斤及以上则总价打7折，则计算程序如下：

``` matlab
unitPrice = 10; % 单价10元/斤
weight = 12; % 输入购买的重量（斤）
if weight >= 5
	if weight >= 10
		price = unitPrice * weight * 0.7
	else
		price = unitPrice * weight * 0.8
	end
else
	price = unitPrice * weight
end

% 以下是命令行输出结果
%
% price =
% 
%     84
```

### switch语句

当我们要判断的分支非常多时，如果使用嵌套if语句写法，将会显得十分繁琐。这时我们可以使用switch语句来使得程序可读性更强。

switch语句的用法是这样的。

``` matlab
switch variable
	case {value11,value12...}
		statement1
	case value2
		statement2
	...
	case valuen
		statementn
	otherwise
		statement
end
```

当variable的值可以匹配上其中的任何value时，会直接跳转执行其对应的语句；如果没有匹配上任何值，则会直接跳转到otherwise语句进行执行。需要注意的是，otherwise语句并不是必须的。

不同于其他语言的switch语句，如果多个值对应同一个执行语句，可以利用大括号将值写在一起。这也意味着MATLAB不需要在每一个分支后使用break进行显式中断。

我们可以编写一个根据年份和月份判断该月有几天的程序：

``` matlab
year = 2012;
month = 2;
switch month
	case {1,3,5,7,8,10,12} % 一三五七八十腊，三十一天永不差
		days = 31
	case {4,6,9,11} % 四六九冬三十日
		days = 30
	case 2 % 二月需要判断是否为闰年
		if (mod(year,4) == 0 && mod(year,100) ~= 0) || (mod(year,400) == 0)
			days = 29
		else
			days = 28
		end
end

% 以下是命令行输出结果
%
% days =
% 
%     29
```

在这个例子中，我们在switch语句内部再次使用了if语句，用于对闰年的判断。其中`mod(a,b)`函数表示a对b的模。

## 循环执行

流程控制中的循环语句非常重要，它避免了我们写几乎重复的代码，而通过程序的循环与退出对其进行控制。常见的循环执行语句有while循环和for循环。

### while循环

while循环是最容易理解的一种循环，其用法如下所示：

``` matlab
while condition
	statement
end
```

当condition条件为真时，循环执行statement语句，然后再次判断condition条件，直至condition条件不再满足时退出循环。如果在进入while循环前，condition条件不满足，则循环直接不会执行。

我们以经典的“水滴数”计算为例，练习while循环。水滴数是这样计算的：对于任意一个正整数，如果它是奇数，则乘3再加1；如果它是偶数，就除以2。循环往复，最后终将变成1（该结果尚未被证明，但至今未发现反例）。

``` matlab
num = 50; % 以50为例，显示水滴数的变化过程。由于num的计算过程没有加分号，会输出每次运算的结果。
while num ~= 1 % 还没有减小到1时，不断循环
	if mod(num,2) == 0 % 如果是偶数
		num = num / 2
	else % 如果是奇数
		num = num * 3 + 1
	end
end

% 命令行输出结果：（略）
```

### for循环

回顾一下高斯的优秀事迹：如果要计算1到100的和，如何写这个循环呢？我们当然可以采用如下的while循环的方式进行编写：

``` matlab
sum = 0;	% 保存求和结果
i = 1;		% 当前数
while i <= 100
	sum = sum + i;
	i = i + 1; % 不要忘记变到下一个数
end
sum		% 不加分号，输出最后结果

% 以下是命令行输出结果
%
% sum =
%
%         5050
```

但是除了while循环外，还有一种更加方便的计数循环：for循环可以快速解决这个问题。for循环的格式如下：

``` matlab
for variable = value
	statement
end
```

其中value一般是利用冒号运算符得到的等差数列。我们利用for循环改写一下求和程序：

``` matlab
sum = 0;		% 保存求和结果
for i = 1:100
	sum = sum + i;	% 累加
end
sum 			% 输出结果

% 以下是命令行输出结果
%
% sum =
%
%         5050
```

可以看到使用for循环后整个程序变得更加简洁易懂。同样的，我们还可以求100以内所有奇数的平方和：

``` matlab
sum = 0;			% 保存求和结果
for i = 1:2:100			% 1~100以内所有奇数
	sum = sum + i^2;	% 累加
end
sum 				% 输出结果

% 以下是命令行输出结果
%
% sum =
%
%       166650
```

我们可以看到，使用for循环避免了编写复杂的循环控制条件，使得程序编写者可以直接确定循环的次数和循环变量的值。而当循环次数未知时，则仍需要使用while循环对其进行控制。
