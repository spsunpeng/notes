# 参考资料

马东阳：第12阶段-springboot的使用，第38节-拓展_@ImportResource

连鹏举：互联网高级工程师就业班



# 一、环境配置



## 1、idea创建项目

可以使用maven或者gradle，不过idea为我们提供了更简单的创建方式Spring Initializr

- 新建 -》 project -》Spring Initializr
- New Project 下选择：
  - type：Maven Project
  - packaging：jar（自动打包的打包方式）
  - java Version：8
  - package：包

- 选择需要的依赖



## 2、依赖

### 2.1 springboot-parent

父项目一般用于依赖的版本管理

```xml
<parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-parent</artifactId>
    <version>2.5.2</version>
    <relativePath/> <!-- lookup parent from repository -->
</parent>
```

spring-boot-starter-parent 中包括 spring-boot-dependencies 和 一些其他配置

#### 2.1.1 spring-boot-dependencies 

```xml
他的父项目是
<parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-dependencies</artifactId>
    <version>2.5.2</version>
</parent>
他来真正管理Spring Boot应用里面的所有依赖版本；
```

spring-boot-dependencies管理依赖的版本，所以它又叫版本仲裁

#### 2.1.2 其他

spring-boot-starter-parent 的配置中，会默认读取配置文件 application.yaml/yml/properties

```xml
<build>
  <resources>
    <resource>
      <directory>${basedir}/src/main/resources</directory>
      <filtering>true</filtering>
      <includes>
        <include>**/application*.yml</include>
        <include>**/application*.yaml</include>
        <include>**/application*.properties</include>
      </includes>
    </resource>
    <resource>
      <directory>${basedir}/src/main/resources</directory>
      <excludes>
        <exclude>**/application*.yml</exclude>
        <exclude>**/application*.yaml</exclude>
        <exclude>**/application*.properties</exclude>
      </excludes>
    </resource>
  </resources>
</build>
```



### 2.2 启动类 start

启动器就是把相关的依赖组合在一起，由springboot官方或者第三方提供

```xml
<!--名字在后，spring提供的启动器，spring整合第三方-->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
</dependency>
<!--名字在前，第三方提供的启动器，第三方支持spring-->
<dependency>
    <groupId>org.mybatis.spring.boot</groupId>
    <artifactId>mybatis-spring-boot-starter</artifactId>
    <version>2.1.3</version>
</dependency>
```



### 2.3 插件

#### 2.3.1 打包工具

依赖中可以定义打包工具

```xml
<!--默认打jar包-->
<packaging>jar</packaging>
<!--打包插件-->
<build>
    <plugins>
        <plugin>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-maven-plugin</artifactId>
            <configuration>
                <fork>true</fork>
            </configuration>
        </plugin>
    </plugins>
</build>
```

springboot打包可以分为jar、war、pom，其中pom用在父依赖中

#### 3.3.2 打包

![maven打包](springboot.assets/maven打包.png)

#### 3.3.3 打包结果

![spring-booot-maven-plugin](springboot.assets/spring-booot-maven-plugin.png)

如果没有spring-boot-maven-plugin组件依赖，那么打包时就没有下面红框中的文件，在cmd中执行时（java -jar），会报错：没有主文件。

#### 3.3.4 执行

受3.3.3的打包结果影响，可能不成功

```sh
#cmd
java -jar name.jar
#idea-Terminal
E:\IDEAProject\spring-boot-02\target>java -jar spring-boot-02-0.0.1-SNAPSHOT.jar --spring.config.location=C:\Users\sunpeng.SINO\Desktop\application.properties
```

#### 3.3.5 war

由于springboot集成了tomcat，所以运行web程序时用jar包即可。如果非要打war包，按照如下步骤：

- 依赖

  需要排除web启动中自动依赖的tomcat插，再加入单独的tomcat依赖（仅在编译时生效）。

  ```xml
  <!--war包-->
  <packaging>war</packaging>
  
  <!--配置SpringBoot的web启动器-->
  <dependency>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-starter-web</artifactId>
      <!--排除web启动中自动依赖的tomcat插件-->
      <exclusions>
          <exclusion>
              <groupId>org.springframework.boot</groupId>
              <artifactId>spring-boot-starter-tomcat</artifactId>
          </exclusion>
      </exclusions>
  </dependency>
  <!--
      手动依赖tomcat插件，但是表明项目打包时该依赖不会被打进去，目的主要是保证开发阶段本地SpringBoot
      项目可以正常运行
  -->
  <dependency>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-starter-tomcat</artifactId>
      <!--打包的时候可以不用包进去，别的设施会提供。事实上该依赖理论上可以参与编译，测试，运行等周期。
          相当于compile，但是打包阶段做了exclude操作-->
      <scope>provided</scope>
  </dependency>
  
  <!--打包插件一样-->
  <build>
      <plugins>
          <plugin>
              <groupId>org.springframework.boot</groupId>
              <artifactId>spring-boot-maven-plugin</artifactId>
              <configuration>
                  <fork>true</fork>
              </configuration>
          </plugin>
      </plugins>
  </build>
  ```

- 启动类

  正常springboot启动要初始化web的配置，现在没有了tomcat依赖，正常springboot无法启动成功。

  ```java
  @SpringBootApplication
  public class MyApplication extends SpringBootServletInitializer {
      //重写配置方法
      @Override
      protected SpringApplicationBuilder configure(SpringApplicationBuilder application) {
          return application.sources(MyApplication.class);
      }
      public static void main(String[] args) {
          //启动SpringBoot
          SpringApplication.run(MyApplication.class,args);
      }
  }
  ```




## 3、启动类

```java
/*@SpringBootConfiguration
@EnableAutoConfiguration
@ComponentScan("com.msb")*/
@SpringBootApplication
public class SpringbootBaseApplication {
    public static void main(String[] args) {
        ConfigurableApplicationContext context = SpringApplication.run(SpringbootBaseApplication.class, args);
/*        String[] beanDefinitionNames = context.getBeanDefinitionNames();
        for (int i = 0; i < beanDefinitionNames.length; i++) {
            System.out.println(beanDefinitionNames[i]);
        }*/
        Person bean = context.getBean(Person.class);
        System.out.println(bean);
    }
}
```

### 3.1 @SpringBootApplication

@SpringBootApplication = @SpringBootConfiguration + @EnableAutoConfiguration + @ComponentScan("com.msb")

### 3.2 获取容器

```java
//springboot
ConfigurableApplicationContext context = SpringApplication.run(SpringbootBaseApplication.class, args);
//spring
ApplicationContext context = new ClassPathXmlApplicationContext("ioc.xml");
```

springboot和spring一样，在初始化时加载容器，扫描包构建bean。spring根据开发者的xml文件构建bean，而springboot会加载其他组件的配置类，大大简化了程序的开发。



## 4、配置

|          | spring                              | springboot                               |
| -------- | ----------------------------------- | ---------------------------------------- |
| 格式     | xml、properties                     | xml、properties、yaml                    |
| 外部文件 | context:property-placeholder(备注1) | @PropertySource（备注2）                 |
| 默认     | 无，获取容器时需要指定配置文件      | application.yaml、application.properties |
| 注意     | 无                                  | **springboot中的spring注解都不支持yaml** |

- 备注1：spring配置文件中引入其他配置文件

```xml
<context:property-placeholder location="classpath:userConfig.properties"></context:property-placeholder>
```

- 备注2：  springboot中引入非默认配置文件

```java
@PropertySource(value = "classpath:bookConfig.properties")
```



### 4.1 application配置文件

#### 4.1.1 优先级

- 本地：/config/*  大于  /* 大于  /classfile/config/*  大于  /classfile/*

- 运维：命令行(参数)  >  命令行(外部配置文件)  > 本地
  - 参数：  --server.port=8087
  - 外部配置文件：--spring.config.location=C:\Users\sunpeng.SINO\Desktop\application.properties

- properties  >   yaml  优先而非覆盖

#### 4.1.2 环境配置

公司中往往有多个环境，比如dev、sit、uat，且各个环境的配置不一样。这时在application-dev.yml、application-sit.yml存入个性化配置，在application.yml中配置使用那个配置文件。

```yaml
spring:
  profiles:
    active: sit
```



### 4.2 yaml

#### 4.2.1 格式

- 根据缩进判断等级
- 连字符 - 代表数组

#### 4.2.2 注意

每行前缩进的空格是yaml文件的格式/语法，一定注意。

##### 4.2.2.1 复制粘贴不规范

在复制粘贴keycloak文件时，由于复制粘贴没有考虑空格，导致只有第一行空格正确，其余几行都错误。

![错误复制yaml内容方法](springboot.assets/错误复制yaml内容方法.png)

正确做法为从顶头位置复制，从顶头位置复制/粘贴。

![正确复制yaml内容方法](springboot.assets/正确复制yaml内容方法.png)

##### 4.2.2.2 修改文件类型

由于servicecomb契约文件的原因，需要先将契约文件改为txt文件，程序启动成功后，再改为yaml文件格式，并将打印出来的文件粘贴到契约文件。注意，一定要先改文件类型，再改内容。反之，空格会乱。

### 

# 二、bean

springboot可以认为是spring的简化配置版，springboot所使用的注解还是沿用spring的。不同点是springboot在启动时自动装配了一些配置。

## 1、启动

@SpringBootApplication注解在启动时会自动扫描包。而spring需要xml配置自动扫描包，并在启动容器启动时传入配置文件。实际@SpringBootApplication的扫包是由@ComponentScan完成，spring也可以使用此注解

```java
//包扫描
@ComponentScan(basePackages = "com.msb")
//通过注解获取容器
@Test
public void testTransaction3(){
    ApplicationContext context =new AnnotationConfigApplicationContext(SpringConfig.class);
}
```



## 2、IOC 构建bean

### 2.1 构建bean

```java
@Configuration(proxyBeanMethods=false)
//@Component
//@Import(value = Person.class) //id="com.msb.config.Person" , type=Person.class 可以放到任意位置
//@EnableConfigurationProperties(value = Person.class) //person-com.msb.config.Person
//@ImportResource("classpath:beans.xml")
public class MyConfig {
    @Bean
    public Person getPerson(){
        return new Person();
    }
}
```

- @Component(value="bean_id")

  默认bean_id=类的首字母小写，参数value可以修改

- @Configuration(proxyBeanMethods=false)

  @Configuration = @Component + proxyBeanMethods

  false 称之为Lite模式  特点启动快；true 称之为Full模式  特点依赖spring容器控制bean单例

  ```java
  @Target({ElementType.TYPE})
  @Retention(RetentionPolicy.RUNTIME)
  @Documented
  @Component
  public @interface Configuration {
      @AliasFor(
          annotation = Component.class
      )
      String value() default "";
  
      boolean proxyBeanMethods() default true;
  }
  ```

- @Bean(value="bean_id") + @Configuration/@Component

  - 这种bean的创建方法就如同工厂模式

  - 默认bean_id=方法名，参数value可以修改

- @Import(value = Person.class)

  bean_id = 全路径名，比如："com.msb.config.Person"

- @ImportResource("classpath:beans.xml") + beans.xml

  ```xml
  <?xml version="1.0" encoding="UTF-8"?>
  <beans xmlns="http://www.springframework.org/schema/beans"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">
      <bean id="person" class="com.msb.config.Person">
          <property name="name" value="jia"></property>
      </bean>
  </beans>
  ```

  bean_id = xml文件中bean标签的id

- @EnableConfigurationProperties(value = Person.class) + @ConfigurationProperties(prefix = "person") + application.yaml

  ```java
  @ConfigurationProperties(prefix = "person")
  public class Person {
      private String name;
      public String getName() {
          return name;
      }
      public void setName(String name) {
          this.name = name;
      }
  }
  ```

  ```yaml
  person:
    name: sunpeng
  ```

  它的主要用途是这只参数，而非构建bean

  bean_id = 类名首字母小写 + 全路径名，比如：person-com.msb.config.Person

### 2.2 参数

```java
@Component
@PropertySource(value = "classpath:bookConfig.properties")
@ConfigurationProperties(prefix = "springboot-base.book")
public class Book {
    //@Value("${springboot-base.book.name}")
    private String name;
    public String getName() {
        return name;
    }
    public void setName(String name) {
        this.name = name;
    }
}
```

```yaml
springboot-base.book.name=shuixu
```

- springboot在spring的基础上，还提供了注解：@ConfigurationProperties
- springboot在spring的基础上，还支持yaml文件。
- spring没有默认指定文件，必须指定，springboot默认了指定文件application
- @PropertySource是sping提供，不支持yaml格式文件，特例是application.yaml，可能是springboot做了转换
- 对于springboot而言，配置文件有默认application.yaml的和指定的bookConfig.properties。如果两者做了，相同的配置。@Value会使用application.yaml，@ConfigurationProperties会使用bookConfig.properties。
- @ConfigurationProperties不支持大写字母，支持连字符 - 
- @Component可以替换为@EnableConfigurationProperties(value = Person.class) ，这样这个注解可以放到任意类上。

## 3、DI 依赖注入

```java
@Configuration(proxyBeanMethods = false)
public class DIConfig {
    @Bean
    public Address getAddress(){
        return new Address();
    }
    @Bean(value = "student")
    public Student getStudent(Address address){ //通过bean_type获取，加注解也可以通过id获取
        return new Student(address);
    }
}
```

```java
public class DIService {
    
    @Autowired
    @Qualifier(value = "getAddress") //通过bean_id获取
    private Address address;
    
    @Autowired //通过bean_type获取
    private Student student;
}
```



## 4、@Conditional 条件装配

满足Conditional指定的条件,则进行组件注入

```java
@Configuration(proxyBeanMethods=false)
@ConditionalOnClass(value = Person.class)
public class MyConfig {
    @Bean
    @ConditionalOnProperty(name = "person.name", havingValue = "sunpeng")
    public Person getPerson(){
        return new Person();
    }
}
```

查看Conditional的子类（查看子类快捷键 ctrl + h）

![Conditional](springboot.assets/Conditional.png)



## 5、bean相关注解总结

| 注解                                                 | 解释                             | 提供商     |
| ---------------------------------------------------- | -------------------------------- | ---------- |
| Bean对象                                             |                                  |            |
| @Component                                           | id=类名首字母小写                | spring     |
| @Configuration(proxyBeanMethods=false)               | id=类名首字母小写，标识配置类    | spring     |
| @Bean                                                | id=方法名                        | spring     |
| @Import(value = Person.class)                        | id=全限定名                      | spring     |
| @ImportResource("classpath:beans.xml")               | id=xml中配置                     | spring     |
|                                                      |                                  |            |
| Bean参数，需要@Component/@Configuration配合          |                                  | spring     |
| @Value(value="")                                     | 配置参数                         | spring     |
| @ConfigurationProperties(prefix = "person")          | 配置参数：key是前缀+字段         | spring     |
| @PropertySource("classpath:preson.properties")       | 修改默认位置文件                 | springboot |
| @EnableConfigurationProperties(value = Person.class) | 替代@Component，可在配置类上配置 | spring     |
|                                                      |                                  |            |
| DI                                                   |                                  |            |
| @Autowird                                            | 根据类型注入                     | spring     |
| @Qualifier(value = "bean_id")                        | 根据id注入                       | spring     |
| @Resource                                            | 两种都支持，但不用               | java       |
|                                                      |                                  |            |
| 其他                                                 |                                  |            |
| @Conditional                                         | 条件                             | spring     |
| @ConditionalOnClass(value = Person.class)            | 是否有Person.class               | springboot |
|                                                      |                                  |            |

## 6、bean_接口

见本文四-1

## 7、bean_泛型

见spring-三-4



# 三、web

## 1、整合servlet

应该也可以认为时整合springmvc吧

### 1.1 http

```java
@WebServlet(name = "myServlet",urlPatterns = "/servlet")
public class HelloServlet extends HttpServlet {

    @Override
    protected void doGet(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        System.out.println("123");
        super.doGet(req, resp);
    }
}
```

```java
@ServletComponentScan
```

### 1.2 过滤器

```java
@WebFilter(filterName = "MyFilter", urlPatterns = "/*")
public class MyFilter implements Filter {
    @Override
    public void init(FilterConfig filterConfig) throws ServletException {
        System.out.println("init");
    }
    @Override
    public void doFilter(ServletRequest servletRequest, ServletResponse servletResponse, FilterChain filterChain) throws IOException, ServletException {
        System.out.println("doFilter");
        filterChain.doFilter(servletRequest, servletResponse);
    }
    @Override
    public void destroy() {
        System.out.println("destroy");
    }
}
```

```java
    @Bean
    public ServletRegistrationBean<HelloServlet> getServletRegistrationBean(){
        ServletRegistrationBean<HelloServlet> bean = new ServletRegistrationBean<>(new HelloServlet());
        ArrayList<String> url = new ArrayList<>();
        url.add("/helloServlet");
        bean.setUrlMappings(url);
        bean.setLoadOnStartup(1);
        return bean;
    }
```

### 1.3 监听器

```java
public class MyListen implements HttpSessionListener {
    public static int online=0;
    @Override
    public void sessionCreated(HttpSessionEvent se) {
        online++;
        System.out.println("创建+1");
    }
    @Override
    public void sessionDestroyed(HttpSessionEvent se) {
        System.out.println("销毁-1");
    }
}
```

```java
@Bean
public ServletListenerRegistrationBean listenerRegist(){
    ServletListenerRegistrationBean srb = new ServletListenerRegistrationBean();
    srb.setListener(new MyListen());
    System.out.println("listener");
    return srb;
}
```

## 2、整合springmvc

### 2.1 拦截器

和spring一样

```java
@Component
public class WebInterceptor implements HandlerInterceptor {
    @Override
    public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler) throws Exception {
        System.out.println("拦截器");
        return true;
    }
}
```

将spring的xml配置改为springboot的配置类

```java
@Configuration
public class InterceptorConfig implements WebMvcConfigurer {
    @Autowired
    private WebInterceptor webInterceptor;
    @Override
    public void addInterceptors(InterceptorRegistry registry) {       registry.addInterceptor(webInterceptor).addPathPatterns("/**").excludePathPatterns("/hi");
    }
}
```

## 3、整合前端

### 3.1 jsp

优点：
1、功能强大，可以写java代码
2、支持jsp标签（jsp tag）
3、支持表达式语言（el）
4、官方标准，用户群广，丰富的第三方jsp标签库
缺点：
性能问题。不支持前后端分离

### 3.2 freemarker

FreeMarker是一个用Java语言编写的模板引擎，它基于模板来生成文本输出。FreeMarker与Web容器无关，即在Web运行时，它并不知道Servlet或HTTP。它不仅可以用作表现层的实现技术，而且还可以用于生成XML，JSP或Java 等。
目前企业中:主要用Freemarker做静态页面或是页面展示

优点：
1、不能编写java代码，可以实现严格的mvc分离
2、性能非常不错
3、对jsp标签支持良好
4、内置大量常用功能，使用非常方便
5、宏定义（类似jsp标签）非常方便
6、使用表达式语言
缺点：
1、不是官方标准
2、用户群体和第三方标签库没有jsp多

### 3.3 Thymeleaf

Thymeleaf是个XML/XHTML/HTML5模板引擎，可以用于Web与非Web应用。
Thymeleaf的主要目标在于提供一种可被浏览器正确显示的、格式良好的模板创建方式，因此也可以用作静态建模。你可以使用它创建经过验证的XML与HTML模板。相对于编写逻辑或代码，开发者只需将标签属性添加到模板中即可。接下来，这些标签属性就会在DOM（文档对象模型）上执行预先制定好的逻辑。Thymeleaf的可扩展性也非常棒。你可以使用它定义自己的模板属性集合，这样就可以计算自定义表达式并使用自定义逻辑。这意味着Thymeleaf还可以作为模板引擎框架。
优点：静态html嵌入标签属性，浏览器可以直接打开模板文件，便于前后端联调。springboot官方推荐方案。
缺点：模板必须符合xml规范

### 3.4 VUE

VUE: 前后端分离,最多,未来趋势

### 

# 四、db

## 1、接口的注入

### 1.1 dao层的接口需要注入

一般的接口是我们自己实现，接口都有自己的实现类，注入时会注入实现类。但是dao层的类我我们定义接口，mybatis/jpa自动实现，那么在编译如何让它通过。

dao层接口：

```java
public interface AccountMapper {
    List<Account> queryAccount();
}
```

#### 1.1.1 mybatis

```java
//在启动类上添加注释，使编译时通过
@MapperScan("com.msb.springboot_mybatis.mapper")     //mybatis实现
@ComponentScan("com.msb.springboot_mybatis.mapper")  //spring实现
//或者在接口类上添加注释，使编译时通过
@Mapper  //mybatis实现
```

#### 1.1.2 jpa

```java
//继承JpaRepository
public interface AccountMapper extends JpaRepository<Account, Long>{
    List<Account> queryAccount();
}
```

### 1.2 接口多种实现时的注入

当接口有多个实现时

```java
public interface Person {
}
@Compoment
public class Man implements Person {
}
@Compoment
public class Woman implements Person {
}
```

#### 1.2.1 单独注入一个

```java
public class PersonService{
    @Autowired
	private List<Person> personList;
}
```

#### 1.2.2 注入多个

```java
//注入一个
public class PersonService{
    @Autowired
	private Person person
}
//只让一个成为组件
public interface Person {
}
@Compoment
@ConditionalOnExpression("true")
public class Man implements Person {
}
@Compoment
@ConditionalOnExpression("false")
public class Woman implements Person {
}
```

## 2、配置

### 2.1 application.yaml

```yaml
spring:
  datasource:
    driver-class-name: com.mysql.jdbc.Driver
    password: 610527
    username: root
    url: jdbc:mysql://127.0.0.1:3306/mydatabase?useUnicode=true&characterEncoding=utf-8
mybatis:
  mapper-locations: classpath:mybatis/*.xml
  type-aliases-package: com.msb.entity
```

### 2.2 Mapper.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN" "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="com.msb.springboot_mybatis.mapper.AccountMapper">
    <select id="queryAccount" resultType="com.msb.springboot_mybatis.entity.Account">select * from account</select>
</mapper>
```

## 3、分页（整合PageHelper）

### 3.1 依赖

```xml
<dependency>
    <groupId>com.github.pagehelper</groupId>
    <artifactId>pagehelper-spring-boot-starter</artifactId>
    <version>1.2.12</version>
</dependency>
```

### 3.2 使用

```java
//提前调用分页方法
Page<Account> pages = PageHelper.startPage(pageNum, pageSize);
//然后再查询
List<Account> accounts = accountMapper.queryAccount();
//将结果存入PageInfo中，因为Page中数据不稳定
PageInfo<Account> pi =new PageInfo<>(accounts);
//结果
System.out.println("当前页"+pi.getPageNum());
System.out.println("总页数"+pi.getPages());
System.out.println("页大小"+pi.getSize());
System.out.println("总记录数"+pi.getTotal());
System.out.println("当前页数据"+pi.getList());
```

## 4、连接池（整合druid）

### 4.1 依赖

```xml
<!--数据库连接池-->
<dependency>
    <groupId>com.alibaba</groupId>
    <artifactId>druid-spring-boot-starter</artifactId>
    <version>1.1.10</version>
</dependency>
```

### 4.2 配置

```yaml
spring:
  datasource:
    driver-class-name: com.mysql.jdbc.Driver
    password: 610527
    username: root
    url: jdbc:mysql://127.0.0.1:3306/mydatabase?useSSL=false&useUnicode=true&characterEncoding=utf-8&serverTimezone=Asia/Shanghai
    type: com.alibaba.druid.pool.DruidDataSource
  druid:
    # 连接池的配置信息
    # 初始化大小，最小，最大
    initial-size: 5
    min-idle: 5
    maxActive: 20
    # 配置获取连接等待超时的时间
    maxWait: 60000
    # 配置间隔多久才进行一次检测，检测需要关闭的空闲连接，单位是毫秒
    timeBetweenEvictionRunsMillis: 60000
    # 配置一个连接在池中最小生存的时间，单位是毫秒
    minEvictableIdleTimeMillis: 300000
    validationQuery: SELECT 1
    testWhileIdle: true
    testOnBorrow: false
    testOnReturn: false
    # 打开PSCache，并且指定每个连接上PSCache的大小
    poolPreparedStatements: true
    maxPoolPreparedStatementPerConnectionSize: 20
    # 配置监控统计拦截的filters，去掉后监控界面sql无法统计，'wall'用于防火墙
    filters: stat,wall,slf4j
    # 通过connectProperties属性来打开mergeSql功能；慢SQL记录
    connectionProperties: druid.stat.mergeSql\=true;druid.stat.slowSqlMillis\=5000
    # 配置DruidStatFilter
    web-stat-filter:
    enabled: true
    url-pattern: "/*"
    exclusions: "*.js,*.gif,*.jpg,*.bmp,*.png,*.css,*.ico,/druid/*"
    # 配置DruidStatViewServlet
    stat-view-servlet:
    url-pattern: "/druid/*"
    # IP白名单(没有配置或者为空，则允许所有访问)
    allow: 127.0.0.1,192.168.8.109
    # IP黑名单 (存在共同时，deny优先于allow)
    deny: 192.168.1.188
    #  禁用HTML页面上的“Reset All”功能
    reset-enable: false
    # 登录名
    login-username: admin
    # 登录密码
    login-password: 123456
```





# 五、其他

## 1、junit

依赖使用spring-boot-starter-test启动器，启动时只需要使用springboot提供的注解@SpringBootTest。

依赖

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-test</artifactId>
    <scope>test</scope>
</dependency>
```

使用

```java
package com.msb;
import com.msb.pojo.Emp;
import com.msb.service.EmpService;
import org.junit.jupiter.api.Test;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.context.SpringBootTest;
import java.util.List;

@SpringBootTest(classes = Springboot03Application.class)
class Springboot03AppliactionTests {
    @Autowired
    private EmpService empService;
    @Test
    public void testFindAll() {
        List<Emp> list = empService.findAll();
        list.forEach(System.out::println);
    }
}
```





## 2、logback

### 2.1 模板

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<configuration>
    <!-- 控制台输出 -->
    <appender name="Stdout" class="ch.qos.logback.core.ConsoleAppender">
        <!-- 日志输出格式 -->
        <layout class="ch.qos.logback.classic.PatternLayout">
            <!--格式化输出：%d表示日期，%thread表示线程名，%-5level：级别从左显示5个字符宽度%msg：日志消息，%n是换行符-->
            <pattern>%d{yyyy-MM-dd HH:mm:ss.SSS} [%thread] %-5level %logger{50} - %msg%n
            </pattern>
        </layout>
    </appender>

    <!-- 按照每天生成日志文件 -->
    <property name="LOG_HOME" value="${catalina.base}/logs/" /> <!--定义日志文件的存储地址 勿在 LogBack 的配置中使用相对路径-->
    <appender name="RollingFile"  class="ch.qos.logback.core.rolling.RollingFileAppender">
        <rollingPolicy class="ch.qos.logback.core.rolling.TimeBasedRollingPolicy">
            <!--日志文件输出的文件名-->
            <FileNamePattern>${LOG_HOME}/server.%d{yyyy-MM-dd}.log</FileNamePattern>
            <MaxHistory>30</MaxHistory>
        </rollingPolicy>
        <layout class="ch.qos.logback.classic.PatternLayout">
            <!--格式化输出：%d表示日期，%thread表示线程名，%-5level：级别从左显示5个字符宽度%msg：日志消息，%n是换行符-->
            <pattern>%d{yyyy-MM-dd HH:mm:ss.SSS} [%thread] %-5level %logger{50} - %msg%n
            </pattern>
        </layout>
        <!--日志文件最大的大小-->
        <triggeringPolicy class="ch.qos.logback.core.rolling.SizeBasedTriggeringPolicy">
            <MaxFileSize>10MB</MaxFileSize>
        </triggeringPolicy>
    </appender>

    <!--日志异步到数据库 -->
    <appender name="DB" class="ch.qos.logback.classic.db.DBAppender">
        <!--日志异步到数据库-->
<!--        <connectionSource class="ch.qos.logback.core.db.DriverManagerConnectionSource">
           &lt;!&ndash;连接池&ndash;&gt;
           <dataSource class="com.mchange.v2.c3p0.ComboPooledDataSource">
              <driverClass>com.mysql.jdbc.Driver</driverClass>
              <url>jdbc:mysql://127.0.0.1:3306/mydatabase</url>
              <user>root</user>
              <password>610527</password>
            </dataSource>
        </connectionSource>
      </appender>-->

    <!-- 日志输出级别 -->
    <root level="info">
        <appender-ref ref="Stdout" />
        <appender-ref ref="RollingFile" />
    </root>
    <!--com.msb.mapper包下日记级别调整为DEBUG，可以打印sql语句-->
    <logger name="com.msb.mapper" level="DEBUG"></logger>
</configuration>
```

### 2.2 整合

![springboot整合logback](springboot.assets/springboot整合logback.png)

springboot默认使用logback

```java
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
private static final Logger LOGGER = LoggerFactory.getLogger(Class clazz);
//接口（协议）都是slf4j，springboot默认实现是logback
```

### 2.3 日志级别

xml格式

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<configuration>
	<!-- 日志输出级别 -->
    <root level="info">
        <appender-ref ref="Stdout" />
        <appender-ref ref="RollingFile" />
    </root>
    <!--com.msb.mapper包下日记级别调整为DEBUG，可以打印sql语句-->
    <logger name="com.msb.mapper" level="DEBUG"></logger>
<configuration>
```

properties格式

```properties
logging.level.root=info
logging.level.com.msb.mapper=DEBUG
```

### 2.4 输出格式

#### 2.4.1 打印方式

日志输出不仅有控制台打印，还可以写入文件，存入数据库

```xml
<root level="info">
    <appender-ref ref="Stdout" />       <!--控制台打印-->
    <appender-ref ref="RollingFile" />  <!--写入文件-->
    <appender-ref ref="DB" />           <!--存库-->
</root>
```

#### 2.4.1 日志格式

```properties
log4j.rootLogger=error,logfile
log4j.appender.stdout=org.apache.log4j.ConsoleAppender
log4j.appender.stdout.Target=System.err
log4j.appender.stdout.layout=org.apache.log4j.SimpleLayout
log4j.appender.logfile=org.apache.log4j.FileAppender
log4j.appender.logfile.File=d:/msb.log
log4j.appender.logfile.layout=org.apache.log4j.PatternLayout
log4j.appender.logfile.layout.ConversionPattern=%d{yyyy-MM-dd HH:mm:ss} %l %F %p %m%n
```

%p：输出日志信息的优先级，即DEBUG，INFO，WARN，ERROR，FATAL。

%d：输出日志时间点的日期或时间，默认格式为ISO8601，也可以在其后指定格式，如：%d{yyyy/MM/dd HH:mm:ss,SSS}。

%r：输出自应用程序启动到输出该log信息耗费的毫秒数。

%t：输出产生该日志事件的线程名。

%l：输出日志事件的发生位置，相当于%c.%M(%F:%L)的组合，包括类全名、方法、文件名以及在代码中的行数。例如

test.TestLog4j.main(TestLog4j.java:10)。
		%c：输出日志信息所属的类目，通常就是所在类的全名。
       %M：输出产生日志信息的方法名。
       %F：输出日志消息产生时所在的文件名称。
       %L:：输出代码中的行号。
       %m:：输出代码中指定的具体日志信息。
       %n：输出一个回车换行符，Windows平台为"rn"，Unix平台为"n"。
       %x：输出和当前线程相关联的NDC(嵌套诊断环境)，尤其用到像java servlets这样的多客户多线程的应用中。
       %%：输出一个"%"字符。







## 3、热部署

### 3.1 配置

需要三步配置：导入依赖、修改idea自动编译、修改Reigstry

导入依赖


```xml
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-devtools</artifactId>
        <version>2.4.5</version>
        <optional>true</optional>
    </dependency>
```

修改idea自动编译

<img src="E:/java/studyNotes/pictures/springboot/idea自动编译.png" alt="idea自动编译" style="zoom:75%;" />

修改Reigstry：Ctrl+Shift+Alt+/，然后点击弹出框中Registry

![idea注册](springboot.assets/idea注册.png)

### 3.2 使用

- 前端页面修改很方便，修改的部分直接编译到字节文件上。
- 后端代码修改不是很快捷，服务还是要重启，只不过系统帮我们重启而已。



















