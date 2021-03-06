---
layout: post
title:  "同学，你掉了一份MATLAB入门教程(3)"
categories: MATLAB
tags:  MATLAB 数据类型
author: 风之筝
---

* content
{:toc}

这篇文章中我们将学习MATLAB中的各种数据类型，主要包括10种基本数据类型和4种复合数据类型。如果你拥有一些编程基础，那这篇文章对你应该非常轻松。




## 基本数据类型

<script type="text/javascript" src="http://cdn.mathjax.org/mathjax/latest/MathJax.js?config=default"></script>

和一般的编程语言一样，MATLAB的基本数据类型分为两种，即**整型(int)**和**浮点数(float)**。其中整形根据其是否包含负数可以进一步分为有符号整形和无符号整形，而浮点数根据其精度不同又可以进一步划分为单精度浮点数和双精度浮点数。下面的表格列出了各种基本数据类型对应的表示范围，这里类型名称沿用了一般编程语言中的名称。

|类型|符号|最小值|最大值|
| - | - | - | - | - |
|布尔型|logical|\\(0\\)|\\(1\\)|
|字节型|int8|\\(-2^7\\)|\\(2^7-1\\)|
|短整型|int16|\\(-2^{15}\\)|\\(2^{15}-1\\)|
|整形|int32|\\(-2^{31}\\)|\\(2^{31}-1\\)|
|长整型|int64|\\(-2^{63}\\)|\\(2^{63}-1\\)|
|无符号字节型|uint8|\\(0\\)|\\(2^8-1\\)|
|短整型|uint16|\\(0\\)|\\(2^{16}-1\\)|
|整形|uint32|\\(0\\)|\\(2^{32}-1\\)|
|长整型|uint64|\\(0\\)|\\(2^{64}-1\\)|
|单精度浮点数|single|\\(\pm1.1755\times10^{-38}\\)|\\(\pm3.4028\times10^{38}\\)|
|双精度浮点数|double|\\(\pm2.2251\times10^{-308}\\)|\\(\pm1.7977\times10^{308}\\)|

在MATLAB中，不显式指定数据类型的情况下，默认所有的数据类型均为double类型。如果需要创建一个其他类型的变量，只需要加上其符号名即可，例如

``` matlab
a = int8(13);		% 创建一个字节型整数13
b = single(25.56);	% 创建一个单精度浮点数25.56
c = -32;		% 未显式指明数据类型，创建一个双精度浮点数32
```

MATLAB中，有几个特殊的常量，它们分别是：

1. pi：圆周率π
2. eps：最小浮点数
3. Inf：正无穷大，即1/0
4. NaN：非数，即0/0
5. i,j：虚数单位

## 复合数据类型——矩阵

类似一般编程语言中的数组，MATLAB中也有很多复合数据类型，可以将众多基本数据类型合并到一起。最为常见的符合数据类型包括**矩阵**，**结构体**和**元胞矩阵**。

矩阵是MATLAB中的基本运算单元，即使是一个数在MATLAB中也是被当做1*1的矩阵看待的。矩阵由若干维度组成，例如0维矩阵表示一个元素，1维矩阵表示一个向量，还有2维、3维甚至更高维的矩阵。在MATLAB中，常规矩阵的最小维度为2维，即一般数学意义上的矩阵，而标量、向量都是矩阵的特例。下面我们将以2维矩阵为例，介绍其生成与运算。

矩阵的生成方式主要包括以下几种方式：

### 1. 直接输入

利用直接输入的方法，我们可以生成一个矩阵。输入的矩阵用中括号表示，同一行元素之间使用逗号或空格分开，用分号或回车结束一行的输入。例如

``` matlab
>> a = [1,2,3; 4,5,6]

a =

     1     2     3
     4     5     6

>> b = [2 3 4
5 6 7]

b =

     2     3     4
     5     6     7
```

当输入的矩阵不指定变量名称时，结果将暂时保存到ans变量中，即

``` matlab
>> [2,3; 3,4]

ans =

     2     3
     3     4
```

如果没有及时保存结果，ans变量中的内容将会被下一次不指定变量名称的输出结果覆盖。

### 2. 函数生成

MATLAB中有一些好用的函数，用于构造一些具有特殊性质的矩阵。例如常用的构造函数：

``` matlab
>> zeros23 = zeros(2,3) % 生成2*3的零矩阵

zeros23 =

     0     0     0
     0     0     0

>> ones3 = ones(3) % 生成3*3的全1矩阵，方阵只需要输入行数

ones3 =

     1     1     1
     1     1     1
     1     1     1

>> eye32 = eye(3,2) % 生成3*2的单位对角阵

eye32 =

     1     0
     0     1
     0     0
```

类似的函数还有rand()（生成随机矩阵）, magic()（生成幻方矩阵）等，大家可以自行尝试。

### 3. 矩阵计算

MATLAB中提供了以下的矩阵运算符

- +（加法），-（减法），*（乘法），'（转置），\（左除），/（右除），^（乘幂）

在使用矩阵运算符的过程中，应使得操作矩阵符合矩阵运算性质。例如

``` matlab
>> ones(2)^3

ans =

     4     4
     4     4

>> eye(3,2) * eye(2,4)

ans =

     1     0     0     0
     0     1     0     0
     0     0     0     0

>> ans'

ans =

     1     0     0
     0     1     0
     0     0     0
     0     0     0
```

特别的，当加法、减法和乘法作用于矩阵和标量时，代表对矩阵中每个元素进行对应的处理，例如

``` matlab
>> eye(2,3) + 1

ans =

     2     1     1
     1     2     1

>> 4.5*ones(2)

ans =

    4.5000    4.5000
    4.5000    4.5000
```

另外，对于左除和右除，其用法如下：

- 设\\(A\\)为可逆矩阵，\\(AX=B\\)的解是\\(X=A\setminus B\\)，\\(XA=B\\)的解是\\(X=B/A\\)。

除此之外，矩阵还有一些特殊的运算，称之为“点”运算，使用较多的包括.*（点乘）和.^（点乘幂）。点运算的特殊之处在于，它不是按照矩阵的运算规律进行计算的，而是对具有相同大小的两矩阵，将对应元素进行乘或乘幂的操作。例如

``` matlab
>> [1,2,3] .* [4,5,6]

ans =

     4    10    18

>> [1,2,3] .^ [4,5,6]

ans =

     1    32   729
```

### 4. 冒号运算符与矩阵拼接

在MATLAB中，:（冒号运算符）是非常重要的一个符号，利用冒号可以轻松表示等差数列。例如

``` matlab
>> a = 1:5 % 创建从1到5，公差为1（可省略）的等差数组

a =

     1     2     3     4     5

>> b = 1:3:10 % 创建从1到10，公差为3的等差数组

b =

     1     4     7    10
```

取出矩阵元素时，我们只需要指定选取的行号与列号即可。请注意，**MATLAB中所有下标均从1开始~~而非从0开始~~**，请一定牢记！！！

``` matlab
>> a = magic(5)

a =

    17    24     1     8    15
    23     5     7    14    16
     4     6    13    20    22
    10    12    19    21     3
    11    18    25     2     9

>> a(3,2) % 选取a的第3行，第2列元素

ans =

     6

>> a(1:2,2:4) % 选取a的1~2行，2~4列元素

ans =

    24     1     8
     5     7    14

>> a(2,:) % 选取a的第2行，所有列元素

ans =

    23     5     7    14    16

>> a(1:2:5,[2,5]) % 选取a的第[1,3,5]行，第[2,5]列元素

ans =

    24    15
     6    22
    18     9

>> a(3:end,4) % 选取a的第三行到最后一行，第4列元素

ans =

    20
    21
     2
```

基于此，我们可以对长度吻合的矩阵进行拼接，例如

``` matlab
>> a = [1:5, ones(1,3); zeros(2,8)]

a =

     1     2     3     4     5     1     1     1
     0     0     0     0     0     0     0     0
     0     0     0     0     0     0     0     0

>> b = [a(1,1:2:7); 4:-0.5:2.5]

b =

    1.0000    3.0000    5.0000    1.0000
    4.0000    3.5000    3.0000    2.5000
```

除了冒号运算符，还有一种生成等差数列的方式，即使用`linspace(a,b,n)`函数生成从a到b共n个数值的等差数列。同样的，利用`logspace(a,b,n)`可以生成从\\(10^a\\)到\\(10^b\\)共n个数值的等比数列。例如

``` matlab
>> a = linspace(0,pi,5)

a =

         0    0.7854    1.5708    2.3562    3.1416

>> b = logspace(0,1,4)

b =

    1.0000    2.1544    4.6416   10.0000
```

注意，无论使用冒号运算符，或是linspace函数，生成的矩阵均为1*n的矩阵，即为行向量。使用转置符号可以将其转置为列向量。

## 复合数据类型——字符串

上面介绍的所有类型均为数字类型的变量。而最常见的非数类型的变量就是字符串了。MATLAB中，使用单引号表示字符串：

``` matlab
>> a = 'hello'

a =

hello
```

字符串是一个特殊的行向量，其每个元素是一个字符。

``` matlab
>> b = a(2:4)

b =

ell
```

字符串的拼接类似数组的拼接，但是需要注意由于字符串是行向量，只能进行横向拼接，否则将会报错。

``` matlab
>> [a,b]

ans =

helloell
```

字符串和数字之间的转换也十分容易，可以利用`num2str()`和`str2num()`函数进行：

``` matlab
>> ['1+1=', num2str(1+1)]

ans =

1+1=2

>> 2 + str2num('3.2')

ans =

    5.2000
```

## 复合数据类型——结构体（了解）

结构体是一种特殊的数据类型，是由“变量”和其拥有的“字段”共同组成的，在初学阶段可能见到的比较少，更多的是调用MATLAB内置函数后生成的结构体变量。例如一个学生，他拥有姓名、年龄、期末成绩三个“字段”。利用“.”点运算，我们可以这样定义一个学生：

``` matlab
>> student.name = 'Mike'

student = 

    name: 'Mike'

>> student.age = 22

student = 

    name: 'Mike'
     age: 22

>> student.grade = [83,80,85,90]

student = 

     name: 'Mike'
      age: 22
    grade: [83 80 85 90]
```

可以看到，随着我们不断向student变量中增加字段，其包含的信息也越来越多。同样的，我们也可以使用`struct(filed,value,...)`的方式直接生成结构体：

``` matlab
>> student = struct('name','Mike','age',22,'grade',[83,80,85,90])

student = 

     name: 'Mike'
      age: 22
    grade: [83 80 85 90]

>> student.grade

ans =

    83    80    85    90
```

如果想要了解更多关于结构体的用法，可以查阅MATLAB的帮助文档。

## 复合结构类型——元胞矩阵（了解）

元胞矩阵是矩阵的一种扩展，虽然直到现在我也不知道这个名字是怎么取出来的。在矩阵中，一个元素代表的都是一个数，或者都是一个字符。但是在元胞矩阵中，这种限制被打破，不同位置可以拥有任何类型的元素，包括字符串，数值或矩阵。为了区分元胞矩阵和普通矩阵，我们使用大括号进行定义，在使用元素时，也是用大括号对其进行取出。

``` matlab
>> a = {'hello',500; [30,20,40],[]}

a = 

    'hello'         [500]
    [1x3 double]       []

>> a{1,1}

ans =

hello

>> a{2,1}(3)

ans =

    40
```

可能你在尝试的过程中也发现了，利用小括号也能对其元素进行取出。关于这两种方式有何不同，我们将在之后的一篇关于元胞矩阵的讨论中再次提到。同样的，MATLAB的帮助文档永远是你的好帮手。
