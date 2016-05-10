基本操作
========

数据库操作
-----------
显示数据库：show databases;
建数据库：create database  [if not exists] 数据库名;
建数据表： create table [if not exists] 表名 (字段名1   类型。。。。。。。。)
create table student (编号 int auto_increment primary key, 姓名 varchar(10));
注意:设置了自动增长,就要定为主键,如果选择了BIT 类型,0不显示,非0显示为一个特殊符号!
显示数据表:show tables;
删除库: drop database [if exists] 库名;
删除表: drop table   [if exists] 表名;
显示表结构: desc 表名
如何修改表结构:增长一个字段; alter table 表名 add 字段名  类型
删除一个字段: alter table  表名 drop 字段名

表操作
-------
插入
^^^^^

Insert into 表 (字段表列表) values（字段值），（字段值）。。。。。。

删除
^^^^^
删除一个主键  alter table 表名  drop primary key(字段名)

删除，更新和SQL SERVER没有什么区别，不再累述！

删除数据库：Drop DATABASE 数据库名

删除表：    Drop TABLE    表名

修改
^^^^^^
修改一个字段的属性:  alter table 表名modify 字段 新属性

修改主键: 增加一个主键  alter table 表名 add primary key(字段名)

表改名：    RENAME TABLE 旧表中  TO  新表名 数据库不能改名，但也不是绝对不能改，但改不好会造成里面的数据无法正常读出，后果自负！

查询
^^^^^
查找姓王的记录：Select  * FROM YUANGONG  Where 姓名  like '王%';

查找姓名中有五的记录：Select * FROM YUANGONG  Where 姓名  like '%王%';

查找以王结尾的记录：Select  * FROM YUANGONG  Where 姓名  like '%王';

第三条到第七条记录 select * from 表名 limit 2,5;

通配符 描述 示例
% 通配零个或多个任意字符 
_(下划线) 通配任意一个字符 
注意：如果用like发现结果不正确，有可能是编码的问题

排序
^^^^^

利用order by 对记录进行排序
格式：select 字段名列表 from 表名 [where 条件] order by 排序字段1 [asc ] [desc] [排序字段2……]

如：按年龄对yuangong表进行升序排列！
Select  * from yuangong order by 年龄  asc 或  select  * from yuangong order by 年龄

如：按年龄对yuangong表进行降序排列！
Select  * from yuangong order by 年龄  desc

对员工表先按性别升序排列，性别相同的再按年龄从大到小排序
Select * from 员工表   order by   性别 asc,年龄 desc 



设计原则
=========
