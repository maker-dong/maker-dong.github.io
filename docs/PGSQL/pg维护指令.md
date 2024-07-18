# 查看活跃进程

```sql
SELECT pid, datname, usename, query FROM pg_stat_activity WHERE state = 'active';
```

# 结束进程

```sql
SELECT pg_terminate_backend(1234);
```

# 查看表大小

```sql
SELECT pg_size_pretty(pg_total_relation_size('表名'));
```

