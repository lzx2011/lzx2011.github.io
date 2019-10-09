---
title: sql重复插入问题
date: 2017-01-13 20:21:14
categories: mysql
tags: mysql
---
# sql重复插入问题

# 问题
在项目中，由于别人并发的调用接口，导致插入了重复数据

# 解决方案
1.因为使用多台机器部署，可以使用分布式锁用一台机器处理，对处理的方法加锁或同步关键字，但性能会有很大影响，分布式的优势也没了
2.在MySQL的业务表中，根据业务建立唯一索引，防止数据重复

# 具体操作
建立唯一索引：
```java
ALTER TABLE table_name ADD UNIQUE index_name (column_list)
```

# 程序中处理
如果重复插入，MySQL好像会报数据完整性的错误，到spring中后错误被封装为 DuplicateKeyException，只要在程序中捕获这个异常做相应的处理就可以了。

# 重复插入语句
建立唯一索引或使用主键primary后，还可以使用MySQL的重复插入语句：ignore， replace， ON DUPLICATE KEY UPDATE

```java
INSERT IGNORE INTO `table_name` (`email`, `phone`, `user_id`) VALUES ('test@163.com', '99999', '9999');
```
这样当有重复记录就会忽略,执行后返回数字0

```java
REPLACE INTO `table_name`(`col_name`, ...) VALUES (...);
```
REPLACE的运行与INSERT很相像,但是如果旧记录与新记录有相同的值，则在新记录被插入之前，旧记录被删除。

```java
INSERT INTO `table` (`a`, `b`, `c`) VALUES (1, 2, 3) ON DUPLICATE KEY UPDATE `c`=`c`+1; 
```
如果行作为新记录被插入，则受影响行的值为1；如果原有的记录被更新，则受影响行的值为2。

> 这部分具体可参考 http://www.111cn.net/database/mysql/50135.htm

#总结
不过实际发现异常处理的时间要比MySQL的重复插入语句慢不少,所以可以的话还是使用MySQL的重复插入语句，不要在程序中去处理；但如果业务原因，可能也不得不在程序中处理，比如重复插入了，也要返回插入的数据信息。


> INNODB中NULL字段使用插曲 
>
在大多数情况下字段设计应该避免使用default null的使用，而使用空字符来代表空。
因为INNODB的索引中会存储NULL，如果一个字段可为NULL，并且在该字段上有索引，索引中会存储NULL，每次索引的时候会额外扫更多的字段。在需要使用唯一索引约束一个字段，但是需要部分字段为空时，空字符串会引起唯一索引冲突，NULL可以在唯一索引中不产生冲突。
可参考文章：http://tomblog.readthedocs.io/en/latest/mysql/INNODB%E4%B8%ADNULL%E4%BD%BF%E7%94%A8.html
Mysql联合唯一索引和空值: http://tomblog.readthedocs.io/en/latest/mysql/INNODB中NULL使用.html

# 参考资料
<a href="http://www.111cn.net/database/mysql/50135.htm" target="_blank">MySql避免重复插入记录方法(ignore,Replace,ON DUPLICATE KEY UPDATE)</a>
