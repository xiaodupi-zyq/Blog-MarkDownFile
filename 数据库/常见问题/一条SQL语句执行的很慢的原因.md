# 一条SQL语句执行的很慢的原因
1. 大多数情况正常，偶尔很慢，则有如下原因
   1. 数据库在刷新脏页，例如redo log写满了需要同步到磁盘；内存不够用了，MySQL认为系统空闲，MySQL正常关闭；
   2. 执行的时候，遇到锁，如表锁、行锁；别的事务对表加锁之后我们无法访问该表；
   
2. 这条SQL一直都很慢的原因：
   1. 没有用上索引：例如该字段没有索引；由于对字段进行运算、函数操作导致无法用索引。
   2. 数据库选错了索引；例如本该选择字段索引的选择了主键索引；