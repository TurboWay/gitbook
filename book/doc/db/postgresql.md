[TOC]

## 常用命令

```shell
# 查看pg版本
pg_ctl -V

# 进入命令行
export PGPASSWORD=spider;psql -h 172.16.122.19 -U spider -d gfecp_dev

# 单表授权
grant all on area to gfecp_dev;

# 视图所有表授权
grant all on all tables in schema public to gfecp_dev;

# 修改表的拥有者
alter table area owner to gfecp_dev;

# 增加表默认值
alter table area alter column colname set default 456;

# 增加表默认值
alter table area alter column colname set default 456;

# 增加自增列
CREATE SEQUENCE 表名_id_seq
START WITH 1
INCREMENT BY 1
NO MINVALUE
NO MAXVALUE
CACHE 1;

alter table 表名 alter column id set default nextval('表名_id_seq');

# 建表设置自增列
create table 表名(
id serial)


# 服务启动
service postgresql start       # 启动
service postgresql stop       # 停止
service postgresql status    # 查看状态
```

## 常用sql

```sql
-- 查询表的所有字段
select column_name
from information_schema.columns
where table_schema='public'
and table_name='tablename'

-- 查看表锁
select a.locktype, a.database, a.pid, a.mode, a.relation, b.relname
from pg_locks a
join pg_class b on a.relation = b.oid
where b.relname = 'tablename';

-- 解决表锁
select pg_cancel_backend(pid) -- 取消事务
-- , pg_terminate_backend(pid) -- 终止事务
from pg_locks
where relation in (select oid from pg_class where relname='tablename');

-- 查看所有表名
select tablename
from pg_tables
where schemaname='gfecp'
and tablename <> 't_gfecp_bigdata_prod_accident'
and tablename like 't_gfecp_bigdata_prod_accident%'
order by 1

--完全复制表，包括约束、索引、字段注释
CREATE TABLE "gfecp_dev"."tmp_t_gfecp_bigdata_pdp_apply" ( LIKE "gfecp_dev"."t_gfecp_bigdata_pdp_apply" INCLUDING DEFAULTS INCLUDING CONSTRAINTS INCLUDING INDEXES INCLUDING COMMENTS);
```

## 数据字典

%accordion%查询字典%accordion%

```sql
SELECT
    A .attname 字段,   
    concat_ws (
        '',
        T .typname,
        SUBSTRING (
            format_type (A .atttypid, A .atttypmod)
            FROM
                '\(.*\)'
        )
    ) AS 类型,
        case when s.pk is not null then '是'
                               else '否'
    end as 主键,
    case A.attnotnull when 'f' then '是'
                      when 't' then '否'
    end as 空,
    d.description 注释
FROM pg_attribute A
INNER JOIN pg_class C on A .attrelid = C .oid
INNER JOIN pg_type T on A .atttypid = T .oid
LEFT JOIN (SELECT conrelid, unnest(conkey) as pk
                        FROM pg_constraint
                        WHERE contype = 'p') S ON S.conrelid = C .oid
                                   AND A.attnum = S.pk
LEFT JOIN pg_description d on d.objoid = A .attrelid
                                                    AND d.objsubid = A .attnum
WHERE A .attnum > 0
AND C.relname = 'tbname'
ORDER BY
    C .relname,
    A .attnum
```
%/accordion%