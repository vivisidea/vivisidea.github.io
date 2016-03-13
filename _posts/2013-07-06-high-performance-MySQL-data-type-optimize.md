---
title: 高性能MySQL笔记-数据类型优化
author: vivi
layout: post
tags:
  - mysql
---
高性能MySQL笔记-数据类型优化

基本原则

- 尽量使用可以正确存储数据的最小数据类型。
- 尽量使用简单的数据类型，能用整型就不要用字符串，用内建日期类型保存日期。
- 尽量避免使用NULL，可以设定默认值，例如字符串可设定为NOT NULL DEFAULT '';
    - NULL的存在使索引，值比较都更为复杂。特别是如果计划在列上建索引，更应该避免允许NULL值。
    - 例外：InnoDB使用bit保存NULL值，对于稀疏数据会有比较好的空间效率。

- 选定大的数据类型：数字，字符串，时间等
- 选择具体类型，例如DATETIME和TIMESTAMP都可以存储相同类型的数据，时间和日期，精确到秒。但是TIMESTAMP只使用DATETIME一半的存储空间，并且会根据时区变化。

## 整数类型
`TINYINT(8)`， `SMALLINT(16)`, `MEDIUMINT(24)`, `INT(32)`, `BIGINT(64)`，可存储的值范围从`-2^(N-1) - 2^(N-1) -1`
整数类型有可选的UNSIGNED属性，表示不允许负值，可以将正数的上限提高一倍，例如TINYINT的范围是-128 127，TINYINT UNSIGNED的存储范围是0-255。

MySQL可以为整数类型指定宽度，例如INT(11)，对于大多数应用这是没有意义的，它不会改变可存储值的范围，只是规定了一些工具（例如MySQL命令行客户端）用来显示字符的个数，对于存储和计算值来说，INT(1)和INT(20)是相同的。

## 实数类型
DECIMAL类型可以用来存储比BIGINT还大的整数，也可以用来精确的存储实数。FLOAT(32)和DOUBLE(64)使用标准的浮点运算进行近似计算。

数字类型只是选择数据怎么在内存和磁盘中保存的，整数的计算一般使用64bit的BIGINT整数，而实数类型一般转换成DOUBLE类型进行计算。

## 字符串类型
VARCHAR类型和CHAR类型（以下说明基于InnoDB引擎，不同的数据库引擎可能有不同的实现方式）

VARCHAR用于存储可变长度字符串，使用1个或2个额外的字节记录字符串的长度，适用于字符串列最大长度比平均长度大很多的情况。

CHAR类型是定长的，MySQL总是根据定义的字符串长度分配足够的空间，存储CHAR值时，MySQL会删除所有末尾的空格，CHAR适合存储比较短的字符串，或者所有值都接近同一个长度，例如MD5值。

与CHAR和VARCHAR类似的类型还有BINARY和VARBINARY类型，它们存储的是二进制字符串，二进制字符串跟常规字符串非常相似，但二进制字符串存储的是字节码而不是字符。二进制字符串没有排序规则或者字符集。

VARCHAR(5)和VARCHAR(200)的区别：它们在存储字符串`hello`时空间开销是一样的，但是更长的列会消耗更多的内存，MySQL通常会分配固定大小的内存块来保存内部值，所以，最好的策略是只分配真正需要的空间。

## 其他字符串类型

- TINYTEXT --> TINYBLOB
- SMALLTEXT --> SMALLBLOB
- TEXT --> BLOB
- MEDIUMTEXT --> MEDIUMBLOB
- LONGTEXT --> LONGBLOB

BLOB和TEXT之间的区别在于BLOB存储的是二进制数据，没有排序规则或字符集。

## 使用ENUM类型代替字符串类型
枚举列可以把一些不重复的字符串存储成一个预定义的集合，实际存储的时候会使用整数
{% highlight sql %}
CREATE TABLE  enum_test(
    e ENUM('fish', 'dog', 'apple') NOT NULL
);
INSERT INTO enum_test(e) VALUES('fish'), ('dog'), ('apple');
{% endhighlight %}

## 时间和日期
DATETIME 能保存大范围的值，从1001年到9999年，精度为秒。使用8个字节的存储空间。

TIMESTAMP 保存从1970-01-01 00:00:00以来的秒数，和UNIX时间戳相同，TIMESTAMP只使用4个字节的存储空间，只能表示从1970年到2038年

`FROM_UNIXTIME()`将Unix时间戳转换为日期。

`UNIX_TIMESTAMP()`函数将日期转换为Unix时间戳。

{% highlight sql %}
-- 纯测试的语句一条
mysql> SELECT FROM_UNIXTIME(UNIX_TIMESTAMP(CURRENT_TIMESTAMP()));
+----------------------------------------------------+
| FROM_UNIXTIME(UNIX_TIMESTAMP(CURRENT_TIMESTAMP())) |
+----------------------------------------------------+
| 2013-07-06 15:40:38                                |
+----------------------------------------------------+

CREATE TABLE `t3` (
  `id` int(10) UNSIGNED NOT NULL AUTO_INCREMENT,
  `insert_time` TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
  `modify_time` DATETIME NOT NULL,
  PRIMARY KEY (`id`)
) ENGINE=InnoDB DEFAULT CHARSET=latin1;
{% endhighlight %}

TIMESTAMP有DATETIME没有的特殊属性，默认情况下，如果INSERT时没有指定第一个TIMESTAMP列的值，那么MySQL会设置这个列为当前时间，更新时如果没有指定第一个TIMESTAMP列的值，也会默认被更新为当前时间，可以配置任意TIMESTAMP列的INSERT和UPDATE行为。

## 特殊类型的数据
习惯上使用VARCHAR(15)来保存IPv4地址，但实际上使用INT UNSIGNED来保存更为合适，对于一些特殊的查询需求支持也更好（例如查询IP段），MySQL提供了INET_ATON()和INET_NTOA()来转换这两种表示方式。
{% highlight sql %}
mysql> SELECT INET_ATON('192.168.1.1');
mysql> SELECT INET_NTOA(3232235777);
{% endhighlight %}
