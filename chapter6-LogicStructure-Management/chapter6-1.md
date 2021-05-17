# 第6章-逻辑结构管理





## 6.1-逻辑结构介绍

1. 数据库：一个PGSQL数据库服务可以管理多个数据库，应用链接到一个数据库的时候，一般只能方位这一个数据库
2. 表,索引：表的术语为 `Relation`
3. 数据行：术语为tuple

## 6.2-数据库的基本操作

### 6.2.1-创建数据库

指定字符编码

```sql
 create database testdb01 encoding 'UTF-8' template template0;
```

必须将模板指定为template0（默认为template1），因为模板数据可能包含于字符编码不匹配的数据，而template0不包含受字符集编码影响的数据（索引）

### 6.2.2-修改数据库

将数据库的最大连接数改为10

```sql
alter database testdb01 connection limit 10;
```

修改数据库的名称

```sql
 alter database testdb01 rename to mydb01;
```

修改配置参数，关闭默认索引扫描

```sql
 alter database mydb01 set enable_indexscan to off;
```

### 6.2.3  删除数据库

如果数据库存在就将其删除

```sql
 drop database if exists mydb01;
```

### 6.2.4

不能在事务块中删除数据库，但是可以修改

## 6.3-模式Schema

### 6.3.1-概念

模式相当于一个命名空间，不同的模式下可以有相同名称的表，函数对象而不冲突

一个数据库包含一个或多个模式，模式中包含表，函数以及操作符等数据对象。当要访问另一个数据库中的对象时，需要重新连接，而模式则无该限制，也就是说当连接到一个数据库后,可以访问这个数据库中的多个模式对象

原因：

1. 允许多个用户使用同一个数据库而不相互干扰
2. 组成逻辑组，便于管理
3. 第三方应用可以放在不同的模式中，这样就不会冲突

### 6.3.2-使用

创建一个模式

```sql
create schema ybydba;
```

查看模式

```
\dn
```

删除模式

```sql
 drop schema ybydba ;
```

为某个用户创建模式

```sql
create schema authorization yby;
```

修改名称

```sql
alter schema yby rename to ybyold
```

修改属主

```sql
alter schema ybyold owner to yby;
```

### 6.3.3-公共模式

当不指定模式的时候，一般为public模式

### 6.3.4-模式的搜索路径

类似`shell`的`$PATH`变量，在`PATH`变量下的就不用指定路径（模式名），否则就要全路径名

显示搜索路径

```sql
yby=# show search_path ;
   search_path
-----------------
 "$user", public
(1 row)
```

### 6.3.5模式的权限

撤销在public中的权限，其他用户就不能在public 中创建对象了

```sql
revoke create on schema public from public
```

### 6.3.6 可以执行

有些数据库没有模式，所以对名字加以“修饰”，使他们真正由`username.tablename`组成，建议不要使用（甚至是删除）public模式

## 6.4 表

### 6.4.1创建表

复合主键

```sql
 create table test01(id1 int,id2 int,note varchar(20),constraint pk_test01 primary key(id1,id2));
```

唯一键

```sql
 create table test02(id1 int,id2 int,id3 int,note varchar(20),constraint pk_test02 primary key(id1,id2),constraint uk_test02 Unique(id3));
```

check

```sql
constraint ck_child_age check(age<18)
```

用模板创建表

```sql
create table baby (like child)
```

但是这样不复制约束，使用including 复制约束

```sql
yby=# create table test03 (like test02 including all);
CREATE TABLE
yby=# \d test03
                      Table "public.test03"
 Column |         Type          | Collation | Nullable | Default
--------+-----------------------+-----------+----------+---------
 id1    | integer               |           | not null |
 id2    | integer               |           | not null |
 id3    | integer               |           |          |
 note   | character varying(20) |           |          |
Indexes:
    "test03_pkey" PRIMARY KEY, btree (id1, id2)
    "test03_id3_key" UNIQUE CONSTRAINT, btree (id3)
```

### 6.4.2 表的存储属性

TOAST(The Oversized-Attribute Storage Technique):pgsql的**页面大小是固定的**（通常为8kb)，且**不允许跨越多个页面**，为了存放更大的字段，大字段的值被压缩或切片成多个物理行存放在另一张系统表中。

在边长类型中前4个字节（32bit）表示长度字，第一个bit表示是否压缩，第二个表示是否为**行外存储**（即当前存储的内容只是一个指针），后30位表示长度。

如果表中有任意一个字段是toast，pgsql 自动创造一个相关联的toast表

字段的toast策略

1. plain
2. extended
3. extern
4. main

触发toast的长度

### 6.4.3临时表

1. 会话级临时表（默认）
```sql
create table temporary table tableName(...);
```
2. 事物级临时表
```sql
create table temporary table tableName(...) on commit delete rows;
```

### 6.4.4 unlogged 表

通过禁止产生wal日志的方式来提高性能，无wal日志，则主备库无法同步，此时数据库宕机，则表的内容丢失

### 6.4.5 默认值

修改、插入时若不指明，则为默认值

```sql
 create table test03(id1 int,id2 int default 15);
```

```sql
update test03 set id2=default
```

如果没有明确的默认值，则默认值为null

### 6.4.6 约束

1. 检查约束

```sql
age int check (age>10),

age int constraint check(age>10),

age int,
check(age>10)

age int,
constraint ageCheck check(age>10),
```

2. 非空约束

```sql
age int not null,

check (age is not null)
```

3. 唯一约束

```sql
age int unique,

unique age
```

4. 主键，在后期给表加上主键

```sql
alter table tableName add constraint cnstrntName primary key (...)
```

5. 外键

```sql
class_no int references class(class_no)
```

### 6.4.7修改表

1. 增加字段,填充默认值

```sql
alter table tb_Nm add column clm_Nm tp check (...);
```

2. 删除字段，也删除与之相关的约束，如果该字段与另一个表的外键相关，则报错

```sql
alter table tb_Nm drop column clm_Nm;
```

删除其他表的外键依赖,

```sql
alter table tb_Nm drop column clm_Nm CASCADE;
```

3. 增加约束

```sql
alter table student add check (...);
alter table student add constrain cnstrnt_Nm check (...);
alter table student alter column clm_Nm set not NULL;
```

4. 删除约束

```sql
alter table tb_Nm drop constraint cnstrnt_Nm;
```

5. 修改默认值

```sql
alter table student alter column clm_Nm set default dflt_vl
```

6. 删除默认值

```sql
alter table student alter column clm_Nm drop default
```

7. 修改字段类型，只有可以隐式转化时才能成功，最好在修改前删除约束，修改后在添加回去

```sql
alter table student alter column clm_Nm type yp
```

### 6.4.8 表继承

```sql
create table tb_Nm(...)inherits (parent_tb_Nm);
```

查询父表时会把父表中子表的数据也查询出来，繁殖则不行，如果只想查询父表，则

```sql
select * from only parent_tb_Nm;
```

父表的所有检查约束和非空约束都会自动被子表继承，其他类型的约束（唯一，主键，外键）则不会

一个子表可以继承多个父表，如果同一个名字出现在多个父表或者父表和子表中，则会`融合`，要求数据类型相同，将拥有所有父表的检查约束

### 6.4.9 通过表继承来实现分区表

分区表的好处

1. 删除历史数据更快
2. 某些查询的性能可以提升，使用率较高的分区表的索引可能完全缓冲在内存中
3. 当查询、更新一个分区的大部分数据时，连续扫描该分区而不是使用索引离散的访问，可以获得巨大的性能提升
4. 将少用到的存储数据放在便宜慢额存储设备上

创建分区表的步骤

1. 创建父表，所有分区都从他继承
2. 创建子表，每个表都是从主表上继承的
3. 给分区增加约束，定义允许的键值
4. 对每个分区，在关键字段创建索引
5. 定义`规则`或者`触发器`，将主表的数据插入重定向到每个子表中
6. 确保constraint_exclusion中配置的参数`postgresql.conf`是打开的，打开后，如果查询中where子句的过滤条件宇分区的约束条件匹配，那么该查询会智能地只查询此分区，而不会查询其他分区

触发器如果查不到子表，只会在运行阶段报错，

使用`规则`,

规则的缺点：

1. 有明显较大的开销，且每次检查都会禅城此开销
2. 使用copy插入数据时，由于copy不会触发规则,因此要把复制的数据直接copy到分区表
3. 如果插入的数据时规则范围之外的，就会插入到主表中，而不是直接报错

### 6.4.10 声明式分区

```sql
create table tb_Nm(...) partition by range(col_Nm);
create table parti_tb_Nm partition of tb_Nm for values from (...) to (...)
```

