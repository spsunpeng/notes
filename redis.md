# 快速使用

## 1、启动

```sh
#============================linxu单机======================================
cd /usr/local/redis/bin
./redis-server #启动server
./redis-cli #启动client
.redis-cli -p 7006 shutdown #关闭server
#注意：单机与集群在同一个redis中，启动使用后会生成持久化文件，如需在单机和集群间切换，需要删除持久化文件dump.rdb、appendonly.aof

#=============================linux集群=========================================
#已配置守护进程，已配置集群，已配置主从, 当前ip：10.1.20.122
#集群port: 7001、7002、7003、7004、7005、7006，形成一主一从三节点集群
cd /usr/local/redis/bin
./tartup.sh             #启动server
./redis-cli -p 7001 -c  #启动client
./stop.sh               #关闭server
#注意：单机与集群在同一个redis中，启动使用后会生成持久化文件，如需在单机和集群间切换，需要删除持久化文件dump.rdb、appendonly.aof

#=============================window单机=========================================
cd E:\redis\Redis-x64-5.0.9  #redis安装目录
redis-server.exe #启动服务端
redis-cli.exe    #启动客户端
```

## 2、命令

```sh
#=============================================通用===========================================
keys "*[key]*"   #查询key，模糊匹配
redis-cli  cluster nodes  #查询节点信息

#=============================================string=========================================
get [key]   #查询key，模糊匹配
set [key] [value]
ttl [key] #获取过期时间

#==============================================hash==========================================
hgetall [key] #获取所有key-value
hkeys [key] #获取所有key
hvals [key] #获取所有value

hget [key] [field] #获取value
hset [key] [field] [value] #设置key-value
hdel [key] [field field2 field3 ...] #删除key


#=========================================== 集群&keys ==========================================
#1.redis-cli内部命令（集群环境下无法模糊操作）
redis-cli -c
127.0.0.1:6379> keys "*[key]*"

#2.redis-cli命令（集群环境下可以模糊查询，但无法模糊删除）
redis-cli --scan --pattern "*[key]*" #主从节点，结果：key:vale
redis-cli --cluster call [ip:port] keys "*[key]*" #全部（任意一个节点ip:port即可）结果：port:key:value
redis-cli -h [ip] keys "*[key]*" #主从节点，结果：key
redis-cli -h [ip] keys | xargs redis-cli -h [ip] del #是否可以遍历容器删除，不可以，删除时主从之间跨槽

#3.shell命令（集群环境下模糊删除）
#将所有key查询出来，记录到文件中，用 redis-cli del [key] 命令一个一个删除，这样就不会跨槽了
redis-cli -h 127.0.0.1 -p 6379 -a 密码 cluster nodes | grep master | awk '{print $2}' | awk -F ':' '{print " -h " $1 " -p " $2}' > redis_object_port.info
more redis_object_port.info | while read object; do redis-cli $object -a 密码 keys mallvopdev_product::detail*; done > result.txt
sed -i 's/^/del &/' result.txt
cat result.txt|redis-cli -c -a 密码
rm -f redis_object_port.info
rm -f result.txt

#4.以redis客户端方式：可以（如Another Redis Desktop Manager）

#5.以java代码的方式：可以
```





# 一、redis

## 1、安装

### 1.1 linux安装

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





## 2、命令

```sh
#===========================================通用====================================
keys *[key]*   #查询key，模糊匹配

#===========================================string====================================
get [key]   #查询key，模糊匹配
set [key] [value]
ttl [key] #获取过期时间

#===========================================hash======================================
hgetall [key] #获取所有key-value
hkeys [key] #获取所有key
hvals [key] #获取所有value

hget [key] [field] #获取value
hset [key] [field] [value] #设置key-value
hdel [key] [field field2 field3 ...] #删除key

#===========================================scan======================================
redis-cli --scan --pattern *[key]* #scan：遍历；pattern：匹配
```







## 3、代码

### 3.1 springBoot整合Redis实例

#### 3.1.1 依赖

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-redis</artifactId>
    <version>2.1.5.RELEASE</version>
</dependency>
```

#### 3.1.2 application

```yml
spring:
  redis:
    host: 10.1.20.122
    cluster:
      nodes: 10.1.20.122:7001,10.1.20.122:7002,10.1.20.122:7003,10.1.20.122:7004,10.1.20.122:7005,10.1.20.122:7006 #不配是单子版（默认port为6379），配置为集群
```

#### 3.1.3 RedisConfig

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

#### 3.1.4 使用

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



## 4、Redis持久化策略

### 4.1 RDB

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

### 4.2 AOF

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