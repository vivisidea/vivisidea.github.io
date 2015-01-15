---
title: 高性能MySQL笔记-分区表
author: vivi
layout: post
permalink: /posts/291.html
categories:
  - 技术记录
tags:
  - mysql
---
高性能MySQL笔记-分区表

MySQL的分区表是一个独立的逻辑表，底层由多个物理子表组成，因此索引也是按照分区的子表定义的，而没有全局索引。
分区的主要目的是将数据以一个比较粗的粒度分在不同的表中，适用的场景：

1. 历史数据/热点数据分离
2. 批量删除大量数据，通过分区表可以快速实现，例如将前一天的数据分区直接删除掉
3. 分区表的数据可以分布在不同的物理设备上
4. 可以备份/恢复独立的分区

实际工作中有碰到适用于分区表的需求

1. 数据只需要保留N天
2. 大部分情况下只需要对当天的数据进行操作，即当天的数据为热点数据
3. 数据量比较大，单表查询效率比较低

解决方案：使用分区表，定时任务每天凌晨自动生成下一个分区，同时将N天前的分区drop掉。

{% highlight sql %}
-- 比较常用的是RANGE
CREATE TABLE sales(
    id INT NOT NULL AUTO_INCREMENT,
    order_date DATETIME NOT NULL,
    PRIMARY KEY (id)
)ENGINE=InnoDB PARTITION BY RANGE(id)(
    PARTITION p0 VALUES LESS THAN (100),
    PARTITION p1 VALUES LESS THAN (200),
    PARTITION p2 VALUES LESS THAN MAXVALUE 
    -- 类似if-else的机制，后面的比较值需要比前面的比较值大
);
-- 分析查询语句使用分区情况
EXPLAIN PARTITIONS SELECT * FROM sales WHERE id < 10;
    
-- 使用分区表后有些查询通常都需要带上分区条件字段，以缩小查询范围，或者可以强制指定查询索引
SELECT * FROM bbs FORCE INDEX (`idx_status_cbsts`) 
    WHERE callbackStatus = 0 
      AND status IN (2000, 3000)

-- 修改表结构，将原来非分区表的表修改成分区表
ALTER TABLE t1 PARTITION BY RANGE(year_col)( 
    PARTITION p0 VALUES LESS THAN (2000), 
    PARTITION p1 VALUES LESS THAN (2005), 
    PARTITION p2 VALUES LESS THAN (2010), 
    PARTITION p3 VALUES LESS THAN MAXVALUE
);
-- 分区表删除和增加操作
ALTER TABLE t1 DROP PARTITION P3;
ALTER TABLE t1 ADD PARTITION ( PARTITION p4000 VALUES LESS THAN (2015) );

-- tips：为了考虑到分区增减的可能性，分区名字最好留点空间，例如p1000, p2000, p3000 ...
-- 这在设计`状态`字段的时候同样适用，status=10,20,30会比status=1,2,3要更容易扩展

{% endhighlight %}

参考资料

1. [官方有关分区的介绍](http://dev.mysql.com/doc/refman/5.5/en/partitioning.html)
2. [建表语句partition语法](http://dev.mysql.com/doc/refman/5.5/en/create-table.html)
3. [对已有分区的操作](http://dev.mysql.com/doc/refman/5.1/en/alter-table-partition-operations.html)

