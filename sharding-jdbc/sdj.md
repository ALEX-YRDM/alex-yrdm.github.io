## 数据库架构

### 读写分离

主节点处理写请求， 从节点处理读请求，从节点同步主节点数据

一主多从  多主多从

缓解频繁数据更新导致的行锁，使得系统查询性能得到极大改善，并且增强了可用性

但是没有分散存储压力

### 数据库分片

- 垂直分库

将用户表和订单表分散到2个数据库中

- 垂直分表

将用户表中的一些不重要的字段，拆出到另外一张表中，两张表id保持一致

- 水平分库

可以按照id取模。模为0保存在0号库 为1保存在1号库

- 水平分表

同样按照id取模，模为0保存在table0中，模为1保存在table1中

## MySQL主从节点配置

### 主从同步原理

slave从master读取binlog来进行数据同步

- 1.master将数据修改记录保存到binlog中
- 2.slave使用start slave后，开启IO线程连接master，读取binlog
- 3.slave连接上slave后，开启log dump线程，发送binlog内容
- 4.slave接受主节点bingo dump发送来的更新后，保存在中继日志 relay log中
- 5.slave的sql线程，读取relay log日志，解析具体操作，实现主从操作一直，保持数据一致性

### 一主2从搭建

主节点：3307 

```shell
# 主节点
docker run -d \
-p 3307:3306 \
-v /Users/zbq/data/mysql/master/conf:/etc/mysql/conf.d \
-v /Users/zbq/data/mysql/master/data:/var/lib/mysql \
-e MYSQL_ROOT_PASSWORD=123456 \
--name mysql-master \
mysql:8.0
```

```shell
# 创建配置文件
vim /Users/zbq/data/mysql/master/conf/my.cnf

[mysqld]
# 服务器唯一id，默认值1
server-id=1
# 设置日志格式，默认值ROW
binlog_format=STATEMENT

docker restart mysql-master
```

binlog格式：

- STATEMENT: 日志记录主机数据库的写指令，性能高，但是now()之类的函数以及获取系统参数的操作会出现主从数据不同步的问题
- ROW：记录主机数据库写后的数据，批量操作时性能较差，可以解决数据不一致问题
- MIXED：上面两种的混合使用。有函数用row，没函数用STATEMENT

主节点上配置

```shell
CREATE USER 'slave'@'%';
-- 设置密码
ALTER USER 'slave'@'%' IDENTIFIED WITH mysql_native_password BY '123456';
-- 授予复制权限
GRANT REPLICATION SLAVE ON *.* TO 'slave'@'%';
-- 刷新权限
FLUSH PRIVILEGES;


SHOW MASTER STATUS;

| File | Position |
| :--- | :--- |
| binlog.000003 | 1083 |

```



从节点： 3308 3309

```shell
docker run -d \
-p 3308:3306 \
-v /Users/zbq/data/mysql/slave1/conf:/etc/mysql/conf.d \
-v /Users/zbq/data/mysql/slave1/data:/var/lib/mysql \
-e MYSQL_ROOT_PASSWORD=123456 \
--name mysql-slave1 \
mysql:8.0


docker run -d \
-p 3309:3306 \
-v /Users/zbq/data/mysql/slave2/conf:/etc/mysql/conf.d \
-v /Users/zbq/data/mysql/slave2/data:/var/lib/mysql \
-e MYSQL_ROOT_PASSWORD=123456 \
--name mysql-slave2 \
mysql:8.0
```

```shell
vim /Users/zbq/data/mysql/slave1/conf/my.cnf

[mysqld]
# 服务器唯一id，每台服务器的id必须不同，如果配置其他从机，注意修改id
server-id=2

docker restart mysql-slave1


vim /Users/zbq/data/mysql/slave2/conf/my.cnf

[mysqld]
# 服务器唯一id，每台服务器的id必须不同，如果配置其他从机，注意修改id
server-id=3

docker restart mysql-slave2
```

从机器上执行以下SQL绑定master：

```sql
CHANGE MASTER TO MASTER_HOST='192.168.3.45', 
MASTER_USER='slave',MASTER_PASSWORD='123456', MASTER_PORT=3307,
MASTER_LOG_FILE='binlog.000003',MASTER_LOG_POS=1083; 
```

启动主从同步功能：

```sql
START SLAVE;
-- 查看状态（不需要分号）
SHOW SLAVE STATUS

| Slave\_IO\_Running | Slave\_SQL\_Running |
| :--- | :--- |
| Yes | Yes |
```

测试主从同步是否生效：

```sql
CREATE DATABASE db_user;
USE db_user;
CREATE TABLE t_user(
	id BIGINT AUTO_INCREMENT,
  uname VARCHAR(30),
  PRIMARY KEY (id)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4

INSERT INTO t_user(uname) VALUES('zbq');
INSERT INTO t_user(uname) VALUES(@@hostname);
```

停止与重置

```sql
-- 在从机上执行。功能说明：停止I/O 线程和SQL线程的操作。
stop slave; 

-- 在从机上执行。功能说明：用于删除SLAVE数据库的relaylog日志文件，并重新启用新的relaylog文件。
reset slave;

-- 在主机上执行。功能说明：删除所有的binglog日志文件，并将日志索引文件清空，重新开始所有新的日志文件。
-- 用于第一次进行搭建主从库时，进行主库binlog初始化工作；
reset master;
```

## Sharding-Jdbc 读写分离

### pom依赖

```xml
# SpringBoot版本：2.7.6
<dependency>
    <groupId>com.baomidou</groupId>
    <artifactId>mybatis-plus-boot-starter</artifactId>
    <version>3.5.15</version>
</dependency>

<dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
    </dependency>

    <dependency>
        <groupId>org.apache.shardingsphere</groupId>
        <artifactId>shardingsphere-jdbc-core-spring-boot-starter</artifactId>
        <version>5.5.2</version>
    </dependency>

    <dependency>
        <groupId>mysql</groupId>
        <artifactId>mysql-connector-java</artifactId>
        <scope>runtime</scope>
    </dependency>

    <dependency>
        <groupId>org.projectlombok</groupId>
        <artifactId>lombok</artifactId>
        <optional>true</optional>
    </dependency>
    
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-test</artifactId>
        <scope>test</scope>
        <exclusions>
            <exclusion>
                <groupId>org.junit.vintage</groupId>
                <artifactId>junit-vintage-engine</artifactId>
            </exclusion>
        </exclusions>
    </dependency>
```

### 实体类，mapper和service

```java
@TableName("t_user")
@Data
@AllArgsConstructor
@NoArgsConstructor
public class User {
    @TableId(type = IdType.AUTO)
    private Long id;
    private String uname;
}


@Mapper
public interface UserMapper extends BaseMapper<User> {
}


public interface UserService extends IService<User> {
}

@Service
public class UserServiceImpl extends ServiceImpl<UserMapper, User> implements UserService {
}
```

### 配置文件，多个数据源以及读写分离配置，负载均衡配置

```properties
# 应用名称
spring.application.name=sharging-jdbc
# 开发环境设置
spring.profiles.active=dev
# 内存模式
spring.shardingsphere.mode.type=Memory

# 配置真实数据源
spring.shardingsphere.datasource.names=master,slave1,slave2

# 配置第 1 个数据源
spring.shardingsphere.datasource.master.type=com.zaxxer.hikari.HikariDataSource
spring.shardingsphere.datasource.master.driver-class-name=com.mysql.jdbc.Driver
spring.shardingsphere.datasource.master.jdbc-url=jdbc:mysql://127.0.0.1:3307/db_user
spring.shardingsphere.datasource.master.username=root
spring.shardingsphere.datasource.master.password=123456

# 配置第 2 个数据源
spring.shardingsphere.datasource.slave1.type=com.zaxxer.hikari.HikariDataSource
spring.shardingsphere.datasource.slave1.driver-class-name=com.mysql.jdbc.Driver
spring.shardingsphere.datasource.slave1.jdbc-url=jdbc:mysql://127.0.0.1:3308/db_user
spring.shardingsphere.datasource.slave1.username=root
spring.shardingsphere.datasource.slave1.password=123456

# 配置第 3 个数据源
spring.shardingsphere.datasource.slave2.type=com.zaxxer.hikari.HikariDataSource
spring.shardingsphere.datasource.slave2.driver-class-name=com.mysql.jdbc.Driver
spring.shardingsphere.datasource.slave2.jdbc-url=jdbc:mysql://127.0.0.1:3309/db_user
spring.shardingsphere.datasource.slave2.username=root
spring.shardingsphere.datasource.slave2.password=123456

# 读写分离类型，如: Static，Dynamic
spring.shardingsphere.rules.readwrite-splitting.data-sources.myds.type=Static
# 写数据源名称
spring.shardingsphere.rules.readwrite-splitting.data-sources.myds.props.write-data-source-name=master
# 读数据源名称，多个从数据源用逗号分隔
spring.shardingsphere.rules.readwrite-splitting.data-sources.myds.props.read-data-source-names=slave1,slave2

# 负载均衡算法名称
spring.shardingsphere.rules.readwrite-splitting.data-sources.myds.load-balancer-name=alg_round

# 负载均衡算法配置
# 负载均衡算法类型
spring.shardingsphere.rules.readwrite-splitting.load-balancers.alg_round.type=ROUND_ROBIN
spring.shardingsphere.rules.readwrite-splitting.load-balancers.alg_random.type=RANDOM
spring.shardingsphere.rules.readwrite-splitting.load-balancers.alg_weight.type=WEIGHT
spring.shardingsphere.rules.readwrite-splitting.load-balancers.alg_weight.props.slave1=1
spring.shardingsphere.rules.readwrite-splitting.load-balancers.alg_weight.props.slave2=2

# 打印SQl
spring.shardingsphere.props.sql-show=true
```

### 测试

```java
@SpringBootTest
public class ReadWriteTest {

    @Autowired
    private UserService userService;
		
  	//写入主库
    @Test
    public void testInsert(){
        for (int i = 0; i < 10; i++) {
            userService.save(new User(null, "test" + i));
        }
    }
		//不带事务，主库插入，从库来读
  	//是事务，主库插入主库来读 保证一致性
    @Transactional
    @Test
    public void testTrans(){
        userService.save(new User(null, "zbq"));
        userService.list().forEach(System.out::println);
    }
		//读请求，按照负载均衡策略落到读库中
    @Test
    public void testLoadBalance(){
        for(int i=0;i<5;i++){
            userService.list().forEach(System.out::println);
        }


    }
}
```

## 垂直分片

垂直分表正常操作即可，这块只实现垂直分库  不同的表放在不同的库中

```shell
docker run -d \
-p 3301:3306 \
-v /Users/zbq/data/mysql/user/conf:/etc/mysql/conf.d \
-v /Users/zbq/data/mysql/user/data:/var/lib/mysql \
-e MYSQL_ROOT_PASSWORD=123456 \
--name mysql-user \
mysql:8.0

CREATE DATABASE db_user;
USE db_user;
CREATE TABLE t_user (
 id BIGINT AUTO_INCREMENT,
 uname VARCHAR(30),
 PRIMARY KEY (id)
);


docker run -d \
-p 3302:3306 \
-v /Users/zbq/data/mysql/order/conf:/etc/mysql/conf.d \
-v /Users/zbq/data/mysql/order/data:/var/lib/mysql \
-e MYSQL_ROOT_PASSWORD=123456 \
--name mysql-order \
mysql:8.0


CREATE DATABASE db_order;
USE db_order;
CREATE TABLE t_order (
  id BIGINT AUTO_INCREMENT,
  order_no VARCHAR(30),
  user_id BIGINT,
  amount DECIMAL(10,2),
  PRIMARY KEY(id) 
);
```

### 实体类，mapper和service

```java
@TableName("t_order")
@Data
@AllArgsConstructor
@NoArgsConstructor
public class Order {
    @TableId(type = IdType.AUTO)
    private Long id;
    private String orderNo;
    private Long userId;
    private BigDecimal amount;
}

@Mapper
public interface OrderMapper extends BaseMapper<Order> {
}

public interface OrderService extends IService<Order> {
}

@Service
public class OrderServiceImpl extends ServiceImpl<OrderMapper, Order> implements OrderService {
}
```

### 配置

```properties
# 应用名称
spring.application.name=sharging-jdbc-demo
# 开发环境设置
spring.profiles.active=dev
# 内存模式
spring.shardingsphere.mode.type=Memory

# 配置真实数据源
spring.shardingsphere.datasource.names=server-user,server-order

# 配置第 1 个数据源
spring.shardingsphere.datasource.server-user.type=com.zaxxer.hikari.HikariDataSource
spring.shardingsphere.datasource.server-user.driver-class-name=com.mysql.jdbc.Driver
spring.shardingsphere.datasource.server-user.jdbc-url=jdbc:mysql://127.0.0.1:3301/db_user
spring.shardingsphere.datasource.server-user.username=root
spring.shardingsphere.datasource.server-user.password=123456

# 配置第 2 个数据源
spring.shardingsphere.datasource.server-order.type=com.zaxxer.hikari.HikariDataSource
spring.shardingsphere.datasource.server-order.driver-class-name=com.mysql.jdbc.Driver
spring.shardingsphere.datasource.server-order.jdbc-url=jdbc:mysql://127.0.0.1:3302/db_order
spring.shardingsphere.datasource.server-order.username=root
spring.shardingsphere.datasource.server-order.password=123456


# 标准分片表配置（数据节点）
# spring.shardingsphere.rules.sharding.tables.<table-name>.actual-data-nodes=值
# 值由数据源名 + 表名组成，以小数点分隔。
# <table-name>：逻辑表名
spring.shardingsphere.rules.sharding.tables.t_user.actual-data-nodes=server-user.t_user
spring.shardingsphere.rules.sharding.tables.t_order.actual-data-nodes=server-order.t_order
# 打印SQl
spring.shardingsphere.props.sql-show=true
```

### 测试

```java
@SpringBootTest
public class VerticalTest {

    @Autowired
    private UserService userService;

    @Autowired
    private OrderService orderService;

    @Test
    public void testInsert(){
        userService.save(new User(null, "user1"));

        orderService.save(new Order(null, "order1", 1L, new BigDecimal("100.00")));
    }

    @Test
    public void testQuery(){
        userService.getById(1L);

        orderService.getById(1L);
    }
}
```

## 水平分片

对于order，分2个库，server-order0和server-order1

每个库分两张表 t_order0 t_order1

```shell
docker run -d \
-p 3303:3306 \
-v /Users/zbq/data/mysql/order0/conf:/etc/mysql/conf.d \
-v /Users/zbq/data/mysql/order0/data:/var/lib/mysql \
-e MYSQL_ROOT_PASSWORD=123456 \
--name mysql-server-order0 \
mysql:8.0

docker run -d \
-p 3304:3306 \
-v /Users/zbq/data/mysql/order1/conf:/etc/mysql/conf.d \
-v /Users/zbq/data/mysql/order1/data:/var/lib/mysql \
-e MYSQL_ROOT_PASSWORD=123456 \
--name mysql-server-order1 \
mysql:8.0
```

```sql
CREATE DATABASE db_order;
USE db_order;
CREATE TABLE t_order0 (
  id BIGINT,
  order_no VARCHAR(30),
  user_id BIGINT,
  amount DECIMAL(10,2),
  PRIMARY KEY(id) 
);
CREATE TABLE t_order1 (
  id BIGINT,
  order_no VARCHAR(30),
  user_id BIGINT,
  amount DECIMAL(10,2),
  PRIMARY KEY(id) 
);
```

### 基本水平分片

基本配置

```properties
#========================基本配置
# 应用名称
spring.application.name=sharging-jdbc-demo
# 开发环境设置
spring.profiles.active=dev
# 内存模式
spring.shardingsphere.mode.type=Memory
# 打印SQl
spring.shardingsphere.props.sql-show=true
```

数据源配置

```properties
# 配置真实数据源
spring.shardingsphere.datasource.names=server-user,server-order0,server-order1

# 配置第 1 个数据源
spring.shardingsphere.datasource.server-user.type=com.zaxxer.hikari.HikariDataSource
spring.shardingsphere.datasource.server-user.driver-class-name=com.mysql.jdbc.Driver
spring.shardingsphere.datasource.server-user.jdbc-url=jdbc:mysql://127.0.0.1:3301/db_user
spring.shardingsphere.datasource.server-user.username=root
spring.shardingsphere.datasource.server-user.password=123456

# 配置第 2 个数据源
spring.shardingsphere.datasource.server-order0.type=com.zaxxer.hikari.HikariDataSource
spring.shardingsphere.datasource.server-order0.driver-class-name=com.mysql.jdbc.Driver
spring.shardingsphere.datasource.server-order0.jdbc-url=jdbc:mysql://127.0.0.1:3303/db_order
spring.shardingsphere.datasource.server-order0.username=root
spring.shardingsphere.datasource.server-order0.password=123456

# 配置第 3 个数据源
spring.shardingsphere.datasource.server-order1.type=com.zaxxer.hikari.HikariDataSource
spring.shardingsphere.datasource.server-order1.driver-class-name=com.mysql.jdbc.Driver
spring.shardingsphere.datasource.server-order1.jdbc-url=jdbc:mysql://127.0.0.1:3304/db_order
spring.shardingsphere.datasource.server-order1.username=root
spring.shardingsphere.datasource.server-order1.password=123456
```

标准分片表配置

```properties
#========================标准分片表配置（数据节点配置）
# spring.shardingsphere.rules.sharding.tables.<table-name>.actual-data-nodes=值
# 值由数据源名 + 表名组成，以小数点分隔。多个表以逗号分隔，支持 inline 表达式。
# <table-name>：逻辑表名
spring.shardingsphere.rules.sharding.tables.t_user.actual-data-nodes=server-user.t_user
spring.shardingsphere.rules.sharding.tables.t_order.actual-data-nodes=server-order0.t_order0,server-order0.t_order1,server-order1.t_order0,server-order1.t_order1
```

行表达式 （用于简化）

```properties
spring.shardingsphere.rules.sharding.tables.t_user.actual-data-nodes=server-user.t_user
spring.shardingsphere.rules.sharding.tables.t_order.actual-data-nodes=server-order$->{0..1}.t_order$->{0..1}
```

分片算法配置

#### 水平分库：

分片规则：Order中user_id为偶数时，写入order0服务器， user_id为奇数时，写入order1服务器

```properties
# -----分库策略
# 分片列
spring.shardingsphere.rules.sharding.tables.t_order.database-strategy.standard.sharding-column=user_id
spring.shardingsphere.rules.sharding.tables.t_order.database-strategy.standard.sharding-algorithm-name=alg_inline_userid

# 行表达式分片算法
spring.shardingsphere.rules.sharding.sharding-algorithms.alg_inline_userid.type=INLINE
spring.shardingsphere.rules.sharding.sharding-algorithms.alg_inline_userid.props.algorithm-expression=server-order$->{user_id % 2}

# 取模分片算法
spring.shardingsphere.rules.sharding.sharding-algorithms.alg_mod.type=MOD
spring.shardingsphere.rules.sharding.sharding-algorithms.alg_mod.props.sharding-count=2
```

测试 (只在order_0上测试)

```java
@SpringBootTest
public class HorizontalTest {
    @Autowired
    private OrderService orderService;

    @Test
    public void test() {
        for (long i = 0; i < 100; i++) {
            Order order = new Order();
            order.setOrderNo("order_" + i);
            order.setUserId(i+1);
            order.setAmount(new BigDecimal(i*100));
            orderService.save(order);
        }
    }
}

```

#### 水平分表

规则： order表中order_no的哈希值为偶数时，写入t_order0表，为奇数时写入t_order1表

```properties
#------------------------分表策略
# 分片列名称
spring.shardingsphere.rules.sharding.tables.t_order.table-strategy.standard.sharding-column=order_no
# 分片算法名称
spring.shardingsphere.rules.sharding.tables.t_order.table-strategy.standard.sharding-algorithm-name=alg_hash_mod


#------------------------分片算法配置
# 哈希取模分片算法
# 分片算法类型
spring.shardingsphere.rules.sharding.sharding-algorithms.alg_hash_mod.type=HASH_MOD
# 分片算法属性配置
spring.shardingsphere.rules.sharding.sharding-algorithms.alg_hash_mod.props.sharding-count=2
```

测试

```java
@Test
    public void test2(){
        for (long i = 100; i < 200; i++) {
            Order order = new Order();
            order.setOrderNo("order_" + i);
            order.setUserId(1000L);
            order.setAmount(new BigDecimal(i*100));
            orderService.save(order);
        }

        for(long i = 200; i < 300; i++){
            Order order = new Order();
            order.setOrderNo("order_" + i);
            order.setUserId(1001L);
            order.setAmount(new BigDecimal(i*100));
            orderService.save(order);
        }


    }

@Test
    public void selectAll() {
        orderService.list().forEach(System.out::println);
    }

    @Test
    public void selectByid() {
        LambdaQueryWrapper<Order> queryWrapper = new LambdaQueryWrapper<>();
        queryWrapper.eq(Order::getUserId, 1000L);
        orderService.list(queryWrapper).forEach(System.out::println);
    }
```

