## 6.5 触发器

### 6.5.1 创建触发器

先创建触发器函数

```sql
create or replace function func_Nm()
returns trigger as $$
begin
	...
	return old
end;
$$
language plpgsql;
```

在创建触发器

```sql
create trigger trggr_Nm
after action on tb_Nm
for each row execute procedure funcNm();
```

### 6.5.2 语句级触发器和行级触发器



语句级触发器对每个sql语句执行一次，无论更新多少数据，是否有更新，而行级触发器对每行要执行的sql语句都会执行一次。

语句级触发器，语句级触发器函数的返回值必须是NULL

```sql
create trigger trggr_Nm
after action on tb_Nm
for statement execute procedure funcNm();
```

### 6.5.3 before after触发器

before在执行之前触发（语句级在语句执行前，行级在特定操作执行前），after在执行后触发，语句级和行级同理。

### 6.5.4 删除触发器

```sql
drop trigger [if exists] name on table [cascade | restrict];
```

cascade:删除初以来该触发器的对象

restrict : 有依赖对象就拒绝删除

删除触发器时，触发器函数不会被删除，而删除表时，触发器会被删除

### 6.5.5触发器的行为

语句级触发器函数必须显示的返回NULL，否则报错

对于before和instead of 这类行级触发器来说，如果返回的是NULL，则表示忽略当前操作，如果是非NULL，对于insert和update来说，返回的行将成为要被插入的行或是要更新的行

同一个事务上如果有多个触发器，则按触发器名称的顺序来触发。

### 6.5.6触发器中的特殊变量

触发器函数中特殊的变量

- new 新加入的行
- old，原有的行
- tg_name 实际触发的触发器名
- tg_when: before/after
- tg_level: row/statement
- tg_op: insert ,update delete truncate
- ......

## 6.6 事件触发器