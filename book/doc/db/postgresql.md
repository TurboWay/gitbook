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

# 查看表大小
select pg_size_pretty(pg_relation_size('table_name'));

# 查看所有表大小
SELECT  table_schema || '.' || table_name AS table_full_name
      , pg_size_pretty(pg_total_relation_size('"' ||table_schema || '"."' || table_name || '"')) AS size
FROM information_schema.tables
ORDER BY pg_total_relation_size('"' ||table_schema || '"."' || table_name || '"') DESC limit 20

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
LEFT JOIN pg_namespace n on n.oid = c.relnamespace    
WHERE A.attnum > 0
AND n.nspname = 'public'   
AND C.relname = 't_gfecp_bigdata_blacklist'
ORDER BY
    C .relname,
    A .attnum
```
%/accordion%

%accordion%批量修改表所有者%accordion%

```sql
-- 批量修改 所有者
DO $$
DECLARE
    r record;
    i int;
    v_schema text[] := '{public}';
    v_new_owner varchar := 'gfecp_dev';
BEGIN
    FOR r IN
        SELECT 'ALTER TABLE "' || table_schema || '"."' || table_name || '" OWNER TO ' || v_new_owner || ';' AS a FROM information_schema.tables WHERE table_schema = ANY (v_schema)
        UNION ALL
        SELECT 'ALTER TABLE "' || sequence_schema || '"."' || sequence_name || '" OWNER TO ' || v_new_owner || ';' AS a FROM information_schema.sequences WHERE sequence_schema = ANY (v_schema)
        UNION ALL
        SELECT 'ALTER TABLE "' || table_schema || '"."' || table_name || '" OWNER TO ' || v_new_owner || ';' AS a FROM information_schema.views WHERE table_schema = ANY (v_schema)
--        UNION ALL
--        SELECT 'ALTER FUNCTION "' || nsp.nspname || '"."' || p.proname || '"(' || pg_get_function_identity_arguments(p.oid) || ') OWNER TO ' || v_new_owner || ';' AS a FROM pg_proc p JOIN pg_namespace nsp ON p.pronamespace = nsp.oid WHERE nsp.nspname = ANY (v_schema)
--        UNION ALL
--        SELECT 'ALTER DATABASE "' || current_database() || '" OWNER TO ' || v_new_owner
    LOOP
        EXECUTE r.a;
    END LOOP;
    FOR i IN array_lower(v_schema, 1)..array_upper(v_schema, 1)
    LOOP
        EXECUTE 'ALTER SCHEMA "' || v_schema[i] || '" OWNER TO ' || v_new_owner;
    END LOOP;
END
$$;
```
%/accordion%