开启数据库

```bash
pg_ctl start -D $PGDATA
```

关闭数据库

```bash
 pg_ctl stop -D $PGDATA
```

开启psql

```bash
psql
```

查看有哪些数据库

```
yby=# \l
                          List of databases
   Name    | Owner | Encoding | Collate |  Ctype  | Access privileges
-----------+-------+----------+---------+---------+-------------------
 postgres  | yby   | UTF8     | C.UTF-8 | C.UTF-8 |
 template0 | yby   | UTF8     | C.UTF-8 | C.UTF-8 | =c/yby           +
           |       |          |         |         | yby=CTc/yby
 template1 | yby   | UTF8     | C.UTF-8 | C.UTF-8 | =c/yby           +
           |       |          |         |         | yby=CTc/yby
 yby       | yby   | UTF8     | C.UTF-8 | C.UTF-8 |
```

连接数据库

```
\c yby 
```

查看帮助

```
\h create user
```

显示匹配的"pattern"信息

```
yby-# \d #列出当前数据库的所有表
       List of relations
 Schema | Name | Type  | Owner
--------+------+-------+-------
 public | t    | table | yby
(1 row)
```

```
yby-# \d t #+表名 查看结构定义 可加上通配符如 "*" "?" 等
                        Table "public.t"
 Column |         Type          | Collation | Nullable | Default
--------+-----------------------+-----------+----------+---------
 id     | integer               |           | not null |
 name   | character varying(40) |           |          |
Indexes:
    "t_pkey" PRIMARY KEY, btree (id)
```

```
yby-# \d t_pkey # 查看索引信息
        Index "public.t_pkey"
 Column |  Type   | Key? | Definition
--------+---------+------+------------
 id     | integer | yes  | id
primary key, btree, for table "public.t"
```

```
yby-# \d+ t #d+ 用于显示更详细的信息
								Table "public.t"
 Column |         Type          | Collation | Nullable | Default | Storage  | Stats target | Description
--------+-----------------------+-----------+----------+---------+----------+--------------+-------------
 id     | integer               |           | not null |         | plain    |              |
 name   | character varying(40) |           |          |         | extended |              |
Indexes:
    "t_pkey" PRIMARY KEY, btree (id)
Access method: heap
```

匹配不同的对象信息

| \dt  | 只显示表的信息   |
| ---- | ---------------- |
| \di  | 只显示索引的信息 |
| \ds  | 只显示序列       |
| \dv  | 只显示试图       |
| \df  | 只显示函数       |

显示sql的执行时间

```
yby=# \timing on # \timing off
Timing is on.
yby=#  select count(*) from testtable;
 count
-------
     0
(1 row)

Time: 0.724 ms
```

列出所有的模式

```
yby=# \dn
List of schemas
  Name  | Owner
--------+-------
 public | yby
(1 row)
```

显示所有的表空间

```
yby=# \db
      List of tablespaces
    Name    | Owner | Location
------------+-------+----------
 pg_default | yby   |
 pg_global  | yby   |
(2 rows)
```

列出所有角色或用户

```
yby=# \du # \dg
                                   List of roles
 Role name |                         Attributes                         | Member of
-----------+------------------------------------------------------------+-----------
 yby       | Superuser, Create role, Create DB, Replication, Bypass RLS | {}
```

显示表的权限分配

```
yby=# \dp testtable #\z
                               Access privileges
 Schema |   Name    | Type  | Access privileges | Column privileges | Policies
--------+-----------+-------+-------------------+-------------------+----------
 public | testtable | table |                   |                   |
(1 row)
```

指定客户端的字符集

```
\encoding UTF8 #指定为UTF8
```

格式化输出

```
yby=# \pset border 2  #像msql一样带有内外边框 0为无边框
Border style is 2.
yby=# select * from testtable ; 
+----+------+
| id | name |
+----+------+
|  2 | 钱   |
|  3 | 孙   |
|  4 | 李   |
|  5 | 周   |
|  6 | 武   |
|  7 | 郑   |
+----+------+
(6 rows)

Time: 0.588 ms
```

```
yby=# \pset format unaligned #不对齐
Output format is unaligned.
yby=# select * from testtable ;
id|name
2|钱
3|孙
4|李
5|周
6|武
7|郑
(6 rows)
```

```
yby=# \pset fieldsep '\t' #设置分隔符
Field separator is "    ".
yby=# select * from testtable ;
id      name
2       钱
3       孙
4       李
5       周
6       武
7       郑
(6 rows)
```

```
yby=# \o output.txt #输出到文件 \o 则恢复
yby=# select * from testtable;
```

```
yby=# \t #去掉列头信息 同时应该去掉边框 再次执行则恢复
Tuples only is on.
yby=# \o output.txt
yby=# select * from testtable;
```



按行展示变为按列展示

```
yby=# \x #再次执行恢复
Expanded display is on.
yby=# select * from testtable;
+-[ RECORD 1 ]-+
| id   | 2  |
| name | 钱 |
+-[ RECORD 2 ]-+
| id   | 3  |
| name | 孙 |
+-[ RECORD 3 ]-+
| id   | 4  |
| name | 李 |
+-[ RECORD 4 ]-+
| id   | 5  |
| name | 周 |
+-[ RECORD 5 ]-+
| id   | 6  |
| name | 武 |
+-[ RECORD 6 ]-+
| id   | 7  |
| name | 郑 |
+------+----+
```

执行外部文件中的sql命令

```
yby=# \i input.txt
 id | name
----+------
  2 | 钱
  3 | 孙
  4 | 李
  5 | 周
  6 | 武
  7 | 郑
(6 rows)
```

```
~ ➤ psql -x -f input.txt #-x 表示使用 \x
-[ RECORD 1 ]
id   | 2
name | 钱
-[ RECORD 2 ]
id   | 3
name | 孙
-[ RECORD 3 ]
id   | 4
name | 李
-[ RECORD 4 ]
id   | 5
name | 周
-[ RECORD 5 ]
id   | 6
name | 武
-[ RECORD 6 ]
id   | 7
name | 郑
```

编辑命令

```
\e #进入编辑器，退出后执行其内的任务
```

```
\e input.txt #指定文件名，文件名必须存在
```

```
\ef #编辑函数 	\ev #编辑视图 结束后输入 \reset 来刷新缓冲区
```

输出信息

```
\echo  在input文件中使用可输出该行后面的信息
```

其他命令

```
\?
```

自当提交

```
#为了防止自动提交
begin;
......
commit (rollback)

#或者
set AUTOMMIT off
```

得到psql中快捷命令实际执行的sql

```
psql -E

yby=# \d
********* QUERY **********
SELECT n.nspname as "Schema",
  c.relname as "Name",
  CASE c.relkind WHEN 'r' THEN 'table' WHEN 'v' THEN 'view' WHEN 'm' THEN 'materialized view' WHEN 'i' THEN 'index' WHEN 'S' THEN 'sequence' WHEN 's' THEN 'special' WHEN 'f' THEN 'foreign table' WHEN 'p' THEN 'partitioned table' WHEN 'I' THEN 'partitioned index' END as "Type",
  pg_catalog.pg_get_userbyid(c.relowner) as "Owner"
FROM pg_catalog.pg_class c
     LEFT JOIN pg_catalog.pg_namespace n ON n.oid = c.relnamespace
WHERE c.relkind IN ('r','p','v','m','S','f','')
      AND n.nspname <> 'pg_catalog'
      AND n.nspname <> 'information_schema'
      AND n.nspname !~ '^pg_toast'
  AND pg_catalog.pg_table_is_visible(c.oid)
ORDER BY 1,2;
**************************

         List of relations
 Schema |   Name    | Type  | Owner
--------+-----------+-------+-------
 public | testtable | table | yby
(1 row)
```

```
\set
```

tips:

字符串用单引号而非双引号括起来

