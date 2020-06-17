[TOC]

## SQL 基础

```sql
-- 增删改查：
insert into table_name values(值1,值2...)
insert into table_name（列1，列2...）values(值1，值2...) 

delete from table_name where 列名=某值
delete from table_name

update table_name set 列名=新值 where 列名=旧值

select * from table_name  
select distinct 列名称 from table_name order by 列名称 desc
select top 3 * from table_name
select top 50 percent * from table_name
select * from table_name where 列名称 like 'w%' or '%w' or '%lon%'

-- DDL：
-- 添加约束
alter table table_name add column_name datetype --not null
-- 添加主键
alter table table_name add primary key(列名) 
-- 添加外键
alter table table_name add foreign key(外键名) references table2_name(主键名) 
-- 删除字段
alter table table_name drop column column_name   
-- 更改字段类型
alter table table_name alter column column_name datetype  --not null
-- 设置默认值
alter column column_name set default 'AA'                     
-- 取消默认值
alter column column_name drop default  
-- 修改字段名
exec sp_rename '表名.原列名','新列名','column' 

-- 建表：

create table table_name(
   [列名1] 数据类型1（长度）-- not null --unique --primary key identity(20,10）/*20起，以10递增*/ ，/*可以添加约束、主键、外键*/
   [列名2] 数据类型2（长度），--foreign key references table2_name(主键名)
   。。。。）

--删除：  
delete from table_name                /*删除内容不删除定义，不释放空间。*/
drop database_name/table_name      /*删除内容和定义，释放空间。*/
truncate table table_name   /*删除内容、释放空间但不删除定义。*/*初始化*/

-- 注释符号: --  /* */

-- join
inner join 内连接 -- 只显示两表都存在的记录 记录数<=任一表
left join 左连接 -- 显示左表所有存在的记录 记录数=左表
right join 右连接 -- 显示右表所有存在的记录 记录数=右表
full join 外连接 -- 显示两表中，某个表有存在的记录 记录数>=任一表
cross join 交叉连接 -- 对左表的每条记录，都对应右表的每条记录 记录数=左表*右表

-- EXCEPT 是指在第一个集合中存在，但是不存在于第二个集合中的数据。
-- INTERSECT 是指在两个集合中都存在的数据
select * from a
except
select * from b

-- 建表压缩，在建表后面加如下语句：
WITH (DATA_COMPRESSION = ROW)         /*行压缩*/
WITH (DATA_COMPRESSION = PAGE)        /*页压缩*/

-- 开窗函数
ROW_NUMBER() OVER ( PARTITION BY [Part] ORDER BY Price DESC ) 
RANK() OVER ( PARTITION BY [Part] ORDER BY Price DESC ) 
DENSE_RANK() OVER ( PARTITION BY [Part] ORDER BY Price DESC ) 
NTILE(6) OVER ( PARTITION BY [Part] ORDER BY Price DESC ) 

-- 正则表达式：
-- PATINDEX 支持通配符，CHARINDEX 不支持通配符
SELECT  COUNT(1)
FROM    Dim_store
WHERE   PATINDEX('%[吖-做]%', storename) > 0
or PATINDEX('%[A-Z]%', storename) > 0
or PATINDEX('%[0-9]%', storename) > 0

----------------------------调优-----------------------------------------
-- 查看SQL耗时：

SET STATISTICS PROFILE ON 
SET STATISTICS IO ON 
SET STATISTICS TIME ON 
 /*--你的SQL脚本开始*/

SELECT [TestCase] FROM [TestCaseSelect] 

 /*--你的SQL脚本结束*/
SET STATISTICS PROFILE OFF 
SET STATISTICS IO OFF 
SET STATISTICS TIME OFF

----------------------------延时执行--------------------------------------
WAITFOR DELAY '00:00:03'
BEGIN
    PRINT 1
END

WAITFOR TIME '12:00:03'
BEGIN
    PRINT 1
END

--------------------------重置表的自增字段---------------------------------
DBCC CHECKIDENT (tablename,reseed,10)
```



## 常用函数

| 函数 | 说明 |
| :---- | :---- |
|ltrim(rtrim(column_name)) |去掉字段左右两边空格|
|cast(expression as datetype)  |转换数据类型|
|isnull(a,b)  |if a= null then b else a|
|COALESCE(NULL,NULL,NULL,NULL,123)  |返回第一个非空值|
|replace(1,2,3) | 用3替代1中的所有2 |
|substring(abcde,4,2) |  de |
|getdate()  |获取当前日期|
|datediff(datepart,startdate,endstart) |间隔时间|
|dateadd（datepart,number,date）|添加或减去指定的时间间隔（依number正负而定）|
|datepart(year,date) |用于返回日期/时间的单独部分，比如年、月、日、小时、分钟等等|
|day(date_expression)  |用于返回 日|
|month(date_expression) |用于返回 月|
|year(date_expression)  |用于返回 年|
|datetime2(N) | 精度比datetime高（可以精确到100纳秒），N是(0—7)，0精确到秒，3相当于datetime。|



## 常用 sql 脚本

%accordion%表数据字典%accordion%

```sql
/*查看表 描述*/
SELECT  OBJECT_NAME(a.major_id) 表名 ,
        b.name 列名 ,
        a.name 描述 ,
        a.value 字段说明
FROM    sys.extended_properties a
        LEFT JOIN sys.columns b ON a.minor_id = b.column_id
                                   AND a.major_id = b.object_id
WHERE a.major_id = OBJECT_ID('Usr_Contact')


/*描述的增删改*/

--为表添加描述信息
EXECUTE sp_addextendedproperty N'MS_Description', '人员信息表', N'user', N'dbo', N'table', N'表名', NULL, NULL

--为字段a1添加描述信息
EXECUTE sp_addextendedproperty N'MS_Description', '字段描述', N'user', N'dbo', N'table', N'表名', N'column', N'字段名'

--更新表中列a1的描述属性：
EXEC sp_updateextendedproperty 'MS_Description','字段描述','user',dbo,'table','表名','column',列名

--删除表中列a1的描述属性：
EXEC sp_dropextendedproperty 'MS_Description','user',dbo,'table','表名','column',列名
```
%/accordion%

%accordion%每张表数据量%accordion%

```sql
/*查看数据库表情况*/
IF OBJECT_ID('tempdb..#TB_TEMP_SPACE') IS NOT NULL 
    DROP TABLE #TB_TEMP_SPACE
GO
CREATE TABLE #TB_TEMP_SPACE
    (
      NAME VARCHAR(500) ,
      ROWS INT ,
      RESERVED VARCHAR(50) ,
      DATA VARCHAR(50) ,
      INDEX_SIZE VARCHAR(50) ,
      UNUSED VARCHAR(50)
    )
GO
SP_MSFOREACHTABLE 'INSERT INTO #TB_TEMP_SPACE exec sp_spaceused ''?''' 
GO
SELECT  Name ,
        Rows ,
        RESERVED ,
        DATA AS 'DATA  (KB)' ,
        CAST(CONVERT(FLOAT, LEFT(DATA, LEN(DATA) - 3)) / 1024 AS NUMERIC(18, 2)) AS 'DATA  (M)' ,
        CAST(CONVERT(FLOAT, LEFT(DATA, LEN(DATA) - 3)) / 1024 / 1024 AS NUMERIC(18,
                                                              2)) AS 'DATA  (G)' ,
        INDEX_SIZE ,
        UNUSED
FROM    #TB_TEMP_SPACE
ORDER BY 2 DESC 
GO 

```
%/accordion%

%accordion%如何知道 SQL 运行了多久%accordion%

```sql
DECLARE @ms_per_tick DECIMAL(10, 6)  
 --millisecond per tick  
SELECT  @ms_per_tick = 1.0 * DATEDIFF(millisecond, sqlserver_start_time,  
                                      GETDATE()) / ( ms_ticks  
                                                     - sqlserver_start_time_ms_ticks )  
FROM    sys.[dm_os_sys_info] ;  
  
--select @ms_per_tick  
  
SELECT  req.session_id ,  
        req.start_time request_start_time ,  
        ( ( SELECT  ms_ticks  
            FROM    sys.dm_os_sys_info  
          ) - workers.task_bound_ms_ticks ) * @ms_per_tick 'ms_since_task_bound' ,  
        DATEDIFF(ms, req.start_time, GETDATE()) 'ms_since_request_start' ,  
        tasks.task_state ,  
        workers.state worker_state ,  
        req.status request_state ,  
        st.text ,  
        SUBSTRING(st.text, ( req.statement_start_offset / 2 ) + 1,  
                  ( ( CASE req.statement_end_offset  
                        WHEN -1 THEN DATALENGTH(st.text)  
                        ELSE req.statement_end_offset  
                      END - req.statement_start_offset ) / 2 ) + 1) AS stmt ,  
        qp.query_plan ,  
        req.*  
FROM    sys.dm_exec_requests req  
        LEFT JOIN sys.dm_os_tasks tasks ON tasks.task_address = req.task_address  
        LEFT JOIN sys.dm_os_workers workers ON tasks.task_address = workers.task_address  
        CROSS APPLY sys.dm_exec_sql_text(req.sql_handle) st  
        CROSS APPLY sys.dm_exec_query_plan(req.plan_handle) qp  
WHERE   ( req.session_id > 50  
          OR req.session_id IS NULL  
        )  

```
%/accordion%

%accordion%收缩日志%accordion%

```sql
--设置要收缩的数据库的模式为简单模式
ALTER DATABASE [DC_HQ]
SET RECOVERY SIMPLE
GO
--将数据库日志文件收缩到1M
DBCC SHRINKFILE (DC_HQ_Log, 1)
GO
--恢复数据库模式为完整模式
ALTER DATABASE [DC_HQ]
SET RECOVERY FULL
GO
```
%/accordion%

%accordion% merge %accordion%

```sql
/*功能：根据与源表联接的结果，对目标表执行插入、更新或删除操作。
例如，根据在另一个表中找到的差异在一个表中插入、更新或删除行，可以对两个表进行同步。
http://www.cnblogs.com/downmoon/archive/2010/10/17/1853833.html */

--确定目标表
Merge Into Demo_AllProducts p

--从数据源查找编码相同的产品
using Demo_Shop1_Product s on p.DCode=s.DCode 

--如果编码相同，则更新目标表的名称
When Matched and P.DName<>s.DName Then Update set P.DName=s.DName

--如果目标表中不存在，则从数据源插入目标表
When Not Matched By Target Then Insert (DName,DCode,DDate) values (s.DName,s.DCode,s.DDate)

--如果数据源的行在源表中不存在，则删除目标表行
When Not Matched By Source Then Delete;
```
%/accordion%

%accordion%错误捕捉 try catch%accordion%

```sql
SET XACT_ABORT ON     --可能某些错误使得事务中断，并且没有被Catch到，导致事务没有提交回滚造成堵塞死锁。开启此选项可以自动回滚，会话域，默认OFF。

                      --查看XACT_ABORT选项 SELECT (CASE WHEN (16384 & @@OPTIONS) = 16384 THEN 'ON' ELSE 'OFF' END) AS XACT_ABORT;  
BEGIN TRAN
BEGIN TRY

    INSERT  INTO [Way].[dbo].[tb_Tel]
            SELECT  'b' ,
                    'a' ,
                    'a' ,
                    'x'
    INSERT  INTO [Way].[dbo].[tb_Tel]
            SELECT  'b' ,
                    'a' ,
                    'a' ,
                    'z'
    --INSERT INTO BB SELECT 1 
      
    COMMIT TRAN
    
END TRY
BEGIN CATCH
    SELECT  ERROR_NUMBER() AS 返回错误号 ,
            ERROR_SEVERITY() AS 返回严重性 ,
            ERROR_STATE() AS 返回错误状态号 ,
            ERROR_PROCEDURE() AS 返回出错误的存储过程或触发器的名称 ,
            ERROR_LINE() AS 返回导致错误的例程中的行号 ,
            ERROR_MESSAGE() AS 返回错误消息的完整文本该文本  
    ROLLBACK       
         
END CATCH
```
%/accordion%




## 事务

|  事务类型    |  说明    |
| :---- | :---- |
|自动提交事务 Autocommit Transactions | SQL Server 默认事务类型，每一条单独的语句都是一个事务, 执行完自动提交 |
|显示事务 Explicit Transactions | (调用API或者Begin Transcation,Rollback 或者 Commit |
| 隐式事务 Implicit Transactions | 执行语句时自动打开事务，Rollback Transaction 或者 Commit Transaction |

设置事务类型
```sql
set Implicit Transactions/Explicit Transactions on/off
```



## 事务隔离级别

隔离级别越高，读操作的请求锁定就越严格，锁的持有时间久越长；
所以隔离级别越高，一致性就越高，并发性就越低，同时性能也相对影响越大。


|  隔离级别    |  脏读    | 不可重复读   |  幻读  | 说明 |
| :---- | :---- | :---- | :---- |:---- |
|未提交读(read uncommitted)|	是	|是	|是	|如果其他事务更新，不管是否提交，立即执行|
|提交读(read committed 默认)|	否	|是	|是	|读取提交过的数据。如果其他事务更新没提交，则等待|
|可重复读(repeatable read)	|否	|否	|是	|查询期间，不允许其他事务update|
|可串行读(serializable)	     |   否	|否	|否	|查询期间，不允许其他事务insert或delete|

```sql
-- 获取数据库事务隔离级别
DBCC USEROPTIONS
-- 更改事务隔离级别(作用域 会话)
SET TRANSACTION ISOLATION LEVEL read uncommitted
SET TRANSACTION ISOLATION LEVEL read committed
SET TRANSACTION ISOLATION LEVEL repeatable read
SET TRANSACTION ISOLATION LEVEL serializable 
-- 查询表隔离
SELECT ....FROM <TABLE> WITH (read uncommitted) 
SELECT ....FROM <TABLE> WITH (read committed) 
SELECT ....FROM <TABLE> WITH (repeatable read) 
SELECT ....FROM <TABLE> WITH (serializable) 
```

%accordion%监控当前正在运行的事务%accordion%

```sql
SELECT  ST.transaction_id AS TransactionID ,
        DB_NAME(DT.database_id) AS DatabaseName ,
        AT.transaction_begin_time AS TransactionStartTime ,
        DATEDIFF(SECOND, AT.transaction_begin_time, GETDATE()) AS TransactionDuration ,
        CASE AT.transaction_type
          WHEN 1 THEN 'Read/Write Transaction'
          WHEN 2 THEN 'Read-Only Transaction'
          WHEN 3 THEN 'System Transaction'
          WHEN 4 THEN 'Distributed Transaction'
        END AS TransactionType ,
        CASE AT.transaction_state
          WHEN 0 THEN 'Transaction Not Initialized'
          WHEN 1 THEN 'Transaction Initialized & Not Started'
          WHEN 2 THEN 'Active Transaction'
          WHEN 3 THEN 'Transaction Ended'
          WHEN 4 THEN 'Distributed Transaction Initiated Commit Process'
          WHEN 5 THEN 'Transaction in Prepared State & Waiting Resolution'
          WHEN 6 THEN 'Transaction Committed'
          WHEN 7 THEN 'Transaction Rolling Back'
          WHEN 8 THEN 'Transaction Rolled Back'
        END AS TransactionState
FROM    sys.dm_tran_session_transactions AS ST
        INNER JOIN sys.dm_tran_active_transactions AS AT ON ST.transaction_id = AT.transaction_id
        INNER JOIN sys.dm_tran_database_transactions AS DT ON ST.transaction_id = DT.transaction_id
ORDER BY TransactionStartTime  
```
%/accordion%

%accordion%查看阻塞查询%accordion%

```sql
SELECT  R.session_id AS BlockedSessionID ,  
        S.session_id AS BlockingSessionID ,  
        Q1.text AS BlockedSession_TSQL ,  
        Q2.text AS BlockingSession_TSQL ,  
        C1.most_recent_sql_handle AS BlockedSession_SQLHandle ,  
        C2.most_recent_sql_handle AS BlockingSession_SQLHandle ,  
        S.original_login_name AS BlockingSession_LoginName ,  
        S.program_name AS BlockingSession_ApplicationName ,  
        S.host_name AS BlockingSession_HostName  
FROM    sys.dm_exec_requests AS R  
        INNER JOIN sys.dm_exec_sessions AS S ON R.blocking_session_id = S.session_id  
        INNER JOIN sys.dm_exec_connections AS C1 ON R.session_id = C1.most_recent_session_id  
        INNER JOIN sys.dm_exec_connections AS C2 ON S.session_id = C2.most_recent_session_id  
        CROSS APPLY sys.dm_exec_sql_text(C1.most_recent_sql_handle) AS Q1  
        CROSS APPLY sys.dm_exec_sql_text(C2.most_recent_sql_handle) AS Q2  
        
-- kill 正在阻塞的进程ID/会话ID
```
%/accordion%



## 索引

1. 没有聚集索引的表被称为堆，拥有聚集索引的表叫聚集索引表

2. 索引碎片
当修改表(UPDATE、INSERT、DELETE等)中数据，数据库引擎自动维护索引的数据和结构。但是随着修改次数的累积，可能会现：
  
   - 外部碎片
     索引中记录的数据顺序（逻辑顺序）和数据的实际顺序不一致（物理顺序）
  
   - 内部碎片
  
     索引页的数据填充度变小（页密度）
  
3. 索引的数据存储方式 B-Tree
   - 聚集索引          稀疏索引    物理顺序    叶子节点即数据结点     某页
   - 非聚集索引        密集索引    逻辑顺序    叶子节点并非数据结点   指向某一页 某一行

4. 索引类型
   - 唯一索引----加了unique的索引（唯一约束+索引)
   - 聚集索引 >  主键（主键是约束CONSTRAINT，非空+唯一，同时生成聚集索引）
   - 非聚集索引

5. 索引优化：

| 类型       | 说明     | 优化方法       |
| :--------- | :------- | :--- |
| 索引不合理 | 无用索引 | 删除无用索引   |
| 索引过多   | 合并索引 | 删除合并  |
| 索引缺失   | 增加索引 | 新增   |
| 索引碎片   | 索引维护 | 重建、重组索引 |

```sql
/*创建索引*/

--创建主键
CREATE TABLE Skills (
   SkillID INTEGER NOT NULL,
   SkillName CHAR( 20 ) NOT NULL,
   SkillType CHAR( 20 ) NOT NULL,
   PRIMARY KEY( SkillID )
)   
ALTER TABLE [TEST].[dbo].[Bi_code] ADD PRIMARY KEY (code)

--聚集索引
CREATE CLUSTERED INDEX PK_code ON [TEST].[dbo].[Bi_code](code)
--非聚集索引 覆盖索引
CREATE NONCLUSTERED INDEX PK_code2 ON [TEST].[dbo].[Bi_code](code,tbkey)
--非聚集索引 包含索引
CREATE NONCLUSTERED INDEX PK_code3 ON [TEST].[dbo].[Bi_code](code) INCLUDE (tbkey)
--唯一索引
CREATE UNIQUE NONCLUSTERED INDEX PK_code4 ON [TEST].[dbo].[Bi_code](code) 
CREATE UNIQUE CLUSTERED INDEX PK_code5 ON [TEST].[dbo].[Bi_code](code) 

/*删除索引*/
DROP INDEX PK_code ON [TEST].[dbo].[Bi_code]
ALTER TABLE [dbo].[Bi_code] DROP CONSTRAINT [PK__Bi_code__357D4CF84A8310C6]

/*重组索引*/

--指定重组
ALTER INDEX PK_code5 ON [dbo].[Bi_code] REORGANIZE  
--全部重组
ALTER INDEX ALL ON [dbo].[Bi_code] REORGANIZE 

/*重建索引*/

--使用联机方式重建索引  
ALTER INDEX PK_code5 ON [dbo].[Bi_code] REBUILD WITH (FILLFACTOR=80,ONLINE =ON)  
  
--使用脱机方式重建索引  
ALTER INDEX PK_code5 ON [dbo].[Bi_code] REBUILD WITH (FILLFACTOR=80,ONLINE =OFF)  
  
--使用脱机方式重建表上所有索引  
ALTER INDEX ALL ON [dbo].[Bi_code] REBUILD WITH (FILLFACTOR=80,ONLINE =OFF )  
 
--使用DROP_EXISTING 来重建索引 
CREATE CLUSTERED INDEX PK_code5 ON [Bi_code](code) WITH (DROP_EXISTING=ON ,FILLFACTOR=70,ONLINE=ON ) 
```







