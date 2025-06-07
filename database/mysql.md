# MySQL

## 慢SQL如何查询, 有哪些SQL调优手段?

- 开启慢查询日志, 设置 slow_query_log和long_query_time记录超过指定时间的查询

```sql
SET GLOBAL slow_query_log = 'ON'; -- 启用慢查询日志
SET GLOBAL long_query_time = 1; -- 设置慢查询的阈值，单位为秒（例如 1 秒）
SET GlOBAL log_queries_not_using_indexes = 'ON';

-- 查看慢日志默认位置
SHOW VARIABLES LIKE '%slow_query_log_file%';
```

- 监控工具 接入SkyWalking

```
GET /api/order/list
|
|-- Mapper: OrderMapper.selectOrderByUserId()
     SQL: SELECT * FROM t_order WHERE user_id = ?
     耗时：2.3s
```

- 使用Explain分析查询

```sql
EXPLAIN SELECT * FROM your_table WHERE your_column = 'value';
```

### 实战 百万条数据

```sql
CREATE TABLE t_order (
    id BIGINT PRIMARY KEY AUTO_INCREMENT COMMENT '主键ID',
    order_no VARCHAR(64) NOT NULL COMMENT '订单编号',
    user_id BIGINT NOT NULL COMMENT '用户ID',
    product_id BIGINT NOT NULL COMMENT '商品ID',
    amount DECIMAL(10,2) NOT NULL COMMENT '订单金额',
    status TINYINT NOT NULL DEFAULT 0 COMMENT '订单状态 0-待支付 1-已支付 2-已发货 3-已完成 4-已取消',
    create_time DATETIME NOT NULL DEFAULT CURRENT_TIMESTAMP COMMENT '创建时间',
    update_time DATETIME NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP COMMENT '更新时间',
    INDEX idx_user_id (user_id),
    INDEX idx_order_no (order_no),
    INDEX idx_create_time (create_time)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COMMENT='订单表';
```

使用python脚本插入百万条数据

```python
import pymysql
import random
from datetime import datetime, timedelta

conn = pymysql.connect(host='localhost', user='root', password='123456', db='test')
cursor = conn.cursor()

for i in range(1, 1000001):
    order_no = f"ORD{str(i).zfill(10)}"
    user_id = random.randint(1, 10000)
    product_id = random.randint(1, 1000)
    amount = round(random.uniform(0, 1000), 2)
    status = random.randint(0, 4)
    days_ago = random.randint(0, 365)
    create_time = (datetime.now() - timedelta(days=days_ago)).strftime('%Y-%m-%d %H:%M:%S')

    sql = f"""INSERT INTO t_order(order_no, user_id, product_id, amount, status, create_time)
              VALUES('{order_no}', {user_id}, {product_id}, {amount}, {status}, '{create_time}')"""
    cursor.execute(sql)

    if i % 1000 == 0:
        conn.commit()
        print(f"Inserted: {i}")

cursor.close()
conn.close()
```

explain详解，比如下面sql

```sql
EXPLAIN SELECT * FROM t_order  ORDER BY create_time DESC LIMIT 50 OFFSET 500000;
```

输出结果为：
| id | select\_type | table | partitions | type | possible\_keys | key | key\_len | ref | rows | filtered | Extra |
| :--: | :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- |
| 1 | SIMPLE | t\_order | null | ALL | null | null | null | null | 995600 | 100 | Using filesort |

-  select_type: 常见的有simple(单表查询）, primary（主查询）, subquery（子查询）
- type：
  - ALL（全表扫描）
  - index （全索引扫描）不回表
  - range 范围查询
  - ref 非唯一索引等值匹配
  - eq_ref 多表连接中使用主键连接
  - const 主键/唯一索引 = 常量
  - system 表只有一行
- Possible_keys: 可能使用的索引  key：实际使用的索引  key_len: 使用索引长度
- ref：用于索引查找的比较值   rows：预估扫描的行数 
- extra：
  - using index 覆盖索引（不回表）
  - using where (where过滤)
  - using filesort (额外排序)
  - Using temporary (使用临时表)
  - impossible where （条件不成立，不会返回数据）
  - using join buffer (不使用索引，需创建连接缓存)

**深分页问题**
比如分页查询订单列表

```sql
-- 查询第 10000 页，每页 20 条
SELECT * FROM t_order ORDER BY create_time DESC LIMIT 199980, 20;
```

MySQL仍然会扫描前N+M条，只取最后M条，即会便利前200000万条

执行时间对比：

```sql
-- 查询第 1 页，每页 20 条
SELECT * FROM t_order ORDER BY create_time DESC LIMIT 0, 20;

[2025-06-07 15:40:40] 20 rows retrieved starting from 1 in 345 ms (execution: 4 ms, fetching: 341 ms)


-- 查询第100页
SELECT * FROM t_order ORDER BY create_time DESC LIMIT 1980, 20;
[2025-06-07 15:42:07] 20 rows retrieved starting from 1 in 354 ms (execution: 26 ms, fetching: 328 ms)

-- 查询第1000页
SELECT * FROM t_order ORDER BY create_time DESC LIMIT 19800, 20;
[2025-06-07 15:44:03] 20 rows retrieved starting from 1 in 751 ms (execution: 564 ms, fetching: 187 ms)

-- 查询第10000页



```

