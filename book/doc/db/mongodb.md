[TOC]

mongodb  对大小写敏感
数据库文件：BSON，JSON

## 常用命令

|  命令    |   说明   |
| :---- | :---- |
|show dbs    | 查看所有数据库     |
|show tables      | 查看所有表     |
|db     | 查看当前数据库     |
|use dbname      | 创建数据库     |
|db.dropDatabase()       |  删除当前数据库    |
|mongostat      |查看用户、进程、锁...      |
|mongotop      | 查看读写情况     |
| //     | 行注释     |



## 常用操作

| 操作 | 格式 | 范例 | RDBMS中的类似语句 |
| :--- | :---- | :---- | :----------------- |
|等于|{<key>:<value>}	| db.col.find({"by":"菜鸟教程"}).pretty() 	|where by = '菜鸟教程'|
|小于| {<key>:{$lt:<value>}}|db.col.find({"likes":{$lt:50}}).pretty()	|where likes < 50|
|小于或等于|	{<key>:{$lte:<value>}}	|db.col.find({"likes":{$lte:50}}).pretty()|	where likes <= 50|
|大于	 |  {<key>:{$gt:<value>}}|	db.col.find({"likes":{$gt:50}}).pretty()	|where likes > 50|
|大于或等于|	{<key>:{$gte:<value>}}|db.col.find({"likes":{$gte:50}}).pretty()|	where likes >= 50|
|不等于| {<key>:{$ne:<value>}}	|db.col.find({"likes":{$ne:50}}).pretty()|	where likes != 50 |




## 常用语句

```shell
# 增
db.tablename.insert({"name":"jack","age":11,"sex":0})
db.tablename.insert([{"name":"jack","age":11,"sex":0},{"name":"lucy","age":11,"sex":0},{"name":"luff","age":11,"sex":0}])

# 插入一条数据
var document = db.tablename.insertOne({"name":"jack","age":11,"sex":0})
document
# 批量插入数据：
var res = db.tablename.insertMany([{"a":"hello"},{"s":"world"},{"e":"!!"}])
res

# 删
db.tablename.remove({"name":"jack"})  
db.tablename.remove({})                

# 改
db.tablename.update({"name":"jack","age":11,"sex":0},{"name":"way"})  

# 查
db.tablename.find({"name":"jack"})  
db.tablename.find()                   

# 计数
db.tablename.count()

# 删表
db.tablename.drop()

# 聚合 select url,sum(1) where like=750 group by url
db.mycol.aggregate([{$match:{likes:750}}, {$group : {_id : "$url", sum : {$sum : 1}}}]) 

# 1 升序/-1 降序 SELECT top 4 * from hello where ID>=5 and ID<=6 order by id desc
db.hello.find({"ID":{$gte:5,$lte:6}}).limit(4).sort({"ID":-1}) 

# 逻辑且 where ID>=5 and ID<=6
db.hello.find({"ID":{$gte:5,$lte:6}})   

# 逻辑或 where ID=5 OR ID =4
db.hello.find({$or:[{"ID":5},{"ID":4}]})   

# 逻辑 且/或 并用 where ID>=5 and (ID=5 OR ID =4)
db.hello.find({"ID":{$gte:5},$or:[{"ID":5},{"ID":4}]})   

# 根据数据类型查询 1 double,2 string,9 date, 10 null ....
db.hello.find({"ID":{$type:1}})

# 索引创建
db.hello.ensureIndex({"ID":1,"Image":-1},{background: true})
```



## 运维命令

```shell
# 备份：备份的时候，会在指定路径下，新建一个同名文件夹
mongodump -h 127.0.0.1:27017 -d test -o F:\mongodb\data_bak
# 还原: 还原的时候，数据库名称可以重设
mongorestore -h 127.0.0.1:27017 -d test2 --dir F:\mongodb\data_bak\test 
```
