# SQL语句
一个名为 "Persons" 的表：

Id | LastName | FirstName | Address | City
---|----------|-----------|---------|-----
1 | Adams | John | Oxford Street | London
2 | Bush | George | Fifth Avenue | New York
3 | Carter | Thomas | Changan Street | Beijing

**选取LastName列的数据**
SELECT LastName FROM Persons;

SQL对大小写不敏感。

**SQL DML和DDL**
可以把SQL分为两个部分：数据操作语言DML和数据定义语言DDL；

**SQL语言的DML部分**
语句 | 操作
---|---
SELECT | 获取操作
UPDATE | 更新操作
DELETE | 删除操作
INSERT INTO | 插入操作

**SQL语言的DDL** 创建和删除表格
语句 | 操作
---|---
CREATE DATABASE | 创建新数据库
ALTER DATABASE | 修改数据库
CREATE TABLE | 创建新表
ALTER TABLE | 修改表
DROP TABLE | 删除表
CREATE INDEX | 创建索引
DROP INDEX | 删除索引

### SQL SELECT语法

    SELECT 列名称 FROM 表名称
    SELECT * FROM 表名称 // *代表所有列

例如 ： SLECT LastName,FirstName FROM Persons;

### SQL SELECT DISTINCT语句
表中包括重复值，但你只想要不同的值，（也就类似于set）
关键词DISTINCT 用于返回唯一不同的值

    SELECT DISTINCT 列名称 FROM 表名称

表Orders：
Company | OrderNumber
--------|------------
IBM | 3532
W3School | 2356
Apple | 4698
W3School | 6953

SELECT Company FROM Orders; //所有内容都返回

SELECT DISTINCT Company FROM Orders; //只返回不同的Company

### WHERE子句
如需有条件的从表中选取数据，可将WHERE子句添加到SELECT中；

    SELECT 列名称 FROM 表名称 WHERE 列 运算符 值;

操作符 | 描述
----|---
= | 等于
<> | 不等于
> | 大于
< | 小于
>= | 大于等于
<= | 小于等于
BETWEEN | 在某个范围内
LIKE | 搜索某种模式

在上文中的Persons表中查询居住在Beijing中的人；

    SELECT * FROM Persons WHERE City = 'Beijing';

### AND和OR运算符
AND 和 OR 可在 WHERE 子语句中把两个或多个条件结合起来。

Persons表中显示所有姓Carter名Thomas的人；

    SELECT * FROM Persons WHERE LastName = 'Carter' AND FirstName = 'Thoms';

OR 运算符实例

使用 OR 来显示所有姓为 "Carter" 或者名为 "Thomas" 的人：
    SELECT * FROM Persons WHERE LastName = 'Carter' OR FirstName = 'Thoms';

### ORDER BY语句
默认升序排序，可使用DESC变为降序；

Orders表中：

以字母顺序显示公司名称：

    SELECT * FROM Orders ORDER BY Company;

以字母顺序显示公司名称（Company），并以数字顺序显示顺序号（OrderNumber）：

    SLECT * FROM Orders ORDER BY Company，OrderNumber;

以逆字母顺序显示公司名称，并以数字顺序显示顺序号：

    SLECT * FROM Orders ORDER BY Company DESC，OrderNumber ASC;

### INSERT INTO语句
向表格中插入新的行：

    INSERT INTO 表名称 VALUES (值1，值2，)
    INSERT INTO table_name (列1, 列2,...) VALUES (值1, 值2,....)

INSERT INTO Persons VALUES ('Gates', 'Bill', 'Xuanwumen 10', 'Beijing')
INSERT INTO Persons (LastName, Address) VALUES ('Wilson', 'Champs-Elysees')

### UADATE语句
修改表格中数据：

    UPDATE 表名称 SET 列名称 = 新值 WHERE 列名称 = 某值

Persons表：
LastName | FirstName | Address | City
---------|-----------|---------|-----
Gates | Bill | Xuanwumen 10 | Beijing
Wilson |   | Champs-Elysees |  

    UPDATE Person SET FirstName = 'Fred' WHERE LastName = 'Wilson' 

### DELETE语句

    DELETE FROM 表名称 WHERE 列名称 = 值

删除某行：
    DELETE FROM Person WHERE LastName = 'Wilson' 

    DELETE FROM table_name

    DELETE * FROM table_name