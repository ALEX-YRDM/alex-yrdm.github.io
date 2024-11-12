# MySQL

## 慢SQL如何查询, 有哪些SQL调优手段?

- 开启慢查询日志, 设置 slow_query_log和long_query_time记录超过指定时间的查询

```sql
SET GLOBAL slow_query_log = 'ON'; -- 启用慢查询日志
SET GLOBAL long_query_time = 1; -- 设置慢查询的阈值，单位为秒（例如 1 秒）
```

- 使用Explain分析查询

```sql
EXPLAIN SELECT * FROM your_table WHERE your_column = 'value';
```

SQL调优:

- 添加索引:  
  - 为查询中常用的列 Where  join  group by  order by的列添加索引
  - 使用合适的索引类型   B-Tree   哈希索引
- 重写
  - 避免 select *
  - 使用where子句过滤不必要的行
  - 避免使用子查询: 使用join代替子查询
- 优化查询逻辑
  - 避免长时间运行的事务
  - 限制结果集 使用Limit限制返回行数

