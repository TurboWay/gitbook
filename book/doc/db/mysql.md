[TOC]

## 注意点

- 注释：

  "-- 注释" 需要加空格

- 反引号、单引号、双引号

  ```sql
  -- 使用关键字作为字段名、表名时，需要用反引号
  create table desc   -- 报错 
  create table `desc` -- 成功
  -- 使用字符串需要用 单引号 或者 双引号
  select `way`        -- 报错
  select 'way', "way"  -- 成功
  ```

- 大小写 
    ```sql
  SELECT 'a' = 'A'  -- 默认不区分大小写
  SELECT BINARY 'a' = 'A'  -- 区分大小写 
  ```


## 存储引擎

```sql
-- 查看引擎说明
show engines 
```

| 名称     |  适用说明    |
| :---- | :---- |
|InnoDB | 默认引擎，支持事务|
|MyISAM  |  不支持事务，大数据量|
|MEMORY  |  存在内存，常做临时表，用完记得删除|
|Archive   |  适合作为日志 |
|blackhole  |  可以用来备份表结构，备份，主从库之间缓冲|



## 数据文件

| 类型     |  级别   |  说明  |
| :---- | :---- | :---- |
|ibdata1  |    服务器级别 |  InnoDB 数据文件|
|ib_logfile0 | 服务器级别   |InnoDB 日志文件|
|ib_logfile1 | 服务器级别   |InnoDB 日志文件|
|.opt  | 数据库级别 | 指定默认字符集和排序规则，在该库建表时的缺省定义值|
|.frm  | 表级别  | 存储表定义|
|.ibd |  表级别 |  InnoDB单独一个表的数据及索引内容|
|.MYD  | 表级别 |  MyISAM数据文件|
|.MYI |  表级别 |  MyISAM索引文件|



## 常用函数

| 函数 |  说明  |
| :---- | :---- |
|LENGTH |  字符串长度 |
|CHAR_LENGTH |  字符串字符个数 |
|concat('a','b','c') |  合并字符串(遇到null,结果变成null) |
|concat_ws('-','a','b','c') |  使用分隔符，合并字符串(遇到null,无视) |
|insert('haha',1,2,'t')   |  替换字符串 |
|replace('haha','a','t') |  替换字符串 |
|repeat('ha',3)  |  重复生成相同字符串 |
|space(n)  |  重复n个空格组成的字符串 |
|mid('abhdg',2,3)  |  截取字符串 |
|SUBSTRING('abhdg',2,3) |  截取字符串 |
|locate(str1,str) |  字符串开始位置 |
|position(str1 IN str)    |  字符串开始位置 |
|instr(str,str1) |字符串开始位置|
|CURDATE(),CURRENT_DATE(),CURDATE()+0|获取当前日期|
|CURTIME(),CURRENT_TIME(),CURTIME()+0 |获取当前时间|
|CURRENT_TIMESTAMP(),LOCALTIME(),NOW(),SYSDATE()|获取当前日期时间|
|PASSWORD(明文)|系统用户加密|
|MD5(明文)|md5加密|
|ENCODE(明文, 盐)|加密|
|DECODE(密文, 盐)|解密|
|INET_ATON('192.168.100.45')|IP地址转数字|
|INET_NTOA(3232261165)|数字转IP地址|



## 常用命令

```sql
-- 自增列：auto_increment PRIMARY KEY（必须作为主键的一部分） 相当于 SQLSERVER 的 IDENTITY(1,1)

--查看表结构
DESC TableName    

-- 修改表名
alter table 旧表名 rename 新表名                        

-- 修改字段数据类型 
alter table 表名 modify 字段名 数据类型

-- 修改字段名
alter table 表名 change 旧字段名 新字段名 数据类型

-- 添加字段，语法同SQLserver
alter table 表名 add 字段名 数据类型【约束】 

-- 删除字段：语法同SQLserver
alter table 表名 drop column 字段名

-- 添加字段（指定位置）
alter table 表名 add 字段名 数据类型【约束】 First ; -- 在表的第一列添加字段
alter table 表名 add 字段名 数据类型【约束】 After AA ; -- 在表的AA列后面字段

-- 修改字段位置：
alter table 表名 modify DD 数据类型 First ; -- 把DD移到表的第一列
alter table 表名 modify DD 数据类型 After AA ; -- 把DD移到AA列后面

-- 修改表的存储引擎
alter table 表名 ENGINE = MyISAM

--删除表（表不存在，不报错，会有警告 show warnings）
drop table if exists aa,aa1... 

-- 删除表： 语法同SQLserver
drop table aa,aa1...

-- 正则表达式
select '45' REGEXP '[0-9]', '45' REGEXP '\d'
```

建表 demo
```sql
CREATE TABLE `weibo_get` (
  `ID` int(11) auto_increment PRIMARY KEY,    -- ID自增列
  `WeiboName` varchar(50) DEFAULT NULL,       -- 微博号昵称
  `Title` varchar(300) DEFAULT NULL,          -- 微博内容
  `ShortUrl` varchar(300) DEFAULT NULL,       -- 优惠券短链
  `TextApi` varchar(300) DEFAULT NULL,        -- 微博内容api链接(短时有效，内容会覆盖，仅供测试检查)
  `CommentApi` varchar(300) DEFAULT NULL,     -- 评论内容api链接(短时有效，内容会覆盖，仅供测试检查) 
  `AddTime` datetime DEFAULT NOW()            -- 添加时间 
) ENGINE=InnoDB DEFAULT CHARSET=utf8;
```

## 运维命令

```sql
# 备份数据库
mysqldump -uroot -h 127.0.0.1 -p izone > izone.sql

# 还原数据库
mysql -uroot -pPASSWORD izone < izone.sql
```


## 索引、存储过程、事件

```
1.存储过程

修改mysql结束符(默认;)：DELIMITED //    （在CMD命令行有效，navicat无效）

MySQL还不提供对已存在的存储过程的代码修改，只能先删除再新建 


2.调用过程

call TEST(10)  相当于SQLSERVER  exec TEST 10   


3.索引(和SQLSERVER有较大区别，没有聚集/非聚集的区分)

普通索引和唯一索引
单列索引和组合索引
全文索引和空间索引(只支持MYISAM存储引擎)


4.符号问题

反引号一般在Esc键的下方，和～在一起。
它是为了区分MySQL的保留字与普通字符而引入的符号。 
create table desc 报错 
create table `desc` 成功
一般我们建表时都会将表名、字段名、索引名、库名都加上反引号来保证语句的执行度。

字符串使用的是单引号。


5.触发器

 OLD/NEW 和 SQLSERVER中的 INSERTED 和 DELETED 类似

 可以建立6种触发器，即：BEFORE INSERT、BEFORE UPDATE、BEFORE DELETE、AFTER INSERT、AFTER UPDATE、AFTER DELETE。
 另外有一个限制是不能同时在一个表上建立2个相同类型的触发器，因此在一个表上最多建立6个触发器。(看起来触发器，比SQLSERVER强大)

6.事件

  类似 SQLSERVER 的作业调度
```




