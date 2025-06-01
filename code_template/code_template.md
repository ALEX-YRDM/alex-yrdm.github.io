# 通用代码片段

## Maven配置

```xml
<!--版本配置-->
<properties>
        <maven.compiler.source>17</maven.compiler.source>
        <maven.compiler.target>17</maven.compiler.target>
        <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
        <spring-ai.version>1.0.0-M6</spring-ai.version>
    	<fastjson.version>2.0.28</fastjson.version>
    </properties>

<!--依赖版本管理-->
<dependencyManagement>
    <dependencies>
        <!--spring-ai-bom 是 Spring AI 项目中的一个 Bill of Materials（BOM，物料清单）模块。它的主要作用是为 Spring AI 项目中的各个子模块提供统一的版本管理，确保不同模块之间的版本一致性，简化依赖管理。
	spring-ai-bom 包含了 Spring AI 项目中所有模块的版本定义，例如：

    spring-ai-core
    spring-ai-openai
    spring-ai-anthropic
    spring-ai-google-ai
    spring-ai-mistral-ai
    spring-ai-azure-openai
    spring-ai-prompt
    等等
-->
        <dependency>
            <groupId>org.springframework.ai</groupId>
            <artifactId>spring-ai-bom</artifactId>
            <version>${spring-ai.version}</version>
            <type>pom</type>
            <scope>import</scope>
        </dependency>
        <!--java对象和json字符串互转-->
        <dependency>
            <groupId>com.alibaba</groupId>
            <artifactId>fastjson</artifactId>
            <version>${fastjson.version}</version>
        </dependency>
    </dependencies>
</dependencyManagement>


<!--SpringBoot parent-->
<parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-parent</artifactId>
    <version>3.4.3</version>
    <relativePath/>
</parent>

<!--具体依赖-->
<!--Web应用, controller Tomcat--> 
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
</dependency>
<!--单元测试 集成测试-->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-test</artifactId>
    <scope>test</scope>
</dependency>
<!--Spring ai openai 和 ollama相关-->
<dependency>
    <groupId>org.springframework.ai</groupId>
    <artifactId>spring-ai-openai-spring-boot-starter</artifactId>
</dependency>

<dependency>
    <groupId>org.springframework.ai</groupId>
    <artifactId>spring-ai-ollama-spring-boot-starter</artifactId>
</dependency>
<!--提供统一的 API 接口来调用不同厂商的大模型服务（如 OpenAI、Anthropic、Google、Azure 等）
支持 Prompt 构建、消息对话管理
封装了基础的客户端抽象（如 AiClient、ChatClient、EmbeddingClient）
提供响应流式处理支持
支持函数调用（Function Calling）、工具调用（Tool Calling）
spring-ai-core 只是一个抽象层，实际使用时需要配合具体实现模块，例如：
spring-ai-openai（OpenAI）
spring-ai-anthropic（Claude）
spring-ai-google-ai（Gemini）
spring-ai-azure-openai（Azure OpenAI）
-->
<dependency>
    <groupId>org.springframework.ai</groupId>
    <artifactId>spring-ai-core</artifactId>
</dependency>
<!--mcp related-->
<dependency>
    <groupId>org.springframework.ai</groupId>
    <artifactId>spring-ai-mcp-server-spring-boot-starter</artifactId>
</dependency>

<dependency>
    <groupId>org.springframework.ai</groupId>
    <artifactId>spring-ai-mcp-client-webflux-spring-boot-starter</artifactId>
</dependency>
 <!--文档解析-->
<dependency>
    <groupId>org.springframework.ai</groupId>
    <artifactId>spring-ai-tika-document-reader</artifactId>
</dependency>
<!--中文分词器-->
<dependency>
    <groupId>com.huaban</groupId>
    <artifactId>jieba-analysis</artifactId>
    <version>1.0.2</version>
</dependency>
<!--pgvector 向量数据库-->
<dependency>
    <groupId>org.springframework.ai</groupId>
    <artifactId>spring-ai-pgvector-store</artifactId>
</dependency>
<!--lombok-->
<dependency>
    <groupId>org.projectlombok</groupId>
    <artifactId>lombok</artifactId>
</dependency>

<dependency>
    <groupId>com.alibaba</groupId>
    <artifactId>fastjson</artifactId>
</dependency>
<!--单测-->
<dependency>
    <groupId>junit</groupId>
    <artifactId>junit</artifactId>
    <scope>test</scope>
</dependency>
<!--使用注解定义Http请求,内置使用Okhttp发起请求-->
<dependency>
    <groupId>com.squareup.retrofit2</groupId>
    <artifactId>retrofit</artifactId>
    <version>2.9.0</version>
</dependency>
<!--这是一个 Jackson JSON 转换器 ，它让 Retrofit 能够将 HTTP 响应体自动转换为 Java 对象（POJO）。-->
<dependency>
    <groupId>com.squareup.retrofit2</groupId>
    <artifactId>converter-jackson</artifactId>
    <version>2.9.0</version>
</dependency>
<!--这个适配器让你可以在接口中直接返回 Observable<T>、Single<T> 等 RxJava2 类型 ，从而实现响应式编程。-->
<dependency>
    <groupId>com.squareup.retrofit2</groupId>
    <artifactId>adapter-rxjava2</artifactId>
    <version>2.9.0</version>
</dependency>
<!--一个非常强大的 Markdown 解析与渲染库-->
<dependency>
    <groupId>com.vladsch.flexmark</groupId>
    <artifactId>flexmark-all</artifactId>
    <version>0.64.8</version>
</dependency>
<!--提供一系列实用的 Java 基础类工具方法 ，特别是字符串、数组、日期、系统信息等操作。

🔧 主要功能：
字符串处理：StringUtils.isNotBlank(), StringUtils.abbreviate()
对象辅助：Objects.equals(), ObjectUtils.defaultIfNull()
枚举工具：EnumUtils
系统属性：SystemUtils
异常封装：ExceptionUtils-->
<dependency>
    <groupId>org.apache.commons</groupId>
    <artifactId>commons-lang3</artifactId>
    <version>3.9</version>
</dependency>
<!--Google 开源的 Java 核心库，提供了大量增强版的集合类、缓存、函数式编程、IO 操作、并发工具等-->
<dependency>
    <groupId>com.google.guava</groupId>
    <artifactId>guava</artifactId>
    <version>32.1.3-jre</version>
</dependency>
<!--这是 Redisson 提供的与 Spring Boot 集成的启动器模块 功能比spring-boot-starter-data-redis强大的多-->
<dependency>
    <groupId>org.redisson</groupId>
    <artifactId>redisson-spring-boot-starter</artifactId>
    <version>3.44.0</version>
</dependency>
<!--操作git仓库-->
<dependency>
    <groupId>org.eclipse.jgit</groupId>
    <artifactId>org.eclipse.jgit</artifactId>
    <version>5.13.0.202109080827-r</version>
</dependency>

<!--build-->
 <build>
     <finalName>XXX-app</finalName>
     <resources>
         <resource>
             <directory>src/main/resources</directory>
             <filtering>true</filtering>
             <includes>
                 <include>**/**</include>
             </includes>
         </resource>
     </resources>
     <testResources>
         <testResource>
             <directory>src/test/resources</directory>
             <filtering>true</filtering>
             <includes>
                 <include>**/**</include>
             </includes>
         </testResource>
     </testResources>
     <plugins>
         <plugin>
             <groupId>org.apache.maven.plugins</groupId>
             <artifactId>maven-surefire-plugin</artifactId>
             <version>2.6</version>
             <configuration>
                 <skipTests>true</skipTests>
                 <testFailureIgnore>false</testFailureIgnore>
                 <includes>
                     <include>**/*Test.java</include>
                 </includes>
             </configuration>
         </plugin>
         <plugin>
             <groupId>org.springframework.boot</groupId>
             <artifactId>spring-boot-maven-plugin</artifactId>
             <version>3.4.3</version>
             <configuration>
                 <mainClass>top.stackpop.Application</mainClass>
                 <layout>JAR</layout>
                 <excludes>
                     <exclude>
                         <groupId>org.projectlombok</groupId>
                         <artifactId>lombok</artifactId>
                     </exclude>
                 </excludes>
             </configuration>
         </plugin>
         <!-- Maven 编译插件，启用 Lombok -->
         <plugin>
             <groupId>org.apache.maven.plugins</groupId>
             <artifactId>maven-compiler-plugin</artifactId>
             <version>3.13.0</version>
             <configuration>
                 <source>17</source> <!-- 根据项目实际 JDK 版本调整 -->
                 <target>17</target>
                 <annotationProcessorPaths>
                     <path>
                         <groupId>org.projectlombok</groupId>
                         <artifactId>lombok</artifactId>
                     </path>
                 </annotationProcessorPaths>
             </configuration>
         </plugin>

         <!-- Spring Boot 打包插件 -->
         <plugin>
             <groupId>org.springframework.boot</groupId>
             <artifactId>spring-boot-maven-plugin</artifactId>
             <configuration>
                 <excludes>
                     <exclude>
                         <groupId>org.projectlombok</groupId>
                         <artifactId>lombok</artifactId>
                     </exclude>
                 </excludes>
             </configuration>
         </plugin>
     </plugins>
</build>

<!--jvm启动参数-->
<profiles>
    <profile>
        <id>dev</id>
        <activation>
            <activeByDefault>true</activeByDefault>
        </activation>
        <properties>
            <java_jvm>-Xms1G -Xmx1G -server  -XX:MaxPermSize=256M -Xss256K -Dspring.profiles.active=test -XX:+DisableExplicitGC -XX:+UseG1GC  -XX:LargePageSizeInBytes=128m -XX:+UseFastAccessorMethods -XX:+HeapDumpOnOutOfMemoryError -XX:HeapDumpPath=/export/Logs/xfg-frame-archetype-lite-boot -Xloggc:/export/Logs/xfg-frame-archetype-lite-boot/gc-xfg-frame-archetype-lite-boot.log -XX:+PrintGCDetails -XX:+PrintGCDateStamps</java_jvm>
            <profileActive>dev</profileActive>
        </properties>
    </profile>
    <profile>
        <id>test</id>
        <properties>
            <java_jvm>-Xms1G -Xmx1G -server  -XX:MaxPermSize=256M -Xss256K -Dspring.profiles.active=test -XX:+DisableExplicitGC -XX:+UseG1GC  -XX:LargePageSizeInBytes=128m -XX:+UseFastAccessorMethods -XX:+HeapDumpOnOutOfMemoryError -XX:HeapDumpPath=/export/Logs/xfg-frame-archetype-lite-boot -Xloggc:/export/Logs/xfg-frame-archetype-lite-boot/gc-xfg-frame-archetype-lite-boot.log -XX:+PrintGCDetails -XX:+PrintGCDateStamps</java_jvm>
            <profileActive>test</profileActive>
        </properties>
    </profile>
    <profile>
        <id>prod</id>
        <properties>
            <java_jvm>-Xms6G -Xmx6G -server  -XX:MaxPermSize=256M -Xss256K -Dspring.profiles.active=release -XX:+DisableExplicitGC -XX:+UseG1GC  -XX:LargePageSizeInBytes=128m -XX:+UseFastAccessorMethods -XX:+HeapDumpOnOutOfMemoryError -XX:HeapDumpPath=/export/Logs/fq-mall-activity-app -Xloggc:/export/Logs/xfg-frame-archetype-lite-boot/gc-xfg-frame-archetype-lite-boot.log -XX:+PrintGCDetails -XX:+PrintGCDateStamps</java_jvm>
            <profileActive>prod</profileActive>
        </properties>
    </profile>
</profiles>

```

## Application.yml

```yml
spring:
  application:
    name: ai-rag
  profiles:
    active: dev
```

## Application-dev.yml

```yml
server:
  port: 80

spring:
  mvc:
    async:
      request-timeout: 0
  datasource:
    driver-class-name: org.postgresql.Driver
    username: postgres
    password: postgres
    url: jdbc:postgresql://localhost:5432/springai
    type: com.zaxxer.hikari.HikariDataSource
  ai:
    ollama:
      base-url: http://localhost:11434
      embedding:
        options:
          num-batch: 512
        model: nomic-embed-text
    openai:
      base-url: 
      api-key: 
      embedding:
        options:
          model: text-embedding-ada-002
    rag:
      embed: text-embedding-ada-002

redis:
  sdk:
    config:
      host: 127.0.0.1
      port: 6379
      pool-size: 10
      min-idle-size: 5
      idle-timeout: 30000
      connect-timeout: 5000
      retry-attempts: 3
      retry-interval: 1000
      ping-interval: 60000
      keep-alive: true

logging:
  level:
    root: info
  config: classpath:logback-spring.xml
```

## log.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!-- 日志级别从低到高分为TRACE < DEBUG < INFO < WARN < ERROR < FATAL，如果设置为WARN，则低于WARN的信息都不会输出 -->
<configuration scan="true" scanPeriod="10 seconds">

    <contextName>logback</contextName>
    <!-- name的值是变量的名称，value的值时变量定义的值。通过定义的值会被插入到logger上下文中。定义变量后，可以使“${}”来使用变量。 -->
    <springProperty scope="context" name="log.path" source="logging.path"/>
    <!-- 日志格式 -->
    <conversionRule conversionWord="clr" converterClass="org.springframework.boot.logging.logback.ColorConverter"/>
    <conversionRule conversionWord="wex"
                    converterClass="org.springframework.boot.logging.logback.WhitespaceThrowableProxyConverter"/>
    <conversionRule conversionWord="wEx"
                    converterClass="org.springframework.boot.logging.logback.ExtendedWhitespaceThrowableProxyConverter"/>

    <!-- 输出到控制台 -->
    <appender name="CONSOLE" class="ch.qos.logback.core.ConsoleAppender">
        <!-- 此日志appender是为开发使用，只配置最底级别，控制台输出的日志级别是大于或等于此级别的日志信息 -->
        <filter class="ch.qos.logback.classic.filter.ThresholdFilter">
            <level>info</level>
        </filter>
        <encoder>
            <pattern>%d{yy-MM-dd.HH:mm:ss.SSS} [%-16t] %-5p %-22c{0}%X{ServiceId} -%X{trace-id} %m%n</pattern>
            <charset>UTF-8</charset>
        </encoder>
    </appender>

    <!--输出到文件-->
    <!-- 时间滚动输出 level为 INFO 日志 -->
    <appender name="INFO_FILE" class="ch.qos.logback.core.rolling.RollingFileAppender">
        <!-- 正在记录的日志文件的路径及文件名 -->
        <file>./data/log/log_info.log</file>
        <!--日志文件输出格式-->
        <encoder>
            <pattern>%d{yy-MM-dd.HH:mm:ss.SSS} [%-16t] %-5p %-22c{0}%X{ServiceId} -%X{trace-id} %m%n</pattern>
            <charset>UTF-8</charset>
        </encoder>
        <!-- 日志记录器的滚动策略，按日期，按大小记录 -->
        <rollingPolicy class="ch.qos.logback.core.rolling.TimeBasedRollingPolicy">
            <!-- 每天日志归档路径以及格式 -->
            <fileNamePattern>./data/log/log-info-%d{yyyy-MM-dd}.%i.log</fileNamePattern>
            <timeBasedFileNamingAndTriggeringPolicy class="ch.qos.logback.core.rolling.SizeAndTimeBasedFNATP">
                <maxFileSize>100MB</maxFileSize>
            </timeBasedFileNamingAndTriggeringPolicy>
            <!--日志文件保留天数-->
            <maxHistory>15</maxHistory>
            <totalSizeCap>10GB</totalSizeCap>
        </rollingPolicy>
    </appender>

    <!-- 时间滚动输出 level为 ERROR 日志 -->
    <appender name="ERROR_FILE" class="ch.qos.logback.core.rolling.RollingFileAppender">
        <!-- 正在记录的日志文件的路径及文件名 -->
        <file>./data/log/log_error.log</file>
        <!--日志文件输出格式-->
        <encoder>
            <pattern>%d{yy-MM-dd.HH:mm:ss.SSS} [%-16t] %-5p %-22c{0}%X{ServiceId} -%X{trace-id} %m%n</pattern>
            <charset>UTF-8</charset>
        </encoder>
        <!-- 日志记录器的滚动策略，按日期，按大小记录 -->
        <rollingPolicy class="ch.qos.logback.core.rolling.TimeBasedRollingPolicy">
            <fileNamePattern>./data/log/log-error-%d{yyyy-MM-dd}.%i.log</fileNamePattern>
            <timeBasedFileNamingAndTriggeringPolicy class="ch.qos.logback.core.rolling.SizeAndTimeBasedFNATP">
                <maxFileSize>100MB</maxFileSize>
            </timeBasedFileNamingAndTriggeringPolicy>
            <!-- 日志文件保留天数【根据服务器预留，可自行调整】 -->
            <maxHistory>7</maxHistory>
            <totalSizeCap>5GB</totalSizeCap>
        </rollingPolicy>
        <!-- WARN 级别及以上 -->
        <filter class="ch.qos.logback.classic.filter.ThresholdFilter">
            <level>WARN</level>
        </filter>
    </appender>

    <!-- 异步输出 -->
    <appender name="ASYNC_FILE_INFO" class="ch.qos.logback.classic.AsyncAppender">
        <!-- 队列剩余容量小于discardingThreshold,则会丢弃TRACT、DEBUG、INFO级别的日志;默认值-1,为queueSize的20%;0不丢失日志 -->
        <discardingThreshold>0</discardingThreshold>
        <!-- 更改默认的队列的深度,该值会影响性能.默认值为256 -->
        <queueSize>8192</queueSize>
        <!-- neverBlock:true 会丢失日志,但业务性能不受影响 -->
        <neverBlock>true</neverBlock>
        <!--是否提取调用者数据-->
        <includeCallerData>false</includeCallerData>
        <appender-ref ref="INFO_FILE"/>
    </appender>

    <appender name="ASYNC_FILE_ERROR" class="ch.qos.logback.classic.AsyncAppender">
        <!-- 队列剩余容量小于discardingThreshold,则会丢弃TRACT、DEBUG、INFO级别的日志;默认值-1,为queueSize的20%;0不丢失日志 -->
        <discardingThreshold>0</discardingThreshold>
        <!-- 更改默认的队列的深度,该值会影响性能.默认值为256 -->
        <queueSize>1024</queueSize>
        <!-- neverBlock:true 会丢失日志,但业务性能不受影响 -->
        <neverBlock>true</neverBlock>
        <!--是否提取调用者数据-->
        <includeCallerData>false</includeCallerData>
        <appender-ref ref="ERROR_FILE"/>
    </appender>

    <!-- 开发环境：控制台打印 -->
    <springProfile name="dev">
        <logger name="com.nmys.view" level="debug"/>
    </springProfile>

    <root level="info">
        <appender-ref ref="CONSOLE"/>
        <!-- 异步日志-INFO -->
        <appender-ref ref="ASYNC_FILE_INFO"/>
        <!-- 异步日志-ERROR -->
        <appender-ref ref="ASYNC_FILE_ERROR"/>
    </root>

</configuration>

```

## Dockerfile

```dockerfile
FROM openjdk:17-jdk-slim

MAINTAINER yrdm2002

ENV PARAMS=""

# 时区
ENV TZ=PRC
RUN ln -snf /usr/share/zoneinfo/$TZ /etc/localtime && echo $TZ > /etc/timezone

ADD target/ai-rag-knowledge-app.jar /app.jar

ENTRYPOINT ["sh","-c","java -jar $JAVA_OPTS /app.jar  $PARAMS"]
```

## Docker-Compose.yml

```yml
version: '3'
services:
  ollama:
    image: ollama/ollama:0.7.1
    container_name: ollama
    restart: unless-stopped
    ports:
      - "11434:11434"
  redis:
    image: redis:6.2
    container_name: redis
    restart: always
    hostname: redis
    privileged: true
    ports:
      - "6379:6379"
    volumes:
      - ./redis/redis.conf:/usr/local/etc/redis/redis.conf
    command: redis-server /usr/local/etc/redis/redis.conf
    networks:
      - rag-network
    healthcheck:
      test: ["CMD","redis-cli","ping"]
      interval: 10s
      timeout: 5s
      retries: 3
  redis-admin:
    image: spryker/redis-commander:0.8.0
    container_name: redis-admin
    hostname: redis-commander
    restart: always
    ports:
      - "8081:8081"
    environment:
      - REDIS_HOSTS=local:redis:6379
      - HTTP_USER=admin
      - HTTP_PASSWORD=admin
      - LANG=C.UTF-8
      - LANGUAGE=C.UTF-8
      - LC_ALL=C.UTF-8
    networks:
      - rag-network
    depends_on:
      redis:
        condition: service_healthy
  vector_db:
    image: pgvector/pgvector:0.8.0-pg16
    container_name: vector_db
    restart: always
    environment:
      - POSTGRES_USER=postgres
      - POSTGRES_PASSWORD=postgres
      - POSTGRES_DB=springai
      - PGPASSWORD=postgres
    volumes:
      - ./pgvector/sql/init.sql:/docker-entrypoint-initdb.d/init.sql
    logging:
      options:
        max-size: 10m
        max-file: "3"
    ports:
      - "5432:5432"
    healthcheck:
      test: "pg_isready -U postgres -d vector_store"
      interval: 2s
      timeout: 20s
      retries: 10
    networks:
      - rag-network
  nginx:
    image: nginx:1.27.5-perl
    container_name: nginx
    restart: always
    ports:
      - "3000:80"
    volumes:
      - ./nginx/html:/usr/share/nginx/html
      - ./nginx/conf/nginx.conf:/etc/nginx/nginx.conf
      - ./nginx/conf/conf.d:/etc/nginx/conf.d
    privileged: true
  ai-rag-konwledge-app:
    image: yrdm2002/ai-rag-knowledge-app:1.0
    container_name: ai-rag-knowledge-app
    restart: on-failure
    ports:
      - "80:80"
    environment:
      - TZ=PRC
      - SERVER_PORT=80
      - SPRING_DATASOURCE_USERNAME=postgres
      - SPRING_DATASOURCE_PASSWORD=postgres
      - SPRING_DATASOURCE_URL=jdbc:postgresql://10.29.60.8:5432/springai
      - SPRING_DATASOURCE_DRIVER_CLASS_NAME=org.postgresql.Driver
      - SPRING_AI_OLLAMA_BASE_URL=http://10.29.60.8:11434
      - SPRING_AI_OLLAMA_EMBEDDING_OPTIONS_NUM_BATCH=512
      - SPRING_AI_OLLAMA_MODEL=nomic-embed-text
      - SPRING_AI_OPENAI_BASE_URL=
      - SPRING_AI_OPENAI_API_KEY=
      - SPRING_AI_OPENAI_EMBEDDING_MODEL=text-embedding-ada-002
      - SPRING_AI_RAG_EMBED=text-embedding-ada-002
      - REDIS_SDK_CONFIG_HOST=10.29.60.8
      - REDIS_SDK_CONFIG_PORT=6379
    volumes:
      - ./log:/data/log
    logging:
      driver: "json-file"
      options:
        max-size: "10m"
        max-file: "3"
    networks:
      - rag-network

networks:
  rag-network:
    driver: bridge
```

## Nginx.conf 主配置文件

```
user  nginx;
worker_processes  auto;

error_log  /var/log/nginx/error.log notice;
pid        /var/run/nginx.pid;


events {
    worker_connections  1024;
}


http {
    include       /etc/nginx/mime.types;
    default_type  application/octet-stream;

    log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
                      '$status $body_bytes_sent "$http_referer" '
                      '"$http_user_agent" "$http_x_forwarded_for"';

    access_log  /var/log/nginx/access.log  main;

    sendfile        on;
    #tcp_nopush     on;

    keepalive_timeout  65;

    #gzip  on;

    include /etc/nginx/conf.d/*.conf;
}
```

### 单站点

```
server {
    listen 80;

    server_name 127.0.0.1;  # 修改为你的实际服务器 IP 或域名【域名需要备案】

    location / {
        proxy_set_header   X-Real-IP         $remote_addr;
        proxy_set_header   Host              $http_host;
        proxy_set_header   X-Forwarded-For   $proxy_add_x_forwarded_for;
        root   /usr/share/nginx/html;
        index  index.html index.htm;
    }

    error_page   500 502 503 504  /50x.html;
    location = /50x.html {
        root   /usr/share/nginx/html;
    }

}
```

### 前后端分离 反向代理 (前端请求自动转发到127.0.0.1:3000)

```
server {
    listen 80;
    server_name api.example.com;

    location / {
        proxy_pass http://127.0.0.1:3000;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
        proxy_connect_timeout 60s;
        proxy_read_timeout 120s;
    }

    access_log /var/log/nginx/api_access.log;
    error_log /var/log/nginx/api_error.log;
}
```

### 负载均衡 

```
upstream backend_servers {
    least_conn;
    server 192.168.1.10:3000 weight=3;
    server 192.168.1.11:3000;
    server 192.168.1.12:3000 backup;
}

server {
    listen 80;
    server_name app.example.com;

    location / {
        proxy_pass http://backend_servers;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }

    access_log /var/log/nginx/app_access.log;
    error_log /var/log/nginx/app_error.log;
}
```

## .gitignore

```
target/
!.mvn/wrapper/maven-wrapper.jar
!**/src/main/**/target/
!**/src/test/**/target/

### IntelliJ IDEA ###
.idea/modules.xml
.idea/jarRepositories.xml
.idea/compiler.xml
.idea/libraries/
*.iws
*.iml
*.ipr
.idea

### Eclipse ###
.apt_generated
.classpath
.factorypath
.project
.settings
.springBeans
.sts4-cache

### NetBeans ###
/nbproject/private/
/nbbuild/
/dist/
/nbdist/
/.nb-gradle/
build/
!**/src/main/**/build/
!**/src/test/**/build/

### VS Code ###
.vscode/

### Mac OS ###
.DS_Store

*.log
```

