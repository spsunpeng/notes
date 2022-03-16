# 0、当前

目前redis已经安装配置完成，可以直接使用

```sh
#已配置守护进程，已配置集群，已配置主从
#当前ip：10.1.20.122

###单机
/usr/local/redis/bin/redis-server #启动server
/usr/local/redis/bin/redis-cli #启动client
/usr/local/redis/bin/redis-cli -p 7006 shutdown #关闭server

###集群
#集群port: 7001、7002、7003、7004、7005、7006，形成一主一从三节点集群
/usr/local/redis/bin/startup.sh #启动server
/usr/local/redis/bin/redis-cli -p 7001 -c #启动client
/usr/local/redis/bin/stop.sh #关闭server

###注意
#单机与集群在同一个redis中，启动使用后会生成持久化文件，如需在单机和集群间切换，需要删除持久化文件dump.rdb、appendonly.aof
```





# 一、安装

## 1、linux安装

```sh
###1.资源
#本机资源在/usr/local/tmp/redis-5.0.5.tar.g中，也可另寻资源
tar zxf redis-5.0.5.tar.gz

###2.安装
yum install -y gcc-c++ automake autoconf libtool make tcl #redis依赖的C库
cd /usr/local/tmp/redis-5.0.5/
make
make install PREFIX=/usr/local/redis

###3.启动服务端
./redis-server redis.conf #redis-server直接开启会占据主进程（终端命令行），需要在守护进程
##修改配置文件
vim /usr/local/redis/bin/redis.conf
	daemonize yes
	#bind 127.0.0.1
	protected-mode no
kill port                 #重启redis-server
./redis-server redis.conf #重启redis-server

###4.客户端测试
./redis-cli #进入客户端
set [key] [value] #设置
get [key]         #获取
 
###5.主从
#优点：增加单一节点的健壮性，从而提升整个集群的稳定性，读写分离
mkdir /usr/local/replica
cp -r /usr/local/redis/bin /usr/local/replica/master
cp -r /usr/local/redis/bin /usr/local/replica/slave1
cp -r /usr/local/redis/bin /usr/local/replica/slave2
vim /usr/local/replica/slave1/redis.conf
	replicaof 10.1.20.122 6379
	port 6380
vim /usr/local/replica/slave1/redis.conf
	replicaof 10.1.20.122 6379
	port 6381
#编写启动文件	
vim startup.sh
	cd /usr/local/replica/master/
	./redis-server redis.conf
 	 cd /usr/local/replica/slave1
	./redis-server redis.conf
    cd /usr/local/replica/slave2
	./redis-server redis.conf
chmod a+x startup.sh #给启动文件授权
./startup.sh  #启动，注意关闭单机版，连个redis-server端口号冲突
ps aux|grep redis #查看状态

###6.哨兵
#解决主宕机而从不能写的问题，选出新的主
mkdir /usr/local/sentinel
cp -r /usr/local/redis/bin/* /usr/local/sentinel
cp /usr/local/tmp/redis-5.0.5/sentinel.conf /usr/local/sentinel/
vim /usr/local/sentinel/sentinel.conf
	port 26379 #不变
	daemonize yes 
	logfile “/usr/local/sentinel/26379.log”
	sentinel monitor mymaster 10.1.20.122 6379 2
cp sentinel.conf sentinel-26380.conf
vim sentinel-26380.conf
	port 26380
cp sentinel.conf sentinel-26381.conf
vim sentinel-26380.conf
	port 26381

###7.集群
/usr/local/replica/startup.sh #启动redis主从服务，同样要防止redis端口会冲突
#启动三个哨兵
cd /usr/local/sentinel
./redis-sentinel sentinel.conf
./redis-sentinel sentinel-26380.conf
./redis-sentinel sentinel-26381.conf

###8.主从与哨兵测试
ps aux|grep redis
kill -9 [pid] 
cat 26379.log #查看哨兵日志
#通过客户端查看
./rediscli  #启动主客户端
./rediscli -p 6380 #启动从客户端
info replication #查看redi状态信息（包括主从）

###9.集群
##配置文件
cp /usr/local/redis/bin/redis.conf /usr/local/redis/bin/redis-7001.conf
vim /usr/local/redis/bin/redis-7001.conf
	port 7001
	cluster-enabled yes
	cluster-config-file nodes-7001.conf
	cluster-node-timeout 15000
	# appendonly yes 如果开启aof默认，需要修改为yes。如果使用rdb，此处不需要修改
	daemonize yes
	protected-mode no
	pidfile /var/run/redis_7001.pid
cp /usr/local/redis/bin/redis-7001.conf /usr/local/redis/bin/redis-7002.conf
cp /usr/local/redis/bin/redis-7001.conf /usr/local/redis/bin/redis-7003.conf
cp /usr/local/redis/bin/redis-7001.conf /usr/local/redis/bin/redis-7004.conf
cp /usr/local/redis/bin/redis-7001.conf /usr/local/redis/bin/redis-7005.conf
cp /usr/local/redis/bin/redis-7001.conf /usr/local/redis/bin/redis-7006.conf
vim /usr/local/redis/bin/redis-7002.conf #其他四个同理
	:%s/7001/7002/g
##启动脚本
vim startup.sh
    ./redis-server redis-7001.conf
    ./redis-server redis-7002.conf
    ./redis-server redis-7003.conf
    ./redis-server redis-7004.conf
    ./redis-server redis-7005.conf
    ./redis-server redis-7006.conf
chmod a+x startup.sh
##关闭脚本
vim stop.sh
    ./redis-cli -p 7001 shutdown
    ./redis-cli -p 7002 shutdown
    ./redis-cli -p 7003 shutdown
    ./redis-cli -p 7004 shutdown
    ./redis-cli -p 7005 shutdown
    ./redis-cli -p 7006 shutdown
chmod a+x stop.sh
##建立集群
./redis-cli --cluster create 10.1.20.122:7001 10.1.20.122:7002 10.1.20.122:7003 10.1.20.122:7004 10.1.20.122:7005 10.1.20.122:7006 --cluster-replicas 1 #建立一主一从三个集群，并且生成node.conf配置文件
##启动
/usr/local/redis/bin/startup.sh #启动server
/usr/local/redis/bin/redis-cli -p 7001 -c #启动client
/usr/local/redis/bin/stop.sh #关闭server
```





# 二、使用





# 三、代码

## 1、springBoot整合Redis实例

### 1、依赖

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-redis</artifactId>
    <version>2.1.5.RELEASE</version>
</dependency>
```

### 2、application

```yml
spring:
  redis:
    host: 10.1.20.122
    cluster:
      nodes: 10.1.20.122:7001,10.1.20.122:7002,10.1.20.122:7003,10.1.20.122:7004,10.1.20.122:7005,10.1.20.122:7006 #不配是单子版（默认port为6379），配置为集群
```

### 3、RedisConfig

```java
package com.config;

import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.data.redis.connection.RedisConnectionFactory;
import org.springframework.data.redis.core.RedisTemplate;
import org.springframework.data.redis.core.StringRedisTemplate;
import org.springframework.data.redis.serializer.Jackson2JsonRedisSerializer;
import org.springframework.data.redis.serializer.StringRedisSerializer;

@Configuration
public class RedisConfig {
    @Bean
    public RedisTemplate<String,Object> redisTemplate(RedisConnectionFactory factory){
        RedisTemplate<String,Object> redisTemplate = new RedisTemplate<String, Object>();
        redisTemplate.setConnectionFactory(factory);
        redisTemplate.setKeySerializer(new StringRedisSerializer());
        redisTemplate.setValueSerializer(new Jackson2JsonRedisSerializer<Object>(Object.class));
        return redisTemplate;
    }

    @Bean
    public StringRedisTemplate stringRedisTemplate(RedisConnectionFactory factory){
        StringRedisTemplate stringRedisTemplate = new StringRedisTemplate();
        stringRedisTemplate.setConnectionFactory(factory);
        stringRedisTemplate.setKeySerializer(new StringRedisSerializer());
        stringRedisTemplate.setValueSerializer(new StringRedisSerializer());
        return stringRedisTemplate;
    }

}

```

### 4、使用

```java
@RestController
public class StudentController {

    @Autowired
    private RedisTemplate<String, Object> redisTemplate;

    @Autowired
    private StringRedisTemplate stringRedisTemplate;

    @GetMapping("/testString")
    public String testString(){
        stringRedisTemplate.opsForValue().set("name", "sunpeng")
        return stringRedisTemplate.opsForValue().get("name");
    }

    @PostMapping("/testObject")
    public Student testObject(@RequestBody Student student){
        redisTemplate.opsForValue().set("student1", student);
        redisTemplate.setValueSerializer(new Jackson2JsonRedisSerializer<Student>(Student.class)); //获取前需要序列化
        return (Student) redisTemplate.opsForValue().get("student1");
    }
}
```

# 四、Redis持久化策略

## 1、RDB

​	rdb模式是默认模式，可以在指定的时间间隔内生成数据快照（snapshot），默认保存到**dump.rdb**文件中。当redis重启后会自动加载dump.rdb文件中内容到内存中。

​	用户可以使用SAVE（同步）或BGSAVE（异步）手动保存数据。

​	可以设置服务器配置的save选项，让服务器每隔一段时间自动执行一次BGSAVE命令，可以通过save选项设置多个保存条件，但只要其中任意一个条件被满足，服务器就会执行BGSAVE命令。
　　	例如：
　　	save 900 1
　　	save 300 10
　　	save 60 10000
　　那么只要满足以下三个条件中的任意一个，BGSAVE命令就会被执行
　　服务器在900秒之内，对数据库进行了至少1次修改
　　服务器在300秒之内，对数据库进行了至少10次修改
　　服务器在60秒之内，对数据库进行了至少10000次修改

## 2、AOF

​	AOF默认是关闭的，需要在配置文件中开启AOF。Redis支持AOF和RDB同时生效，如果同时存在，AOF优先级高于RDB（Redis重新启动时会使用AOF进行数据恢复）

​	监听执行的命令，如果发现执行了修改数据的操作，同时直接同步到数据库文件中。

开启方法

```sh
# 默认no
appendonly yes
# aof文件名
appendfilename "appendonly.aof"
```

数据持久化在appendonly.aof中











| 地方   | 情况                                                   | 路程                   |
| ------ | ------------------------------------------------------ | ---------------------- |
| 乐华城 | **地上乐园营业中**；水上乐园夏季才营业。               | 北郊外，时间两个小时   |
| 白鹿原 | **白鹿原滑雪场**，白鹿原白鹿仓（乐园和文化），比较小。 | 东郊外，时间两个小时   |
| 欢乐谷 | 号称最大，水上5月份营业，地上还在建设中                | 西郊外，时间一个半小时 |



