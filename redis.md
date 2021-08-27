# 一、redis

### 1、springBoot整合Redis实例

#### 1、依赖

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-redis</artifactId>
    <version>2.1.5.RELEASE</version>
</dependency>
<!--连接池，redis需要-->
<dependency>
    <groupId>org.apache.commons</groupId>
    <artifactId>commons-pool2</artifactId>
</dependency> 
```

#### 2、配置

```yml
spring:
  redis:
    database: 0
    host: 127.0.0.1
    port: 6379
```

#### 3、使用

```java
redisTemplate.opsForValue().set("student", student);
redisTemplate.opsForValue().get(key);
redisTemplate.delete(key);
```




