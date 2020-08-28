[TOC]

# 1、常用命令

| 命令                                                 | 说明               |
| :--------------------------------------------------- | :----------------- |
| hive --version                                       | 查看hive版本       |
| select version()                                     | 查看hive版本       |
| show databases                                       | 查看数据库         |
| show tables like 'test_*'                            | 查看表             |
| show functions                                       | 查看函数           |
| show create table tbname                             | 查看建表语句       |
| show partitions tbname                               | 查看分区           |
| desc tbname                                          | 查看表字段注释     |
| desc extended tbname                                 | 查看表字段注释     |
| desc formatted tbname                                | 查看表字段注释     |
| show locks extended                                  | 查看锁             |
| unlock table tbname                                  | 释放锁             |
| msck repair table tbname                             | 刷新所有分区元数据 |
| drop database db_name cascade                        | 删表删库           |
| hive -hiveconf bizdate=20180101 -f  test.hql         | 传参执行 hql 文件  |
| ANALYZE TABLE tablename COMPUTE STATISTICS -- noscan | 更新统计信息       |
| alter table tablename set FILEFORMAT orc | 修改表存储格式  |


# 2、常用调优设置

```sql
-- 启用本地模式
set hive.exec.mode.local.auto=true;
set hive.exec.mode.local.auto.inputbytes.max=52428800;
set hive.exec.mode.local.auto.input.files.max=10;
```

```sql
-- hive开启动态分区写入
set hive.exec.dynamici.partition=true;
set hive.exec.dynamic.partition.mode=nonstrict;
set hive.exec.max.dynamic.partitions=10000;
set hive.exec.max.dynamic.partitions.pernode=10000;
```

```sql
-- 使用spark引擎
-- spark引擎参数
-- 内存占用 cores * memory * instances 
-- 核数占用 cores * instances + 1 (driver)
set hive.execution.engine=spark;
set spark.executor.cores=2;
set spark.executor.memory=1G;
set spark.executor.instances=8;
```

```sql
-- hive 不能直接设置 map 数，只能通过设置块大小间接实现控制 map 数
-- 合并输入端的小文件，减少map数
set mapred.max.split.size=256000000;
set mapred.min.split.size.per.node=256000000;
set mapred.min.split.size.per.rack=256000000;
set hive.input.format=org.apache.hadoop.hive.ql.io.CombineHiveInputFormat;
```

```sql
--设置内存缓冲区大小，很多时候可以解决内存不足问题
set io.sort.mb=10;

--最大可用内存
set mapreduce.map.java.opts=-Xmx2048m;
set mapreduce.reduce.java.opts=-Xmx2048m;

--join优化 数据量大时使用
set hive.auto.convert.join=true;

--设置任务数
set mapred.reduce.tasks=20;

--任务并行，会消耗更多资源
set hive.exec.parallel=true;
```


# 3、常用建表语句


%accordion% 内部表/分区表%accordion%

```sql
CREATE TABLE `t_gfecp_bigdata_admin_penalty`(
  `id` string COMMENT '主键',
  `case_name` string COMMENT '案件名称',
  `case_no` string COMMENT '行政处罚决定书文号',
  `penalty_name` string COMMENT '被处罚者',
  `penalty_type` string COMMENT '被处罚者类型：法人、个人、其他（这里一般指法人）',
  `authority` string COMMENT '执法部门',
  `penalty_date` string COMMENT '作出行政处罚的日期',
  `case_detail` string COMMENT '行政处罚决定书（正文）',
  `person_in_charge` string COMMENT '法定代表人或单位负责人',
  `notice_date` string COMMENT '公告日期',
  `severity` string COMMENT '非常严重、严重、一般',
  `detail_url` string COMMENT '处罚信息原文链接URL',
  `file_name` string COMMENT '附件名称',
  `file_id` string COMMENT '附件id',
  `spider_date` string COMMENT '数据采集日期',
  `site_id` string COMMENT '网站id',
  `enterprise_id` string COMMENT '企业ID：当前非必填，先用“被处罚者”与企业表的“企业名称”做关联',
  `insert_user` string COMMENT '创建者',
  `insert_time` string COMMENT '创建时间',
  `update_user` string COMMENT '修改者',
  `update_time` string COMMENT '修改时间',
  `data_dt` string COMMENT '数据日期',
  `keyid` string COMMENT '',
  `all_item` string COMMENT '正文带标签',
  `spider_name` string COMMENT '爬虫名称',
  `file_path` string COMMENT '文件路径',
  `area_name` string COMMENT '',
  `site_desc` string COMMENT '')
COMMENT '行政处罚结果信息'
PARTITIONED BY (
  `bizdate` string COMMENT '业务日期')
```
%/accordion%


%accordion% 内部表/指定分隔符 %accordion%

```sql
CREATE TABLE `edw.f_gf_check_json`(
  `bizdate` string COMMENT '业务日期',
  `detail_url` string COMMENT '详情地址',
  `content` string COMMENT 'json串内容')
PARTITIONED BY (
  `check_tab_name` string COMMENT '表名',
  `check_site_id` string COMMENT '站点id')
row format delimited
fields terminated by '\001'
lines terminated by '\n'
```
%/accordion%


%accordion% 内部表/snappy 压缩%accordion%

```sql
CREATE EXTERNAL TABLE `spider.zhifang_list`(
  `keyid` string,
  `tit0` string COMMENT '字段注释',
  `tit1` string COMMENT '字段注释',
  `txt` string COMMENT '字段注释',
  `price` string COMMENT '字段注释',
  `name` string COMMENT '字段注释',
  `detail_url` string COMMENT '字段注释',
  `pkey` string COMMENT '等于 zhifang_detail.fkey',
  `pagenum` string COMMENT '下单页码',
  `sitename` string COMMENT '站点名称',
  `bizdate` string COMMENT '业务日期',
  `ctime` string COMMENT '入库时间',
  `spider` string COMMENT '爬虫名称')
STORED AS Parquet -- 或者 orc
TBLPROPERTIES ("orc.compress"="SNAPPY");
```
%/accordion%


%accordion%外部表/hbase%accordion%

```sql
CREATE EXTERNAL TABLE edw.test_turboway_hbase(
  `keyid` string COMMENT 'from deserializer',
  `title` string COMMENT 'from deserializer',
  `bizdate` string COMMENT 'from deserializer',
  `loginid` string COMMENT 'from deserializer')
ROW FORMAT SERDE
  'org.apache.hadoop.hive.hbase.HBaseSerDe'
STORED BY
  'org.apache.hadoop.hive.hbase.HBaseStorageHandler'
WITH SERDEPROPERTIES (
  'hbase.columns.mapping'=':key,cf:title,cf:bizdate,cf:loginid',
  'serialization.format'='1')
```
%/accordion%


%accordion%外部表/elasticsearch %accordion%

```sql
--根据 id 追加更新，无法删除
add jar hdfs:/user/hive/es_hadoop/elasticsearch-hadoop-5.4.3/dist/elasticsearch-hadoop-hive-5.4.3.jar;

--es_行政处罚结果信息
DROP TABLE IF EXISTS edw.f_gf_ent_penalty_info_es;
CREATE EXTERNAL TABLE IF NOT EXISTS edw.f_gf_ent_penalty_info_es(
id  string  comment '技术主键'
,case_name  string  comment '案件名称'
,case_no  string  comment '行政处罚决定书文号'
,penalty_name  string  comment '被处罚者'
,penalty_type  string  comment '被处罚者类型'
,authority  string  comment '执法部门'
,authority_clearing_text  string  comment '执法部门_清洗_text'
,authority_clearing_keyword  string  comment '执法部门_清洗_keyword'
,penalty_date  string  comment '行政处罚日期'
,case_detail  string  comment '行政处罚决定书'
,person_in_charge  string  comment '负责人'
,notice_date  string  comment '公告日期'
,detail_url  string  comment '详情链接'
,file_name  string  comment '附件名称'
,file_id  string  comment '附件id'
,corp_matching_degree  string  comment '企业信息匹配度'
,spider_date  string  comment '数据采集日期'
,bizdate  string  comment '业务日期'
,site_id string comment'网站id'
,data_source_area string comment'数据来源地区'
,data_source  string  comment '数据来源说明'
,data_sort_num  string  comment '数据排序号'
,del_flag  string  comment '数据逻辑删除标志'
,etldate  string  comment '数据加载更新日期'
)comment 'es_行政处罚结果信息'
STORED BY 'org.elasticsearch.hadoop.hive.EsStorageHandler'
TBLPROPERTIES('es.resource' = 'idx_f_gf_ent_penalty_info/f_gf_ent_penalty_info',
'es.nodes'='172.16.123.80 ',
'es.port'='8200',
'es.mapping.id' = 'id',
'es.write.operation'='upsert',
'es.net.http.auth.user'='elastic',
'es.net.http.auth.pass'='changeme'
);
```
%/accordion%


# 4、常用语句

```sql
-- 加载数据文件，相当于直接 put 覆盖到指定 hdfs 目录
LOAD DATA LOCAL INPATH '/home/getway/tmp/way/data_test.txt'
OVERWRITE  INTO TABLE spider.test_way_20200818 ;
```

```sql
-- 创建临时表
create temporary table as select id from tb;

-- 分区表添加字段，需要使用 CASCADE
alter table test20200415 add columns (name string, age string) CASCADE;

-- hive删除外部表数据
ALTER TABLE xxx SET TBLPROPERTIES('EXTERNAL'='False'); drop table xxx;

-- hive正则查找替换, 注释使用 \\
select regexp_replace(regexp_extract('asdas.doc','.docx|.xlsx|.xls|.pdf|.doc',0),'\\.','')

-- hive截取
SELECT SUBSTRING("6.5-8.5",INSTR("6.5-8.5",'-')+1,length("6.5-8.5")-INSTR("6.5-8.5",'-')),
SUBSTRING("6.5-8.5",0,INSTR("6.5-8.5",'-')-1)
```

```sql
-- 侧视图 完成 列转行
with tt as (
select '0001/0002/0003' as file_id, '1' as key
union all select '0004' as file_id, '2' as key
)
select *
from tt
lateral view explode(split(file_id,'/'))  b AS col5
```

```sql
-- 行合并
with tt AS(
select 'aaa' as a,'aaa' as b
union all select '1234' as a,'cccc' as b
)
select concat_ws(',',collect_set(a)) as ua,  concat_ws(',',collect_set(b)) as ub
from tt
```

```sql
-- json 解析函数 只有hive支持，impala不支持
-- get_json_object
with json_test as (
select '{"message":"2015/12/08 09:14:4","server": "passport.suning.com","request": "POST /ids/needVerifyCode HTTP/1.1"}' as js
)
select get_json_object(js,'$.message'), get_json_object(js,'$.server') from json_test;

-- json_tuple
with json_test as (
select '{"message":"2015/12/08 09:14:4","server": "passport.suning.com","request": "POST /ids/needVerifyCode HTTP/1.1"}' as js
)
select a.* 
from json_test
lateral view json_tuple(js,'message','server','request') a as f1,f2,f3;
```

# 5、元数据

```sql
--更新统计信息
ANALYZE TABLE tablename COMPUTE STATISTICS -- noscan

--hive元数据查看
select * from(
select a.TBL_NAME,
        sum(case when param_key='numRows' then  param_value else 0 end) 'rownum',
        sum(case when param_key='numRows' then  1 else 0 end) 'part_num' ,
        sum(case when param_key='totalSize' then  param_value else 0 end)/1024/1024/1024 'totalSize',
        sum(case when param_key='numFiles' then  param_value else 0 end) 'numFiles'
from TBLS a
left join TABLE_PARAMS b on a.TBL_ID = b.TBL_ID
where a.TBL_NAME not like '%_hbase'
group by a.TBL_NAME
) as a order by a.totalSize desc
```


```sql
-- 分区表，元数据不一致处理，统一更新
-- select  T1.TBL_NAME, T4.PART_NAME, T5.CD_ID, T3.CD_ID
-- from TBLS T1,DBS T2,SDS T3,PARTITIONS T4, SDS T5
UPDATE TBLS T1,DBS T2,SDS T3,PARTITIONS T4, SDS T5
SET T5.CD_ID = T3.CD_ID
WHERE T2.NAME = 'edw'
AND T1.TBL_NAME = 'test20200415'
AND T1.DB_ID = T2.DB_ID
AND T1.SD_ID = T3.SD_ID
AND T1.TBL_ID=T4.TBL_ID
AND T4.SD_ID = T5.SD_ID
and T5.CD_ID <> T3.CD_ID
```

# 6、UDF

%accordion%UDF 源代码 %accordion%

```python
#!/usr/bin/env python3
# -*- coding:utf-8 -*-
__author__ = "caiwei"
__date__ = "2018/11/12"

'''
更新为py3 update by cw 20190215 

Function:日期格式处理
例如：
20110101、01/01/2011、2011-01-01、二零一一年一月一日、2011.01.01、2011年1月1日 转换为 20110101
201101、01/2011、2011-01、二零一一年一月、2011.01、2011年1月 转换为 201101
2011、二零一一、2011年 转换为 2011
不合法日期 返回 [ERROR]原值
'''

import sys, datetime

CN_NUM = {
    '零' : 0, '一' : 1, '二' : 2, '三' : 3, '四' : 4, '五' : 5, '六' : 6, '七' : 7, '八' : 8, '九' : 9, '〇' : 0,
    '壹' : 1, '贰' : 2, '叁' : 3, '肆' : 4, '伍' : 5, '陆' : 6, '柒' : 7, '捌' : 8, '玖' : 9, '貮' : 2, '两' : 2,
    'Ｏ' : 0,
}

# 日期转换主函数
def todate(dt):
    dt = dt.strip()
    has_CN = check_CN(dt)
    date = strtodate(dt) if has_CN else inttodate(dt)
    return date

# 判断是否含有中文
def check_CN(dt):
    for i in dt:
        if CN_NUM.get(i):
            return True
    return False

# 中文日期格式处理
def strtodate(dt):
    """
    :param dt: 中文日期 格式：二零一一年一月一日、二零一一年一月
    :return: date 格式：20110101
    """
    date = dt
    try:
        dt, st = dt.strip(), []
        if '日' in dt:
            dt = dt.replace('日','')
            for ymd in ['年','月']:
                dt = dt.replace(ymd, '-')
            dt = dt.split('-')
        elif '月' in dt:
            dt = dt.replace('月', '')
            dt = dt.split('年')
        elif '年' in dt or len(dt) == 12:
            dt = dt.replace('年','')
            dt = [dt]

        for i in dt:
            s = 0
            if '十' in i:
                if len(i) == 1:
                    s = 10
                else:
                    a = i.split('十')
                    s = s + CN_NUM[a[0]] * 10 if a[0] else 10
                    if a[1]: s = s + CN_NUM[a[1]]
            elif len(i) == 1:
                s = '0' + str(CN_NUM[i])
            else:
                s = ''.join([str(CN_NUM[k]) for k in i])
            st.append(str(s))
        date = checkdate('-'.join(st))
    except :
        date = '[ERROR]' + date
    return date

# 数字日期格式处理
def inttodate(dt):
    """
    :param dt: 数字日期 格式：20110101、01/2011、2011-01、2011.01.01、2018.7.5、18-12-3、18.2.3
    :return: date 格式：2011-01-01
    """
    date = dt
    try:
        if '年' not in dt and ('月' in dt or '日' in dt): # 没有年份的混搭当做异常
            raise RuntimeError()
        dt = dt.replace('年','-').replace('月','-').replace('日','')
        if dt[-1] == '-': dt = dt[:-1]
        dt = dt[:10].replace('.','-').replace('．','-').strip()
        if '-' in dt:
            dt = dt.split('-')
            if len(str(dt[0])) == 2: # yy-mm-dd 中 yy大于60 为20xx 否则 19xx
                dt[0] = '19' + str(dt[0]) if int(dt[0]) >= 60 else '20' + str(dt[0])
            for index, i in enumerate(dt, 0):
                if len(str(i)) == 1:
                    dt[index] = '0' + str(i)
            st = '-'.join(dt)
        elif '/' in dt:
            dt = dt.split('/')
            if len(dt[0]) < 4:
                dt.reverse()
            dt = (str(i) for i in dt)
            st = '-'.join(dt)
        elif len(dt) == 8:
            st = "{0}-{1}-{2}".format(dt[:4], dt[4:6], dt[6:])
        elif len(dt) == 6:
            if int(dt[:4]) >= 1960: # 前面4位大于1960 认为是yyyy-mm 否则 yy-mm-dd
                st = "{0}-{1}".format(dt[:4], dt[4:6])
            else:
                yyyy =  '19' + str(dt[:2]) if int(dt[:2]) >= 60 else '20' + str(dt[:2])
                st = "{0}-{1}-{2}".format(yyyy, dt[2:4], dt[4:6])
        else:
            st = dt
        date = checkdate(st)
    except :
        date = '[ERROR]' + date if date else date
    return date

# 检查日期合法性
def checkdate(dt):
    """
    :param dt: 数字日期 格式：2011、2011-01、2011-01-01
    :return: correctdate 格式：2011、2011-01、2011-01-01
    """
    dt = dt.split('-')
    if dt[0][0] == '0': # 年份不能以0开头
        raise RuntimeError()
    if len(dt) == 1: # 只有年
        year, month, day = int(dt[0]), 1, 1
        dt = datetime.datetime(year, month, day)
        correctdate = str(dt)[:4]
    elif len(dt) == 2: # 年月
        year, month, day = int(dt[0]), int(dt[1]), 1
        dt = datetime.datetime(year, month, day)
        correctdate = str(dt)[:7].replace('-','')
    else: # 年月日
        year, month, day = int(dt[0]), int(dt[1]), int(dt[2])
        dt = datetime.datetime(year, month, day)
        correctdate = str(dt)[:10].replace('-','')
    return correctdate

if __name__ == '__main__':
    for line in sys.stdin:
        cols = line.split('\t')
        for index, col in enumerate(cols, 0):
            if len(col) >= 3:
                if col[:3] == 'UDF':
                    cols[index] = todate(col[3:])
        cols = [col.replace('\n', '') for col in cols]
        sys.stdout.write('\t'.join(cols) + '\n')
```
%/accordion%


%accordion%hive 调用 udf%accordion%

```sql
add file hdfs:/user/hive/udf/dateformatUDF.py;

with tt as (
select '20181025' as dt, '测试1' as test
union all select '2018-10-25' as dt, '测试2' as test
union all select '26/10/2018' as dt, '测试3' as test
union all select '2018.10.25' as dt, '测试4' as test
union all select '二零一八年十月二十五日' as dt, '测试5'  as test
union all select '二〇一八年十月二十五日' as dt, '测试6'  as test
union all select '201810' as dt, '测试7' as test
union all select '2018-10' as dt, '测试8' as test
union all select '10/2018' as dt, '测试9' as test
union all select '2018.10' as dt, '测试10' as test
union all select '二零一八年十月' as dt, '测试11' as  test
union all select '二〇一八年十月' as dt, '测试12' as  test
union all select '不合法输入' as dt, '测试13' as test
union all select '2018.10.32' as dt, '测试14' as test
union all select '2018-02-29' as dt, '测试15' as test
union all select '32/10/2018' as dt, '测试16' as test
union all select '二零一八年十月三十二日' as dt, '测试17' as test
union all select '13/2018' as dt, '测试18' as test
union all select '2018-13' as dt, '测试19' as test
union all select '二零一八年十三月' as dt, '测试20' as  test
union all select '二〇一八年' as dt, '测试21' as test
union all select '二零一八' as dt, '测试22' as test
union all select '2018' as dt, '测试23' as test
union all select '' as dt, '测试24' as test
union all select NULL as dt, '测试25' as test
union all select '00050203' as dt, '测试26' as test
union all select '2018-11-19 19:13:51' as dt, '测试27'  as test
union all select '2018-1-15' as dt, '测试28' as test
union all select '2018-7-9' as dt, '测试29' as test
union all select '18.5.6' as dt, '测试30' as test
union all select '69.12.6' as dt, '测试31' as test
union all select '201111' as dt, '测试32' as test
union all select '195901' as dt, '测试33' as test
union all select '170203' as dt, '测试34' as test
union all select '560203' as dt, '测试35' as test
union all select '2018年11月28日' as dt, '测试36' as  test
union all select '2018年11月' as dt, '测试37' as test
union all select '2018年11月28' as dt, '测试38' as test
union all select '2018年11' as dt, '测试39' as test
union all select '2018年13月' as dt, '测试40' as test
)
SELECT transform(dt, concat('UDF',dt), test)
USING 'python3 dateformatUDF.py' AS (dt, newdt, test)
FROM tt;
```
%/accordion%