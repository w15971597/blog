# mysql字段类型
TEXT 和 BLOB 的主要差别是 BLOB 能够保存二进制数据；而 TEXT 只能保存字符数据

# count(*)慢怎么办
## count(*)的实现方式
 - MyISAM引擎 表的总数存起来了，count(*)直接返回，效率高
 - InnoDB则需要一行一行累计（因为MVCC的机制，每一行记录都要判断自己是否对这个事务可见，对于count(*),innodb只能都读出来，可见的行才能够用计算表的总行数

## 用缓存保存计数
如果对总数的具体值没有那么敏感，可以用redis保存总行数，表每插入一行redis计数加1，每删除一行redis计数减一
## 数据库保存计数
放到单独的一张计数表中


