* 解锁账号

```sql
alter user gumdbln account unlock;
```

* 查看链接数

```sql
--数据库允许的最大连接数
select value from v$parameter where name = 'processes';
--当前的连接数
select count(*) from v$process;
```

