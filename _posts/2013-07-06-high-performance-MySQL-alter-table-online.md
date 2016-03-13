---
title: 高性能MySQL笔记-在线修改表结构
author: vivi
layout: post
tags:
  - mysql
---
在线修改InnoDB表结构

MySQL修改表结构的时候，会复制一份原有表的副本，对于表结构的修改在副本上进行，修改好之后使用新表替换旧表，整个过程会产生锁表。
实际工作中有做过类似的误操作（需求改动，字段长度不够），以为改表结构很快就结束，结果造成了近40分钟的锁表，系统无法使用。

{% highlight sql %}
ALTER TABLE MODIFY COLUMN xxx VARCHAR(64) NOT NULL DEFAULT '';
{% endhighlight %}

后来查了一些资料，发现有成熟的工具可以完成在线改表结构的工作，例如percona的系列工具之一pt-online-schema-change

{% highlight bash %}
vivi@ubuntu-ssd:~$ pt-online-schema-change --alter \
     "MODIFY COLUMN xxx VARCHAR(64) NOT NULL DEFAULT ''" \
     D=test,t=t2 --user dba --ask-pass --execute \
Enter MySQL password: 
Enter MySQL password: 
Operation, tries, wait:
  copy_rows, 10, 0.25
  create_triggers, 10, 1
  drop_triggers, 10, 1
  swap_tables, 10, 1
  update_foreign_keys, 10, 1
Altering `test`.`t2`...
Creating new table...
Created new table test._t2_new OK.
Altering new table...
Altered `test`.`_t2_new` OK.
Creating triggers...
Created triggers OK.
Copying approximately 1048693 rows...
Copied rows OK.
Swapping tables...
Swapped original and new tables OK.
Dropping old table...
Dropped old table `test`.`_t2_old` OK.
Dropping triggers...
Dropped triggers OK.
Successfully altered `test`.`t2`.
{% endhighlight %}

原理大致如下

- 创建一个新的表
- 针对原表创建触发器，原表的更新操作都会被触发器处理更新到新表中
- 复制原始表的数据到新表（复制过程是分批的，默认 `chunk-size=1000` ）
- 新表替换旧表

小细节

- 实施改表操作的帐号需要PROCESS权限和REPLICATION SLAVE权限（一般由DBA帐号完成）
- 如果表有外键，那么需要特殊处理（参见 `alter-foreign-keys-method` 选项）
- 原表不能有触发器

[详细参数说明](http://www.percona.com/doc/percona-toolkit/2.2/pt-online-schema-change.html)
