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