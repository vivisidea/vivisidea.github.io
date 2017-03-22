---
title: 高性能MySQL笔记-查询优化
author: vivi
layout: post
tags:
  - mysql
---
查询为什么慢

## 是否请求了过多的数据，是否返回太多的行，或者请求了太多的列
通常写SQL语句的时候都会偷懒直接写SELECT * FROM tb 但其实这个并不是个好习惯，最佳实践是只返回需要的列，例如
```sql
SELECT col1, col2 FROM tb1 WHERE col1 = xxx AND col2 = yyy
```

假设有组合索引 col1, col2 如果使用SELECT * ，那么是无法使用到覆盖索引的，MySQL需要通过索引再次查询一次记录，另外即使本来就需要返回所有的列，使用列名也有个好处，那就是当需求变更，表需要增加一列的时候，原有业务不会受到影响

```sql
ALTER TABLE tb1 ADD COLUMN xxx VARCHAR(32) NOT NULL COMMENT 'comment here' AFTER `column`
```

## 在IN中增加子查询，性能通常会非常糟糕，建议使用JOIN等效来改写查询
```sql
-- 查询某个演员出演的所有电影
SELECT * FROM film WHERE film_id IN(
    SELECT film_id FROM film_actor WHERE actor_id = 1
) LIMIT 10;
-- 可以改写成
SELECT film.* FROM film INNER JOIN film_actor USING(film_id) WHERE film_actor.actor_id = 1


-- IN + 子查询示例，无意义
mysql> EXPLAIN SELECT * FROM t2 WHERE id IN(SELECT id FROM t2 WHERE id = 1);
+--------------------+-------+---------------+---------+-------+--------+-------------+
| select_type        | type  | possible_keys | key     | ref   | rows   | Extra       |
+--------------------+-------+---------------+---------+-------+--------+-------------+
| PRIMARY            | ALL   | NULL          | NULL    | NULL  | 120715 | Using where |
| DEPENDENT SUBQUERY | const | PRIMARY       | PRIMARY | const |      1 | Using index |
+--------------------+-------+---------------+---------+-------+--------+-------------+

-- 使用 INNER JOIN 改写 IN 语句
mysql> EXPLAIN SELECT * FROM t2 JOIN (SELECT id FROM t2 WHERE id = 1) AS t USING(id);
+-------------+--------+---------------+---------+-------+------+-------------+
| select_type | type   | possible_keys | key     | ref   | rows | Extra       |
+-------------+--------+---------------+---------+-------+------+-------------+
| PRIMARY     | system | NULL          | NULL    | NULL  |    1 |             |
| PRIMARY     | const  | PRIMARY       | PRIMARY | const |    1 |             |
| DERIVED     | const  | PRIMARY       | PRIMARY |       |    1 | Using index |
+-------------+--------+---------------+---------+-------+------+-------------+
```

MySQL不允许对一张表同时进行更新和查询。
```sql
UPDATE tb1 INNER JOIN(
    SELECT type, COUNT(*) AS cnt FROM tb1 GROUP BY type
) AS tmp USING(type)
SET tb1.cnt = tmp.cnt
```

INNER JOIN会被当作一张临时表来处理，子查询会在UPDATE语句打开表之前完成，语句变成多表关联UPDATE。

## 优化COUNT()查询
COUNT()用来统计某个列值的数量，也可以统计行数。统计列值时要求列值是非空的（不统计NULL），如果在COUNT()括号中指定了列或者列的表达式，则统计的就是则个表达式有值的结果数。因此，COUNT(col) 和 COUNT(\*)是有区别的。COUNT(\*)会忽略所有的列值而直接统计所有的行数。

MyISAM引擎在没有WHERE条件的情况下，可以利用存储引擎的特性直接获得这个值，因此速度非常快，有WHERE条件的情况下，和其他引擎没有什么不同。。。

在不需要完全精确的应用场景下，可以考虑直接使用EXPLAIN的优化器预估输出作为近似值。

## 优化关联查询

- 确保ON或者USING子句中的列有索引。在创建索引的时候要考虑到关联的顺序，通常来说只需要在关联顺序中的第二个表的相应列上创建索引即可。
- 确保GROUP BY 和 ORDER BY中的表达式只涉及到一个表中的列。

## 优化子查询

尽可能使用关联查询来替代子查询。


