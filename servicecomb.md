## 模板 servicecombtest3

- servicecombtest3
  + servicecomb_consumer
  + servicecomb_provider
  + pom.xml

### 1. pom.xml(servicecombtest3)

```xml
<modules>
    <module>servicecomb_provider</module>
    <module>servicecomb_consumer</module>
</modules>

<parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-parent</artifactId>
    <version>1.5.12.RELEASE</version>
    <relativePath/> <!-- lookup parent from repository -->
</parent>

<dependencyManagement>
    <dependencies>
        <dependency>
            <groupId>org.apache.servicecomb</groupId>
            <artifactId>java-chassis-dependencies</artifactId>
            <version>1.0.0-m2</version>
            <type>pom</type>
            <scope>import</scope>
        </dependency>
    </dependencies>
</dependencyManagement>

<build>
    <plugins>
        <plugin>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-maven-plugin</artifactId>
        </plugin>
    </plugins>
</build>
```



### 2.servicecomb_provider

- servicecomb_provider
  + src.main.java
    * com.heima.controller.**ProviderController**
    * com.heima.**Application**
  + src.main.resources
    * **microservice.yaml**
  + **pom.xml**

#### 2.1 pom.xml(servicecomb_provider)

```xml
<dependencies>
    <dependency>
        <groupId>org.hibernate</groupId>
        <artifactId>hibernate-validator</artifactId>
    </dependency>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter</artifactId>
    </dependency>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
    </dependency>
    <dependency>
        <groupId>org.apache.servicecomb</groupId>
        <artifactId>spring-boot-starter-provider</artifactId>
    </dependency>
</dependencies>
```

#### 2.2 microservice.yaml

```yaml
APPLICATION_ID: start.servicecomb.io
service_description:
  name: providerTest3
  version: 1.0.0
servicecomb:
  service:
    registry:
      address: http://127.0.0.1:30100
  rest:
    address: 0.0.0.0:8091
```

#### 2.3 Application

```java
@SpringBootApplication
@EnableServiceComb
public class Application {
    public static void main(String[] args) {
        SpringApplication.run(Application.class, args);
    }
}
```

#### 2.4 ProviderController

```java
@RestSchema(schemaId = "ProviderController")
@RequestMapping("/provider")
public class ProviderController {

    @GetMapping("/hello")
    public String sayRest(String name) {
        return "Hello World "+name;
    }
}
```



### 3.servicecomb_consumer

- servicecomb_consumer
  + src.main.java
    * com.heima.controller.**ConsumerController**
    * com.heima.service.**ConsumerService**
    * com.heima.**Application**
  + src.main.resources
    * **microservice.yaml**
  + **pom.xml**

#### 3.1 pom.xml(servicecomb_consumer)

```xml
<dependencies>
    <dependency>
        <groupId>org.hibernate</groupId>
        <artifactId>hibernate-validator</artifactId>
    </dependency>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter</artifactId>
    </dependency>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
    </dependency>
    <dependency>
        <groupId>org.apache.servicecomb</groupId>
        <artifactId>spring-boot-starter-provider</artifactId>
    </dependency>
</dependencies>
```

#### 3.2 microservice.yaml

```yaml
APPLICATION_ID: start.servicecomb.io
service_description:
  name: consumerTest3
  version: 1.0.0
servicecomb:
  service:
    registry:
      address: http://127.0.0.1:30100
  rest:
    address: 0.0.0.0:8092
```

#### 3.3 Application

```java
@SpringBootApplication
@EnableServiceComb
public class Application {
    public static void main(String[] args) {
        SpringApplication.run(Application.class, args);
    }
}
```

#### 3.4 ConsumerController

```java
@RestSchema(schemaId = "ConsumerController")
@RequestMapping("/consumer")
public class ConsumerController {

    @Autowired
    private ConsumerService consumerService;

    @GetMapping("/hello")
    public String sayRest(String name){
        return consumerService.sayRest(name);
    }
}
```

#### 3.5 ConsumerService

```java
@Service
public class ConsumerService {

    private final RestTemplate restTemplate = RestTemplateBuilder.create();

    public String sayRest(String name){
        String rtnValue = restTemplate.getForObject("cse://"+"providerTest3"+"/provider/hello?name="+name, String.class);
        return rtnValue;
    }
}
```



### 4.总结

#### 4.1 注册

- microservice.yaml
  + service_description.name=providerTest3： 注册到服务中心的名字

- Application
  + @EnableServiceComb：表示ServiceComb服务

- ProviderController
  - @RestSchema(schemaId = "ProviderController")：1是服务中心服务名下的下一级目录，2是代替RestController

#### 4.2 调用

- 调用者：调用者要在服务注册中心调用其他服务，所以他也是微服务，也需要注册
- 调用：ConsumerService
  + private final RestTemplate restTemplate = RestTemplateBuilder.create();
  + String rtnValue = restTemplate.getForObject("cse://"+"providerTest3"+"/provider/hello?name="+name, String.class);











