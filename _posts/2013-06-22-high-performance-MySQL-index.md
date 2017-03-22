---
title: 高性能MySQL笔记-索引
author: vivi
layout: post
tags:
  - mysql
---
## MySQL的RENAME操作
例如在定期重建一张表数据的时候可以用到

```sql
DROP TABLE IF EXISTS my_summary_old, my_summary_new;
CREATE TABLE my_summary_new like my_summary;
RENAME TABLE my_summary TO my_summary_old, my_summary_new TO my_summary;
```

## 并发更新某一行数据场景优化
计数器表场景可以使用多行+汇总的形式，如果只有一行，那么所有更新操作都只能串行，任何想更新这一行的事务都需要对该行记录进行全局锁定。
```sql
CREATE TABLE hit_counter(
	day date not null,
	slot tinyint unsigned not null,
	cnt int unsigned not null,
	primary key(day, slot)
)ENGINE = InnoDB;
-- 插入数据的时候随机的选一行
INSERT INTO hit_counter(day, slot, cnt) VALUES (CURRENT_DAY, RAND() * 100, 1) 
    ON DUPLICATE KEY UPDATE SET cnt = cnt + 1;
```

统计的时候`SUM(cnt) GROUP BY day`即可得到每天的数据；

## 3. B+Tree索引的使用适用于全键值，键值范围或者键前缀查找（最左前缀）
```sql
REATE TABLE people(
    last_name varchar(50) not null,
    first_name varchar(50) not null,
    dob date not null,
    gender enum('m', 'f') not null,
    key(last_name, first_name, dob)
);
```

查询适用于
 
1. 提供`last_name, first_name, dob`条件 
2. 根据`last_name`查询，或者对`last_name`进行范围查询
3. 提供`last_name`并且对`first_name`进行范围查询

不适用于

1. 直接根据`first_name`的查询，也无法查询有特定`dob`的列表
2. 不能跳过索引中的列，例如无法用于查询`last_name ='smith' and dob = 'xxxx'`;， 如果不指定`first_name`，那么MySQL只能使用索引的第一列
3. 如果查询中有某个列是范围查询，那么其右边的所有列都无法使用索引优化查找，例如`last_name = 'smith'; AND first_name LIKE 'J%'; AND dob = 'xxxx'`;，那么查询只能用到索引的前两列。（基于这个规则，通常尽量把范围条件放在多列索引的最后，以尽可能的使用索引列）

因此在建立联合索引的时候，列的顺序是很重要的。

## 创建自定义的hash索引
只有Memory引擎才支持hash索引，但是我们可以使用`CRC32()`函数或者`FNV64()`函数来创建自定义索引，提高检索的效率
```sql
CREATE TABLE url(
    id INT NOT NULL AUTO_INCREMENT COMMENT '子增主键id',
    url VARCHAR(2048) NOT NULL COMMENT '地址',
    url_crc32 INT UNSIGNED NOT NULL COMMENT '测试',
    PRIMARY KEY(id),
    INDEX idx_crc32(url_crc32)
);

-- 插入数据的时候使用触发器自动计算crc32值（update也需要创建相应的触发器）
DELIMITER //
CREATE TRIGGER url_crc_ins BEFORE INSERT ON url FOR EACH ROW BEGIN
SET NEW.url_crc32 = CRC32(NEW.url);
END;
//
```

由于不可避免的出现crc32值冲突，因此查询的时候需要增加`url=';xxx'`;条件
```sql
SELECT url FROM url WHERE url = 'http://www.mysql.com/' AND url_crc32 = CRC32('http://www.mysql.com/')
```

如果表比较大，那么可以考虑把hash函数改成64位的，为了方便可以直接取md5的一部分转换成64位整数
```sql
SELECT CONV(RIGHT(MD5('http://www.mysql.com/'), 16), 16, 10);
```

## 独立的列，始终将索引列单独放在比较符号的一侧
```sql
SELECT xx FROM Person WHERE age + 1 = 5 <-- DON'T DO THIS.
```

## 前缀索引，索引的选择性
索引的选择性是指 不重复的索引记录数量 / 表的总记录数 比值，比值越大，说明索引的选择性越好，计算合适的前缀长度的一个办法就是算完整列的选择性，并使前缀的选择性近于完整列的选择性
```sql
SELECT COUNT(DISTINCT city) / COUNT(*) FROM city_demo;

-- 通常来说前缀7左右就差不多了
SELECT COUNT(DISTINCT LEFT(city, 3)) / COUNT(*) AS sel3, 
        COUNT(DISTINCT LEFT(city, 4)) / COUNT(*) AS sel4, 
        COUNT(DISTINCT LEFT(city, 5)) / COUNT(*) AS sel5, 
        COUNT(DISTINCT LEFT(city, 6)) / COUNT(*) AS sel6, 
        COUNT(DISTINCT LEFT(city, 7)) / COUNT(*) AS sel7, 
        COUNT(DISTINCT LEFT(city, 8)) / COUNT(*) AS sel8
FROM city_demo;

-- 增加前缀索引
ALTER TABLE city_demo ADD INDEX `idx_city`(city(7));
-- 原来SUM还可以这么使用，这里统计的是值等于xx的记录有多少条
SELECT SUM(staff_id = 2) , SUM(customer_id = 584) FROM payment; 
```

## 组合索引的顺序
通常来说，当不想要考虑排序和分组时，将选择性最高的列放在前面通常是比较好的选择。
也需要考虑特殊的情况，例如有些应用的管理员帐号，是所有用户的好友，这时候如果建立userid帐号索引，那么针对管理员帐号的查询效率会比较低。

## 分页处理
```sql
SELECT <cols> FROM table_name WHERE sex = 'M' ORDER BY rating LIMIT 10000, 10;
```

随着偏移量的增加，MySQL需要花费大量的时间来扫描需要丢弃的数据，解决的办法是限制用户能翻页的数量，实际上对用户影响不大，实际上用户并不会去翻那么 页。
另一个解决的方案是使用延迟关联，即通过覆盖索引查询返回需要的主键，再根据这些主键关联原表获得需要的行数据。
```sql
SELECT <cols> FROM profiles INNER JOIN(
        SELECT <primary key cols> FROM profiles
                WHERE x.sex = 'M' ORDER BY rating LIMIT 10000, 10
) AS x USING(<primary key cols>);
```

## 书中提到的命令
```sql
ANALYZE TABLE table_name;   -- 更新数据库表索引统计信息
SHOW INDEX FROM table_name; -- 显示表索引统计数据
```

