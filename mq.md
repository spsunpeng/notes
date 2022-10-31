

# 当下

## 1、启动rabbitmq

```sh
cd /etc/local/rabbitmq/sbin
./rabbitmq-plugins enable rabbitmq_management #启动web
./rabbitmq-server -detached #后台启动，没有启动成功返回，只有一行警告日志
#访问：10.1.20.122:15672
```

注意：有ip配置，一旦虚拟机迁移或者网络变动，需要重新配置ip



# 一、rabbitmq

## 1、安装

```sh
#--------------------------------一、主机名映射-------------------------
#RabbitMQ是通过主机名进行访问的，必须指定能访问的主机名
vim /etc/sysconfig/network
	NETWORKING=yes
	HOSTNAME=smallming
vim /etc/hosts 
	10.1.20.122 smallming



#-------------------------------二、安装Erlang-------------------------
#1.资源
yum -y install make gcc gcc-c++ kernel-devel m4 ncurses-devel openssl-devel unixODBC unixODBC-devel
cd /usr/local/tmp #3 上传并解压
tar xf otp_src_22.0.tar.gz #解压时注意，此压缩包不具有gzip属性，解压参数没有z，只有xf
#2.配置
mkdir -p /usr/local/erlang
cd otp_src_22.0
./configure --prefix=/usr/local/erlang --with-ssl --enable-threads --enable-smp-support --enable-kernel-poll --enable-hipe --without-javac # 配置参数
#3. 编译并安装
make #编译 
make instal #安装
#4.修改环境变量
vim /etc/profile #export PATH=$PATH:/usr/local/erlang/bin
source /etc/profile #运行文件，让修改内容生效
#5.查看配置是否成功
erl -version


#----------------------------------三、安装RabbitMq---------------------------
#1.资源
#上传rabbitmq-server-generic-unix-3.7.18.tar.xz到/usr/loca/tmp中
cd /usr/local/tmp
tar xf rabbitmq-server-generic-unix-3.7.18.tar.xz
cp -r rabbitmq_server-3.7.18 /usr/local/rabbitmq
#2.配置环境变量
vim /etc/profile 
	export PATH=$PATH:/usr/local/rabbitmq/sbin
source /etc/profile
#3.开启web管理插件
cd /usr/local/rabbitmq/sbin
./rabbitmq-plugins list #查看插件列表
./rabbitmq-plugins enable rabbitmq_management #生效管理插件
#4.后台运行
./rabbitmq-server -detached #启动rabbitmq
./rabbitmqctl stop_app #停止命令，如果无法停止，使用kill -9 进程号进行关闭
#5.查看web管理界面
#http://端口号:15672 （放行端口，或关闭防火墙）
#默认用户：guest:guest，但虚拟机外的主机无法通过guest用户访问，需要新建用户


#-------------------------------------四、RabbitMq账户管理--------------------------
#1.创建账户
cd /usr/local/rabbitmq/sbin
./rabbitmqctl add_user mashibing mashibing
#2.给用户授予管理员角色
./rabbitmqctl set_user_tags mashibing administrator
#3.给用户授权
./rabbitmqctl set_permissions -p "/" mashibing ".*" ".*" ".*"
# “/” 表示虚拟机, mashibing 表示用户名, ".*" ".*" ".*" 表示完整权限
#4.登录
#url http://ip:15672/ mashibing:mashibing
```





## 2、基础例子

- 依赖

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-amqp</artifactId>
</dependency>
```

- 配置

  ```yaml
  spring:
    rabbitmq:
      host: 10.1.20.89
      username: mashibing
      password: mashibing
  ```

- provider

  ```java
  import org.springframework.amqp.core.*;
  import org.springframework.context.annotation.Bean;
  import org.springframework.context.annotation.Configuration;
  
  @Configuration
  public class RabbitmqConfig {
      @Bean
      protected Queue queue(){
          return new Queue("myQueue"); //构建队列myQueue
      }
  }
  ```

  ```java
  @SpringBootTest
  class ProviderApplicationTests {
      @Test
      void test1(){
          amqpTemplate.convertAndSend("myQueue", "这是内容"); //向队列myQueue中发消息
          System.out.println("发送成功");
      }
  }
  ```

-  consumer

   ```java
   @RabbitListener(queues = "myQueue") //接收队列myQueue的消息
   public void demo1(String msg){
       System.out.println("demo1获取到的内容："+msg); 
   }
   ```



## 3、交换器类型

### 3.1 交换器类型

交换器负责接收客户端传递过来的消息，并转发到对应的队列中。在RabbitMQ中支持四种交换器，第2节中的例子对应默认交换器

| 交换器   | 直连交换器（默认）         | 扇形交换器                 | 主题交换器                   | 首部交换器      |
| -------- | -------------------------- | -------------------------- | ---------------------------- | --------------- |
| Exchange | Direct Exchange            | Fanout Exchange            | Topic Exchange               | Header Exchange |
| 队列     | 一个交换器只能绑定一个队列 | 一个交换器可以绑定多个队列 | 一个交换器可以绑定多个队列   |                 |
| 消费者   | 多个消费者轮询消费         | 只有一个消费者能消费       | 只有一个消费者能消费         |                 |
| 注意     |                            |                            | *代表一个单词，#表示多个单词 |                 |

### 3.2 使用

#### 3.2.1 provider

```java
package com.msb.provider.config;

import org.springframework.amqp.core.*;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

@Configuration
public class RabbitmqConfig {

    /*
    直连交换器
    */
    @Bean
    protected Queue queue(){
        return new Queue("myQueue");
    }

    /*
    扇形交换器
    */
    @Bean
    public Queue queue1(){
        return new Queue("myfanout1");
    }
    @Bean
    public Queue queue2(){
        return new Queue("myfanout2");
    }
    @Bean
    public FanoutExchange getFanoutExchange(){
        return new FanoutExchange("amq.fanout");
    }
    @Bean
    public Binding binding(Queue queue1, FanoutExchange getFanoutExchange){
        return BindingBuilder.bind(queue1).to(getFanoutExchange);
    }
    @Bean
    public Binding binding2(Queue queue2, FanoutExchange getFanoutExchange){
        return BindingBuilder.bind(queue2).to(getFanoutExchange);
    }

    /*
    主题交换器
    */
    @Bean
    public Queue topicQueue1(){
        return new Queue("topicQueue1");
    }
    @Bean
    public Queue topicQueue2(){
        return new Queue("topicQueue2");
    }
    @Bean
    public TopicExchange topicExchange(){
        return new TopicExchange("amq.topic");
    }
    @Bean
    public Binding bindingTopic1(Queue topicQueue1, TopicExchange topicExchange){
        return BindingBuilder.bind(topicQueue1).to(topicExchange).with("com.msb.*");
        //*代表一个单词，#表示多个单词
    }
    @Bean
    public Binding bindingTopic2(Queue topicQueue2, TopicExchange topicExchange){
        return BindingBuilder.bind(topicQueue2).to(topicExchange).with("com.msb.a");
    }

}
```

#### 3.2.2 provier 测试

```java
package com.msb.provider;

import org.junit.jupiter.api.Test;
import org.springframework.amqp.core.AmqpTemplate;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.context.SpringBootTest;

@SpringBootTest
class ProviderApplicationTests {
    
    @Autowired
    private AmqpTemplate amqpTemplate;
    
    @Test
    void test1(){
        amqpTemplate.convertAndSend("myQueue", "这是内容");
        System.out.println("发送成功");
    }

    @Test
    void test2(){
        amqpTemplate.convertAndSend("amq.fanout", "core", "fanout类型的消息");
        System.out.println("发送成功");
    }

    @Test
    void test3(){
        amqpTemplate.convertAndSend("amq.topic", "com.msb.a", "topic类型的消息");
        System.out.println("发送成功");
    }
}
```



## 4、消息接收确认

| acknowledge-mode                  | none   | auto                  | manual                     |
| --------------------------------- | ------ | --------------------- | -------------------------- |
| 含义                              | 不确认 | 自动确认              | 手动确认                   |
| 使用                              | 无特殊 | 无特殊                | 代码需要确认消息（注2）    |
| 消费者异常时如何确认              | 不存在 | 不断消费此消息        | 服务每次重启都要消费此消息 |
| 消费者异常时阻塞不                | 不存在 | 不阻塞                | 不阻塞                     |
| 生命周期(异常-停止-重启异常)(注3) | 不存在 | Unacked-Ready-Unacked | Unacked-Ready-Unacked      |

注一：消息模式的

```yaml
spring:
    listener:
      simple:
        acknowledge-mode: auto #none/auto/manual
```

注二：确认消息代码

```java
@RabbitListener(queues = "topicQueue2")
public void demo7(String msg, Channel channel, @Header(AmqpHeaders.DELIVERY_TAG) long tag){
    try {
        channel.basicAck(tag, false);
    } catch (IOException e) {
        e.printStackTrace();
    }
}
```

注三：异常时的生命周期

![rabbitmq队列](mq.assets/rabbitmq队列.png)

注四：自动确认和手动确认

自动确认和手动确认区别在于碰到异常时自动确认会不断消费此消息，我怀疑是因为，自动确认会把消息重新放回到Ready中，手动确认指挥因为消费者端停止消息回到Ready。但是，从rabbitmq中看不到任何不同，自动确认消息的状态不会在Ready和Unacked中来回切换。









system

1、权限模块：平台端角色和用户可以正常增删该查，并且新增的用户可以正常访问。权限不会多或少。（重要）

2、装修模块：装修可以正常增删改查，移动端可以正常使用装修界面。（重要）

3、设置模块：平台端的系统设置可以正常展示使用。

4、原因模块：原理管理可以正常增删改查，并且移动端退货时可以正常选择原因。

5、结算模块：平台端结算账单可以正常结算，正常展示。

































