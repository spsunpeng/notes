# 一、RPC

## 1、RPC协议

  RFC(Request For Comments) 是由互联网工程任务组(IETF)发布的文件集。文件集中每个文件都有自己唯一编号，例如：rfc1831。目前RFC文件由互联网协会(Internet Society，ISOC)赞助发型。

RPC在rfc 1831中收录 ，RPC（Remote Procedure Call） 远程过程调用协议

​	RPC协议规定允许互联网中一台主机程序调用另一台主机程序，而程序员无需对这个交互过程进行编程。在RPC协议中强调当A程序调用B程序中功能或方法时，A是不知道B中方法具体实现的。

​	RPC是上层协议，底层可以基于TCP协议，也可以基于HTTP协议。一般我们说RPC都是基于RPC的具体实现，如：Dubbo框架。从广义上讲只要是满足网络中进行通讯调用都统称为RPC，甚至HTTP协议都可以说是RPC的具体实现，但是具体分析看来RPC协议要比HTTP协议更加高效，基于RPC的框架功能更多。

​	RPC协议是基于分布式架构而出现的，所以RPC在分布式项目中有着得天独厚的优势。

#### 1.1 RPC和HTTP对比

- 具体实现

  RPC：可以基于TCP协议，也可以基于HTTP协议。

  HTTP：基于HTTP协议

- 效率

  RPC：自定义具体实现可以减少很多无用的报文内容，使得报文体积更小。

  HTTP：如果是HTTP 1.1 报文中很多内容都是无用的。如果是HTTP2.0以后和RPC相差不大，比RPC少的可能就是一些服务治理等功能。

- 连接方式

  RPC：长连接支持。

  HTTP：每次连接都是3次握手。

- 性能

  RPC可以基于很多序列化方式。如：thrift

  HTTP 主要是通过JSON，序列化和反序列效率更低。

- ##### 注册中心

  RPC ：一般RPC框架都带有注册中心。

  HTTP：都是直连。

- ##### 负载均衡

  RPC：绝大多数RPC框架都带有负载均衡测量。

  HTTP：一般都需要借助第三方工具。如：nginx

- ##### 综合结论

  ​	RPC框架一般都带有丰富的服务治理等功能，更适合企业内部接口调用。而HTTP更适合多平台之间相互调用。





## 2、HttpClient

  在JDK中java.net包下提供了用户HTTP访问的基本功能，但是它缺少灵活性或许多应用所需要的功能。

  HttpClient起初是Apache Jakarta Common 的子项目。用来提供高效的、最新的、功能丰富的支持 HTTP 协议的客户端编程工具包，并且它支持 HTTP 协议最新的版本。2007年成为顶级项目。

### 2.1 代码

#### 2.1.1 client

```xml
<dependency>
    <groupId>org.apache.httpcomponents</groupId>
    <artifactId>httpclient</artifactId>
    <version>4.5.10</version>
</dependency>
```

```java
public class Demo {
    @Test
    public void demo1() throws Exception{
        //浏览器
        CloseableHttpClient httpClient = HttpClients.createDefault();
        //url
        URIBuilder uriBuilder = new URIBuilder("http://127.0.0.1:8080/demoGet");
        //参数
        uriBuilder.addParameter("string", "abc");
        //执行方法
        HttpGet get = new HttpGet(uriBuilder.build());
        //执行
        CloseableHttpResponse response = httpClient.execute(get);
        //响应体
        String result = EntityUtils.toString(response.getEntity(), "utf-8");
        System.out.println(result);
    }

    @Test
    public void demoPost() throws Exception{
        CloseableHttpClient httpClient = HttpClients.createDefault();
        //url
        URIBuilder uriBuilder = new URIBuilder("http://127.0.0.1:8080/demoPost");
        //发法
        HttpPost httpPost = new HttpPost(uriBuilder.build());
        //参数：请求体
        List<NameValuePair> params = new ArrayList<>();
        params.add(new BasicNameValuePair("name", "sunpeng"));
        params.add(new BasicNameValuePair("id", "1"));
        HttpEntity httpEntity = new UrlEncodedFormEntity(params, "utf-8");
        httpPost.setEntity(httpEntity);
        //执行
        CloseableHttpResponse response = httpClient.execute(httpPost);
        //响应体
        String result = EntityUtils.toString(response.getEntity(), "utf-8");
        System.out.println(result);
        //json
        ObjectMapper mapper = new ObjectMapper();
        Person person = mapper.readValue(result, Person.class);
        System.out.println(person);
    }

    @Test
    public void demoJson() throws  Exception{

        //浏览器
        CloseableHttpClient httpClient = HttpClients.createDefault();
        //url、方法
        HttpPost post = new HttpPost("http://localhost:8080/json");
        //请求体
        Person person = new Person(12L, "jia");
        ObjectMapper mapper = new ObjectMapper();
        String json = mapper.writeValueAsString(person);
        StringEntity entity = new StringEntity(json, ContentType.APPLICATION_JSON);
        post.setEntity(entity);
        //执行
        CloseableHttpResponse response = httpClient.execute(post);
        //响应
        String result = EntityUtils.toString(response.getEntity());
        System.out.println(result);
        //关闭连接
        response.close();
        httpClient.close();

    }
}
```

#### 2.1.2 server

```java
@RestController
public class HelloController {

    @RequestMapping("demoGet")
    public String demoGet(String string){
        System.out.println(string);
        return "success: "+string;
    }

    @RequestMapping("demoPost")
    public Person demoPost(Person person){
        System.out.println(person);
        return person;
    }

    @RequestMapping(value = "json")
    public Person demoJson(@RequestBody Person person){
        System.out.println(person);
        return person;
    }
}
```

@RequestBody 服务端接收json格式参数，且是必传



## 3、RMI

RMI(Remote Method Invocation) 远程方法调用。

​	RMI是从JDK1.2推出的功能，它可以实现在一个Java应用中可以像调用本地方法一样调用另一个服务器中Java应用（JVM）中的内容。

​	RMI 是Java语言的远程调用，无法实现跨语言。

### 3.1 代码

#### 3.1.1 server

```java
public interface DemoService extends Remote {
    String demo1(String str) throws RemoteException;
}

public class DemoServiceImpl extends UnicastRemoteObject implements DemoService {
    //不加，idea提示未处理异常，可能是默认构造器不能处理异常
    public DemoServiceImpl() throws RemoteException {
        super();
    }

    @Override
    public String demo1(String str) throws RemoteException {
        System.out.println(str);
        return str+"123";
    }
}

public static void main(String[] args) throws Exception{
        //创建接口实例
        DemoService demoService = new DemoServiceImpl();
        //创建注册表
        LocateRegistry.createRegistry(8081);
        //注册服务
        Naming.bind("rmi://127.0.0.1:8081/demoService", demoService);
        System.out.println("服务器启动成功");
    }
```

#### 3.1.2 client

```java
public class ClientMain {
    public static void main(String[] args) throws Exception{
        DemoService demoService = (DemoService)Naming.lookup("rmi://127.0.0.1:8081/demoService");
        String response = demoService.demo1("hello");
        System.out.println(response);
    }
}
```



## 4、zookeeper

zookeeper分布式管理软件。常用它做注册中心（依赖zookeeper的发布/订阅功能）、配置文件中心、分布式锁配置、集群管理等。

### 4.1 安装使用

#### 4.1.1 下载

```sh
#安装
#下载zookeeper-3.5.5：http://zookeeper.apache.org
cd /usr/local/tmp
tar -zxvf apache-zookeeper-3.5.5-bin.tar.gz
mv apache-zookeeper-3.5.9-bin ../zookeeper

#配置
#创建数据保存目录
cd ..
mkdir data
cd conf
#创建配置文件zoo.cfg，zoo_sample.cfg仅是zookeeper的示例文件
cp zoo_sample.cfg zoo.cfg
#修改配置文件中数据保存目录
vi zoo.cfg #dataDir=/usr/local/zookeeper/data

#启动
cd ../bin
./zkServer.sh start  #启动
./zkServer.sh status #查看状态，结果中standalone表示单机版
```

##### 4.1.2 使用

```sh
#进入客户端
./zkCli.sh
#查看目录
ls [-s][-R] /path #-s 详细信息,-R 当前目录和子目录中内容都罗列出来
#创建目录
create /path [data]
#删除目录
delete /path
#查看数据
get -s /path #-s详细信息
#查看数据
set /path data
```

##### 4.1.3 查看数据

```sh
[zk: localhost:2181(CONNECTED) 16] get -s /demo
null            #存放的数据
cZxid = 0x5
ctime = Tue Jul 20 06:46:17 EDT 2021
mZxid = 0x5
mtime = Tue Jul 20 06:46:17 EDT 2021
pZxid = 0x5          #子节点的zxid
cversion = 0         #子节点更新次数
dataVersion = 0      #节点数据更新次数
aclVersion = 0       #节点ACL(授权信息)的更新次数
ephemeralOwner = 0x0 #如果该节点为ephemeral节点(临时，生命周期与session一样), ephemeralOwner值表示与该节点绑定的session id. 如果该节点不是ephemeral节点, ephemeralOwner值为0.
dataLength = 0       #节点数据字节数
numChildren = 0      #子节点数量
```



### 4.2 代码

```xml
<dependency>
    <groupId>org.apache.zookeeper</groupId>
    <artifactId>zookeeper</artifactId>
    <version>3.5.5</version>
</dependency>
```

#### 4.2.1 发送

```java
public static void main(String[] args) throws Exception {
    //zookeeper对象：ip地址+端口号、超时时间、当连接成功后编写连接信息
    ZooKeeper zooKeeper = new ZooKeeper("10.1.20.89:2181", 10000, new Watcher() {
        public void process(WatchedEvent watchedEvent) {
            System.out.println("获取连接");
        }
    });
    //发送内容：路径、内容、权限、内容的模式（临时永久、是否允许重复）
    String content = zooKeeper.create("/demo/address", "xian".getBytes(), ZooDefs.Ids.OPEN_ACL_UNSAFE, CreateMode.PERSISTENT_SEQUENTIAL);
    System.out.println(content);
}
```

#### 4.2.2 接收

```java
public static void main(String[] args) {
    try {
        ZooKeeper zookeeper = new ZooKeeper("10.1.20.89:2181", 10000, new Watcher() {
            @Override
            public void process(WatchedEvent watchedEvent) {
                System.out.println("获取连接");
            }
        });
        //获取列表
        List<String> list = zookeeper.getChildren("/demo", false);
        for (String child : list) {
            byte[] result = zookeeper.getData("/demo/" + child, false, null);
            System.out.println(new String(result));
        }
    } catch (IOException e) {
        e.printStackTrace();
    } catch (KeeperException e) {
        e.printStackTrace();
    } catch (InterruptedException e) {
        e.printStackTrace();
    }
}

```



## 5、Dubbo

父项目

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>com.msb</groupId>
    <artifactId>DubboDemo</artifactId>
    <packaging>pom</packaging>
    <version>1.0-SNAPSHOT</version>
    <modules>
        <module>api</module>
        <module>provider</module>
        <module>consumer</module>
    </modules>

    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>2.1.10.RELEASE</version>
    </parent>

    <dependencyManagement>
        <dependencies>
            <dependency>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-starter</artifactId>
                <version>2.1.10.RELEASE</version>
            </dependency>
            <dependency>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-starter-web</artifactId>
                <version>2.1.10.RELEASE</version>
            </dependency>
            <dependency>
                <groupId>org.apache.dubbo</groupId>
                <artifactId>dubbo-spring-boot-starter</artifactId>
                <version>2.7.3</version>
            </dependency>
            <dependency>
                <groupId>org.apache.curator</groupId>
                <artifactId>curator-recipes</artifactId>
                <version>4.2.0</version>
            </dependency>
            <dependency>
                <groupId>org.apache.curator</groupId>
                <artifactId>curator-framework</artifactId>
                <version>4.2.0</version>
            </dependency>
        </dependencies>
    </dependencyManagement>

</project>
```

#### 5.1 controller 定义接口

```java
public interface DubboDemoService {
    String demo(String param);
}
```

#### 5.2 provider 提供者

依赖：略，从dependencyManagemen父项目中挑选自己需要的依赖

配置：

```yaml
dubbo:
  application:
    name: dubbo-provider
  registry:
    address: zookeeper://10.1.20.89:2181
  protocol:
    port: 20884 #一个微服务部署多个的时候需要
```

实现：

```java
import com.msb.dubbo.service.DubboDemoService;
import org.apache.dubbo.config.annotation.Service;
@Service(weight = 2) //注意注解是org.apache.dubbo的
public class DubboDemoServiceImpl implements DubboDemoService {
    @Override
    public String demo(String param) {
        System.out.println("server: "+param);
        return "helllo: "+param;
    }
}
```
#### 5.3 consumer 消费者

依赖：略，从dependencyManagemen父项目中挑选自己需要的依赖

配置：

```yaml
dubbo:
  application:
    name: dubbo-consumer
  registry:
    address: zookeeper://10.1.20.89:2181
```

实现：

```java
import com.msb.dubbo.service.DubboDemoService;
import org.apache.dubbo.config.annotation.Reference;
import org.springframework.stereotype.Service;

@Service
public class DemoService {
    @Reference(loadbalance = "roundrobin") //注意注解是org.apache.dubbo的
    private DubboDemoService dubboDemoService;
    public String demo(){
        return dubboDemoService.demo("123");
    }
}
```

#### 5.4 界面

- 执行jar包

  ```sh
  java -jar dubbo-admin-0.2.0.jar
  ```

- 访问

  127.0.0.1:8080就可以访问了。













