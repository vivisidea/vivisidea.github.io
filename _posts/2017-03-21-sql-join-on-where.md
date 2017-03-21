---
title: SQL JOIN语句中的ON和WHERE用法
layout: post
tags:
  - java
---

SQL JOIN定义

* INNER JOIN（内连接）：取得两个表中存在连接匹配关系的记录。
* LEFT JOIN（左连接）：取得左表（table1）完全记录，即是右表（table2）并无对应匹配记录。
* RIGHT JOIN（右连接）：与 LEFT JOIN 相反，取得右表（table2）完全记录，即是左表（table1）并无匹配对应记录。

SQL JOIN 语法

```sql
SELECT ... FROM table1 INNER|LEFT|RIGHT JOIN table2 ON conditions1 WHERE conditions2
```

这里需要注意的是 ON 条件是在 JOIN 之前筛选数据，而 WHERE 是在 JOIN 之后筛选数据。

## 示例

```sql
CREATE TABLE `users` (
  `uid` int(11) NOT NULL AUTO_INCREMENT,
  `username` varchar(45) DEFAULT NULL,
  PRIMARY KEY (`uid`)
) ENGINE=InnoDB AUTO_INCREMENT=7 DEFAULT CHARSET=utf8mb4;

CREATE TABLE `orders` (
  `id` int(11) NOT NULL AUTO_INCREMENT,
  `name` varchar(45) DEFAULT NULL,
  `uid` int(11) DEFAULT NULL,
  PRIMARY KEY (`id`)
) ENGINE=InnoDB AUTO_INCREMENT=7 DEFAULT CHARSET=utf8mb4;
```

### users表

| uid  | username |
| ---- | -------- |
| 1    | user1    |
| 2    | user2    |
| 3    | user3    |
| 4    | user4    |
| 5    | user5    |
| 6    | user6    |

### orders 表

| id   | name   | uid  |
| ---- | ------ | ---- |
| 1    | order1 | 1    |
| 2    | order2 | 1    |
| 3    | order3 | 1    |
| 4    | order4 | 2    |
| 5    | order5 | 3    |
| 6    | order8 | 7    |



* 不管 LEFT JOIN 的条件是什么，左侧始终都是左表的所有记录

```sql
-- 由于没有符合连接条件的记录，右侧记录全部为 NULL
SELECT * FROM orders t1 LEFT JOIN users t2 on 1 > 2;
```

* 注意 ON 和 WHERE 的区别

```sql
-- ON 在 JOIN 之前筛选数据，符合条件的记录才进行连接
SELECT * FROM orders t1 LEFT JOIN users t2 on t1.uid = t2.uid AND t1.uid = 1;

-- WHERE 在连接完成之后筛选记录
SELECT * FROM orders t1 LEFT JOIN users t2 on t1.uid = t2.uid WHERE t1.uid = 1;
```