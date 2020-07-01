* 检查Oracle运行中的SQL

```sql
SELECT b.sid oracleID,
       b.username 登录Oracle用户名,
       b.serial#,
       spid 操作系统ID,
       paddr,
       sql_text 正在执行的SQL,
       b.machine 计算机名
FROM v$process a, v$session b, v$sqlarea c
WHERE a.addr = b.paddr
   AND b.sql_hash_value = c.hash_value
   AND UPPER(sql_text) LIKE '%DELETE FROM RM_TRANS_PORT_TEMP%'
```

