### 前言：

- 代码：git@github.com:spsunpeng/spring.git

- 学习进度：第11阶段:SSM框架的应用 -》spring(连老师版本) -》 8.SpringAOP声明式事务及源码讲解 -》 00:40:00

- 遗留：spring.io文档、事务管理器、源码


![spring-overview](spring.assets/spring-overview.png)



### 一、入门搭建1（不使用maven）

- 总结不适用maven导包的方法
- 搭建一个简单的spring demo

#### 1. 创建项目

创建java项目

#### 2. 引入jar包

##### 2.1 下载jar包

下载：https://repo.spring.io/libs-release-local/org/springframework/spring/

本地存储位置：E:\jar\spring\spring-framework-5.2.3.RELEASE\libs

##### 2.2 告诉idea(jar包的路径)

导入jar包，此外Module和包Libraries的作用也是导入jar包，Atifacts的作用是导出jar包

![](spring.assets/idea导包方式1.png)

###### 2.2.1 idea管理包介绍

- project settings

  - Projects
  - Modules：作用也是导入jar包
  - Libraries：作用是导入jar包

  - facets：【原意方面】
  - Atifacts：作用是导入jar包，【原意神器】

- platform settings

  - sdks
  - global Libraries：作用也是导入jar包

###### 2.2.2 导包方式

- 新建lib文件夹（非必须），将jar包存放在这里
- 右键，选择 Add as Library
- 弹出窗口，选择导入的方式：global 、Modules、Libraries

![idea导包方式2](spring.assets/idea导包方式2.png)

##### 2.3 此外

运行Spring程序额外还需要commons-logging包，方式一样

下载：http://commons.apache.org/proper/commons-logging/download_logging.cgi

本地：E:\jar

导入方式：一样

#### 3. 创建实体类

```java
public class Person {
    private int id;
    private String name;
    private Long age;
    private String sex;
}
```

#### 4. 配置文件(ioc.xml)

##### 4.1 新建带有spring头信息的xml文件

注：只有在导入spring核心jar包后，idea才会出现此选项

新建 --> XML Configuration File --> Spring Config

![新建spring_xml](spring.assets/新建spring_xml.png)

具有spring头信息的xml文件

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">

</beans>
```

- xmlns:xsi -》xmlns全名xml namespace，那么它的意思就是xml的命名空间是xsi，这个命名空间解释文件中的标签
- 此外，还有xmlns:p命名空间

##### 4.2 配置bean

```xml
    <bean id="person" class="com.mashibing.bean.Person">
        <property name="id" value="001"></property>
        <property name="name" value="sunpeng"></property>
    </bean>
```

#### 5. 使用

```java
public class Test1 {
    public static void main(String[] args) {
        ApplicationContext applicationContext = new ClassPathXmlApplicationContext("com/mashibing/Ioc.xml");
        Person person = (Person)applicationContext.getBean("person");
        System.out.println(person);
    }
}
```





### 二、ioc_xml

- maven优势：maven管理jar包的优势：简单方便，此外，在导入时会自动导入相关的jar包，比如只导入context，会自动导入bean、core、exception，还会自动导入common

#### 1. 基础配置

#####  1.1 创建bean

```xml
	<!--通过无参构造器和get/set方法-->
    <bean id="person" class="com.mashibing.bean.Person">
        <property name="id" value="10"></property>
        <property name="name" value="sunpeng"></property>
        <property name="sex" value="nan"></property>
        <property name="age" value="20"></property>
    </bean>

    <!--通过有参构造器-->
    <bean id="person2" class="com.mashibing.bean.Person">
        <constructor-arg name="id" value="11"></constructor-arg>
        <constructor-arg name="name" value="sunyue"></constructor-arg>
        <constructor-arg name="sex" value="nv"></constructor-arg>
        <constructor-arg name="age" value="18"></constructor-arg>
    </bean>
```

- spring底层通过反射实现，反射获取对象时需要无参构造器
- <property>由set/get方法的名字决定，不由定义的属性名决定
- <constructor-arg>由由构造器器参数名决定，不由定义的属性名决定

##### 1.2 使用

```java
public class Test {
    public static void main(String[] args) {
        ApplicationContext applicationContext = new ClassPathXmlApplicationContext("com/mashibing/Ioc.xml");
        Person person = context.getBean("person", Person.class);
    }
}
```

- 代码：ApplicationContext applicationContext = new ClassPathXmlApplicationContext("com/mashibing/Ioc.xml");
  - ClassPathXmlApplicationContext 继承 ApplicationContext
  - ApplicationContext继承Beanfactory

- 获取对象：Person person = context.getBean("person", Person.class);
  - 参数"person"：配置文件中的 bean.id，通过它可以获取唯一的对象
  - 参数Person.class：也可以通过类获取类名获取对象，好处是返回值知道是类型，但是一个类可能会创建多个对象，导致报错
  - 所以，还是两个都给出吧



##### 1.3 图表diagrams

类图在idea中的查看方式

![diagrams](spring.assets/diagrams.png)

#### 2. 复杂参数bean配置

```xml
<bean name="person3" class="com.mashibing.bean.Person">
    <property name="id" value="10"></property>
    <property name="name" value="sunpeng"></property>
    <property name="sex" value="nan"></property>
    <property name="age" value="20"></property>
    <!--[]-->
    <property name="like" >
        <array>
            <value>book</value>
            <value>woman</value>
        </array>
    </property>
    <!--list-->
    <property name="list">
        <list>
            <value>xiaoming</value>
            <value>xiaohong</value>
        </list>
    </property>
    <!--object-->
    <property name="address">
        <bean class="com.mashibing.bean.Address">
            <property name="province" value="shanxi"></property>
            <property name="city" value="xian"></property>
        </bean>
    </property>
    <!--set<object>-->
    <property name="addresses">
        <set>
            <!--使用bean标签-->
            <bean class="com.mashibing.bean.Address">
                <property name="province" value="shanxi"></property>
                <property name="city" value="xian"></property>
            </bean>
            <!--bean标签中的home标签无用，其他地方调用不到-->
            <bean name="company" class="com.mashibing.bean.Address">
                <property name="province" value="shanxi"></property>
                <property name="city" value="xian"></property>
            </bean>
            <!--ref标签使用外面的bean-->
            <ref bean="home"></ref>
        </set>
    </property>

    <!--map<string,object>-->
    <property name="map">
        <map>
            <!--map<string,string>-->
            <entry key="aa" value="11"></entry>
            <!--map<string,list>，这里array不能用-->
            <entry key="bb">
                <list>
                    <value>21</value>
                    <value>22</value>
                </list>
            </entry>
            <!--map<string,object>-->
            <entry key="cc">
                <bean class="com.mashibing.bean.Address">
                    <property name="city" value="weinan"></property>
                </bean>
            </entry>
            <!--value = value-ref-->
            <entry key="dd" value-ref="home"></entry>
        </map>
    </property>
</bean>

<bean name="home" class="com.mashibing.bean.Address">
    <property name="province" value="shanxi2"></property>
    <property name="city" value="xian2"></property>
</bean>
```

- 标签
  - bean：object
  - array：[]
  - ref：引用bean
  - entry：条目，map内键值对需要的标签
- 建议：设计实体类的时候建议用包装类代替基本类型，用list等代替数组
  - Integer 代替 int：标签type="java.lang.Integer"，这里就不能用int
  - map<string,list> 代替 map<string,array> ：map中使用array标签打印出来的时hashcode，没有toSting重载，不能使用



#### 3. 工厂模式

##### 3.1 xml

```xml
<!--静态工厂-->
<bean name="personF1" class="com.mashibing.factory.PersonStaticFactory" factory-method="getInstance"></bean>
<!--工厂方法-->
<bean name="personInstanceFactory" class="com.mashibing.factory.PersonInstanceFactory"></bean>
<bean name="personF2" class="com.mashibing.bean.Person" factory-bean="personInstanceFactory" factory-method="getInstance"></bean>
<!--FactoryBean：创建bean对象的一种补充-->
<bean name="personFactoryBean" class="com.mashibing.factory.PersonFactoryBean"></bean>
```

- 工厂方法是单例，不要想当然认为它可能是多例的，因为personInstanceFactory的bean是单例的，

##### 3.2 测试

```java
Person personF1 = context.getBean("personF1", Person.class);
Person personF2 = context.getBean("personF2", Person.class)
Person personFactoryBean = context.getBean("personFactoryBean", Person.class);
```

##### 3.3 静态工厂

```java
public class PersonStaticFactory {
    public static Person getInstance(){
        System.out.println("静态工厂");
        Person person = new Person();
        person.setId(1);
        person.setName("jia");
        person.setAge(20L);
        return person;
    }
}
```

##### 3.4 工厂方法

```java
public class PersonInstanceFactory {
    public Person getInstance(){
        System.out.println("工厂方法");
        Person person = new Person();
        person.setId(2);
        person.setName("yi");
        person.setAge(21L);
        return person;
    }
}
```

##### 3.5 FectoryBean

```java
/*
* 此方式是创建bean对象的一种补充，
* （1）用户可以按照自己的需求创建对象（person），创建的对象由ioc容器管理(person)，但是（PersonFactoryBean）这个类本身还是需要配置
* （2）是单例模式，但只会在使用时创建
* */
public class PersonFactoryBean implements FactoryBean<Person> {

    @Override
    public Class<?> getObjectType() {
        return Person.class;
    }

    @Override
    public boolean isSingleton() {
        return true;
    }

    @Override
    public Person getObject() throws Exception {
        System.out.println("FactoryBean");
        Person person = new Person();
        person.setId(10);
        person.setName("zhangsan");
        return person;
    }
}
```



#### 4. 配置外部类

```xml
<!--配置外部类-->
<bean name="dataSource" class="com.alibaba.druid.pool.DruidDataSource">
    <property name="username" value="root"></property>
    <property name="password" value="610527"></property>
    <property name="driverClassName" value="com.mysql.jdbc.Driver"></property>
    <property name="url" value="jdbc:mysql://127.0.0.1:3306/mydatabase"></property>
</bean>
```



#### 5. 引用其他配置文件

##### 5.1 引入context命名空间

```xml
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:context="http://www.springframework.org/schema/context"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
       http://www.springframework.org/schema/beans/spring-beans.xsd
       http://www.springframework.org/schema/context
       https://www.springframework.org/schema/context/spring-context.xsd">
</beans>
```

##### 5.2 配置

```xml
<bean name="dataSource2" class="com.alibaba.druid.pool.DruidDataSource">
    <property name="username" value="${jdbc.username}"></property>
    <property name="password" value="${jdbc.password}"></property>
    <property name="driverClassName" value="${jdbc.driverClassName}"></property>
    <property name="url" value="${jdbc.url}"></property>
</bean>
<context:property-placeholder location="classpath:db.properties"></context:property-placeholder>
```

##### 5.3 外部配置文件

```properties
jdbc.username=root
jdbc.password=610527
jdbc.driverClassName=com.mysql.jdbc.Driver
jdbc.url=jdbc:mysql://127.0.0.1:3306/msb_dongbao
```

- 错误1：Driver无法转换成string

  错误：name="driver"：

  正确：name="driverClassName"：

- 错误2：找不到类“com.mysql.jdbc.Driver”

  错误：jdbc.driverClassName=“com.mysql.jdbc.Driver”

  正确：jdbc.driverClassName=com.mysql.jdbc.Driver



#### 6. 自动装配

```xml
<!--自动装配：autowire=byName/byType-->
<bean name="address001" class="com.mashibing.bean.Address">
    <property name="city" value="xian"></property>
</bean>
<bean name="person001" class="com.mashibing.bean.Person" autowire="byName"></bean>
```



#### 7. SpEL

##### 7.1 SpEL

- SpEL：Spring Expression Language，spring表达语言

```xml
<!--SpEL-->
<bean name="address002" class="com.mashibing.bean.Address">
    <property name="city" value="xian"></property>
</bean>
<bean name="person002" class="com.mashibing.bean.Person">
    <!--运算符-->
    <property name="id" value="#{12-1}"></property>
    <!--bean-->
    <property name="addresses" value="#{address002}"></property>
    <!--bean的属性-->
    <property name="name" value="#{address002.city}"></property>
    <!--bean的方法-->
    <property name="like" value="#{address002.getCity()}"></property>
    <!--静态方法，不依靠bean-->
    <property name="age" value="#{T(System).currentTimeMillis()}"></property>
</bean>
```

##### 7.2 EL

- EL定义：表示你从另外一个页面传过来的参数的值，比如你从另外一个页面传了一个name为name，value为value的参数，在本页面写${name }就会显示value，它会从page，request，session，application中取值。
- eg1：在request中setAttribute（“name”，“测试”），那么${name} 就会得到值为测试。
- eg2：在application.properties中 name=测试 ， 那么${name} 就会得到值为测试。

#### 8. 其他bean标签

其他标签：继承、抽象类、初始化、销毁、加强初始化等标签，需要时百度吧

scope：scope="singleton" / "prototype"



### 三、ioc_注解

#### 1、 扫描包

```xml
<context:component-scan base-package="com.mashibing">
    <!-- exclude：排除，type="assignable"：类型是类，expression="完全限定名"
	<context:exclude-filter type="assignable" expression="com.mashibing.bean.Person"></context:exclude-filter>
	-->
</context:component-scan>
```

#### 

#### 2、构建bean

Spring Bean是被实例的，组装的及被Spring 容器管理的Java对象，Spring 容器会自动完成@bean对象的实例化。

@Component  原意是组件，它是spring最基本的注解，作用是将类注入到spring容器

单例@Scope(scopeName="prototype")

```java
//根据id,默认类名小写，这样貌似就设置不了多个bean了，比如person1、person2
@Component(value = "person2")

//默认单例singleton
@Scope(scopeName="prototype")
```

##### 2.1 配置类

```java
@Configuration
public class MyConfig {

    @Bean
    public User getUser(){
        User user = new User();
        user.setName("sunpeng");
        return user;
    }

    @Bean
    public Person getPerson(User user){
        return new Person();
    }

    @Bean(value = "goodStudent")
    public Student getStudent(){
        return new Student();
    }
}
```

##### 2.1 组件

```java
@Component
public class User {
    @Value("${myUser.name}")
    private String name;

    public String getName() {return name;}
    public void setName(String name) {this.name = name;}
}
@Component
public class Person {
}
@Component(value = "goodStudent")
public class Student {
}
```

#### 3、注入

注意：当使用AutoWired注解的时候，自动装配的时候是根据类型实现的。

​		1、如果只找到一个，则直接进行赋值，

​		2、如果没有找到，则直接抛出异常，

​		3、如果找到多个，那么会按照变量名作为id继续匹配,

​				1、匹配上直接进行装配

​				2、如果匹配不上则直接报异常

```java
@RunWith(SpringJUnit4ClassRunner.class)// 指定测试支持类
@ContextConfiguration("classpath:ioc.xml")// 指定核心配置文件位置
public class Test1 {
    @Autowired
    private User user;

    @Autowired
    private Person person;

    @Autowired
    @Qualifier(value = "goodStudent")
    private Student student;
    
    @Test
    public void testConfig(){
        System.out.println(user);
        System.out.println(person);
        System.out.println(student);
    }
}
```

- @Resource：javac包中的注解，和spring中@AutoWire+@Qualifier的功能一样

  

#### 4. 泛型依赖注入

service

```java
public class BaseService<T> {
    @Autowired
    BaseDao<T> baseDao;

    public void save(){
        System.out.println("自动注入的对象："+baseDao);
        baseDao.save();
    }
}
```

```java
@Service
public class StudentService extends BaseService<Student> {
}
```

```java
@Service
public class TeacherService extends BaseService<Teacher> {
}
```

dao

```java
public class BaseDao <T> {
    public void save(){}
}
```

```java
@Repository
public class StudentDao<T> extends BaseDao<Student> {
    public void save(){
        System.out.println("StudentDao save success");
    }
}
```

```java
@Repository
public class TeacherDao extends BaseDao<Teacher> {
    public void save(){
        System.out.println("TeacherDao save success");
    }
}
```

Test

```java
public class MyTest {

    ClassPathXmlApplicationContext context = new ClassPathXmlApplicationContext("ioc.xml");
    
    @Test
    public void test2(){
        StudentService studentService = context.getBean("studentService", StudentService.class);
        studentService.save();

        TeacherService teacherService = context.getBean("teacherService", TeacherService.class);
        teacherService.save();

    }
     /**
     * 自动注入的对象：com.mashibing.dao.StudentDao@2fd6b6c7
     * StudentDao save success
     * 自动注入的对象：com.mashibing.dao.TeacherDao@5bfa9431
     * TeacherDao save success
     */
}
```

loader: 装载机



### 四、AOP

- AOP就是代理，代理就是包装一层，并非真的取切面
- AOP和代理只能在首尾上切面
- endpoint层是前端在调用，所以代理在实现的调用上有所不同
- 配套自定义注解什么都没有做，只是加了一层判断
- aop用途：日志、拦截器、事务管理器

#### 1、代理与AOP

##### 1.1 源程序

```java
//源程序
methodA{
    //业务
}
```

##### 1.2 加入日志

- 问：现在我想要在头尾加入日志
- 答：那就加呗！

```java
methodA{
    System.out.println("开始");
    //业务
    System.out.println("结束");
}
```

##### 1.3 代理与AOP

- 问：现在我们不想要在程序中加入日志语句，太乱了，我想要去掉他们！我想要切面编程！！！
- 答：使用代理，新建methodB，在methodB中的头尾执行打印语句，并调用methodA

```java
//代理
methodB{
    System.out.println("开始");
    methodA;
    System.out.println("结束");
}
```

##### 1.4 使用

- 问：感觉有什么不对！！！这算什么解决方法？AOP高冷的人设瞬间都没了。
- 问：合着就套了层皮，methodA变成methodB，那么我原先调用methodA的语句要改成methodB了？
- 答：并不用，只要接口做的好！！！JDk用静态方法，而spring-aop在使用时和平常一模一样

```java
//JDk-代理
Calculator calculator = CalculatorProxy.getCalculator(new MyCalculatorImpl());
//注意一：不难看出，接口Calculator的实现用的是代理，继而调用代理方法，所以JDk-代理必须需要接口
//spring-AOP
Calculator calculator = context.getBean( Calculator.class);
```

##### 1.5 补充

- 问：还有，这么做不是只能在头尾添加日志了，还是没有实现切面吧！！！
- 答：是的，只能在头尾添加，切面编程还能真的把一个连贯的程序切开，人家叫一个好听的名字不行吗？
- 问：。。。。。。

#### 2、实现

##### 2.1 JDk-代理：

```java
public class CalculatorProxy {
    public static Calculator getCalculator(final Calculator calculator){
        Class cls = calculator.getClass();
        InvocationHandler invocationHandler = new InvocationHandler() {
            @Override
            public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
                Object result = null;
                try{
                    LogUtil.start(method, args);
                    result = method.invoke(calculator, args);
                    LogUtil.stop(method, result);
                }catch (Exception e){
                    LogUtil.exception(method, e);
                }finally {
                    LogUtil.end(method);
                }
                return result;
            }
        };
        Object o = Proxy.newProxyInstance(cls.getClassLoader(), cls.getInterfaces(), invocationHandler);
        return (Calculator) o;
    }
}
```

这里使用LogUtil打印日志，也可以直接在里面System.out.println("开始/结束");

##### 2.2 spring-AOP

```java
package com.mashibing.util;

import org.aspectj.lang.annotation.*;
import org.springframework.stereotype.Component;

import java.lang.reflect.Method;
import java.util.Arrays;

/**
 * @author sunpeng
 * @Date 2021-01-29 13:22
 */
@Aspect
@Component
public class LogUtilAnnotation {

    @Before("execution( public int com.mashibing.service.MyCalculatorImpl.*(Integer,Integer) )")
    public static void start(){
        System.out.println("方法开始：");
    }

    @AfterReturning("execution( public int com.mashibing.service.MyCalculatorImpl.*(Integer,Integer) )")
    public static void stop(){
        System.out.println("方法stop：");
    }

    @AfterThrowing("execution( public int com.mashibing.service.MyCalculatorImpl.*(Integer,Integer) )")
    public static void exception(){
        System.out.println("方法异常：");
    }

    @After("execution( public int com.mashibing.service.MyCalculatorImpl.*(Integer,Integer) )")
    public static void end(){
        System.out.println("方法end...");
    }

}
```

#### 3、使用场景

- 使用场景：service层的日志打印、endpoint层的拦截器、endpoint层的日志打印
- 不同点：endpoint层和其他层不同，他的调用是前端在调，所以jdk代理在实现上稍有不同，注解在实现上还是一样

#### 4、配套自定义注解

##### 4.1 应用场景

问：为什么要加自定义注解，有哪些应用场景

答：反射取出程序的注解，通过是否添加了注解，直接值是否为ture，判断是否使用代理

问：就这！！！

```java
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.METHOD)
public @interface TokenCheck {
    //检验是否校验token
    boolean required() default false;
}
```

##### 4.2 实现

问：如何实现

答：如果通过代理程序实现，只需在程序中判断注解；如果通过@aspect注解实现，只需设置@aspect中的参数值

```java
//通过代理程序实现
//是否有自定义注解
if(method.isAnnotationPresent(TokenCheck.class)){
    TokenCheck annotation = method.getAnnotation(TokenCheck.class);
    //注解中的请求是否为true
    if(annotation.required()) {}
}

//通过@aspect注解
//不加注解
@Pointcut("execution(* com.sinosun.bizmate.backend.basedata.endpoint..*(..))")
//加注解
@Pointcut("@annotation(* com.sinosun.bizmate.backend.aspect.WebLog)")
```

#### 5、aop

```java
package com.mashibing.util;

import org.aspectj.lang.JoinPoint;
import org.aspectj.lang.ProceedingJoinPoint;
import org.aspectj.lang.Signature;
import org.aspectj.lang.annotation.*;
import org.springframework.core.annotation.Order;
import org.springframework.stereotype.Component;

import javax.swing.*;
import java.lang.reflect.Method;
import java.util.Arrays;

/**
 * @author sunpeng
 * @Date 2021-01-29 13:22
 */

/**
 * "execution( public int com.mashibing.service.*.*(Integer,Integer) )")
 * execution( public * com.mashibing.service.*.*(..) )")
 * "execution( * *(..) )")
 */
@Aspect
@Component
//@Order(2) 多切面时的执行顺序
public class LogUtilAnnotation {

    @Pointcut("execution( public * com.mashibing.service.*.*(..) )")
    public void myPointcut(){}

    @Before("myPointcut(){}")
    public void start(JoinPoint joinPoint){
        String name = joinPoint.getSignature().getName();
        Object[] args = joinPoint.getArgs();
        System.out.println(name+"方法开始："+Arrays.asList(args));
    }

    @AfterReturning(value = "myPointcut(){}", returning = "result") //try里
    public void stop(JoinPoint joinPoint, Integer result){
        String name = joinPoint.getSignature().getName();
        System.out.println(name+"方法AfterReturning："+result);
    }


    @AfterThrowing(value = "myPointcut(){}", throwing = "e")
    public void exception(JoinPoint joinPoint, Exception e){
        String name = joinPoint.getSignature().getName();
        System.out.println(name+"方法异常："+"e");
    }

    @After(value = "myPointcut(){}") //try外,没有参数returning
    private int end(JoinPoint joinPoint){         //private int stop不影响，没用
        String name = joinPoint.getSignature().getName();
        System.out.println(name+"方法After...");
        return 1;
    }

    @Around(value = "myPointcut(){}")
    public Object around(ProceedingJoinPoint joinPoint) {
        String name = joinPoint.getSignature().getName();
        Object result = "";
        try {
            System.out.println(name+"(around)方法开始："+Arrays.asList(joinPoint.getArgs()));
            result = joinPoint.proceed();
            System.out.println(name+"(around)方法AfterReturning："+result);
        } catch (Throwable throwable) {
            System.out.println(name+"(around)方法异常："+"e");
            throwable.printStackTrace();
        }
        System.out.println(name+"(around)方法After...");
        return result;
    }

}
```

##### 5.1切入点表达式

```java
@Pointcut("execution( public int com.mashibing.service.*.*(Integer,Integer) )") 
@Pointcut("execution( public * com.mashibing.service.*.*(..) )") //常用
@Pointcut("execution( * *(..) )") 
```

##### 5.2 参数

```java
public class LogUtilAnnotation {

    @Pointcut("execution( public * com.mashibing.service.*.*(..) )")
    public void myPointcut(){}

    @Before("myPointcut(){}")
    public void start(JoinPoint joinPoint){
        String name = joinPoint.getSignature().getName(); //代理的方法名
        Object[] args = joinPoint.getArgs();  //代理的参数
    }

    @AfterReturning(value = "myPointcut(){}", returning = "result") //代理的结果
    public void stop(JoinPoint joinPoint, Integer result){}


    @AfterThrowing(value = "myPointcut(){}", throwing = "e") //代理的异常
    public void exception(JoinPoint joinPoint, Exception e){}

    @After(value = "myPointcut(){}") 
    private int end(JoinPoint joinPoint){}//它本身 修饰符（private）、返回类型（int）、方法名（stop）随便
```

##### 5.3 AfterReturning和After

```java
 public static Calculator getCalculator(final Calculator calculator){ 
        InvocationHandler invocationHandler = new InvocationHandler() {
            @Override
            public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
                Object result = null;
                try {
                    //start
                    result = method.invoke(calculator, args);
                    //AfterReturning
                } catch (Throwable throwable) {
                    //AfterThrowing
                }
                //After
                return result; 
            }
        };
     	Class cls = calculator.getClass();
        Object o = Proxy.newProxyInstance(cls.getClassLoader(), cls.getInterfaces(), invocationHandler);
        return (Calculator) o;
    }
```

- 区别一：AfterReturning在try{}里面，After在外面

- 区别二：所以，AfterReturning不一定能执行到，After一定能执行到

- 区别三：所以，AfterReturning有返回参数，恰恰因为它不一定能执行到，所以它的返回结果才一定是是有用

##### 5.4 Around

```java
    @Around(value = "myPointcut(){}")
    public Object around(ProceedingJoinPoint joinPoint) {
        String name = joinPoint.getSignature().getName();
        Object result = "";
        try {
            System.out.println(name+"(around)方法开始："+Arrays.asList(joinPoint.getArgs()));
            result = joinPoint.proceed();
            System.out.println(name+"(around)方法AfterReturning："+result);
        } catch (Throwable throwable) {
            System.out.println(name+"(around)方法异常："+"e");
            throwable.printStackTrace();
        }
        System.out.println(name+"(around)方法After...");
        return result;
    }
```

@Around就是 spring_Proxy，它在使用上等同于 jdk_Proxy，所以它可以一个注解就可以代替@start、@AfterReturning、@AfterThrowing、@After

问：同时有@Around 和 @start、@AfterReturning、@AfterThrowing、@After，他们谁先执行

答：@Around注解在外，四个注解在里

```
add(around)方法开始：[1, 2]
add方法开始：[1, 2]
add方法AfterReturning：3
add方法After...
add(around)方法AfterReturning：3
add(around)方法After...
```

##### 5.5 多个代理

问：多个代理的执行顺序？如何改变他们的执行顺序？答：

- 默认按照代理类名的字母顺序
- 可用 @Order(数字)  注解表明第几个执行
- 先执行的在外，后执行的在里

#### 6、jdk与CGLIB

```java
@Test
public void test3(){
    ApplicationContext context = new ClassPathXmlApplicationContext("applicationContext.xml");

    //jdk
    Calculator calculator = context.getBean( Calculator.class);
    calculator.add(1,2);
    System.out.println(calculator); //com.mashibing.service.MyCalculatorImpl@783a467b
    System.out.println(calculator.getClass()); //class com.sun.proxy.$Proxy18

    //CGLIB
    MyCalculator bean = context.getBean(MyCalculator.class);
    bean.add(1, 2);
    System.out.println(bean); //com.mashibing.service.MyCalculator@7db12bb6
    System.out.println(bean.getClass()); //class com.mashibing.service.MyCalculator$$EnhancerBySpringCGLIB$$6dad0c00
}
```

结论：

- 用bean.getClass()验证：接口用jdk，类用CGLIB

- 打印：System.out.println(bean)就是打印bean.tostring()，如果不重写tostring打印的就是全限定名，即bean.getClass()，但这里不一样。

- System.out.println(bean); 还调用了方法After，为什么就只能看源码了

  

#### 7、aop配置方法

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:aop="http://www.springframework.org/schema/aop"
       xsi:schemaLocation="http://www.springframework.org/schema/beans 
       http://www.springframework.org/schema/beans/spring-beans.xsd
       http://www.springframework.org/schema/aop
       http://www.springframework.org/schema/aop/spring-aop.xsd
">
    
    <bean id="myCalculator" class="com.mashibing.service.MyCalculatorImpl"></bean>
    <bean id="logUtil" class="com.mashibing.util.LogUtil"></bean>
    
    <aop:config>
        <aop:aspect ref="logUtil">
            <aop:pointcut id="mypointcut" expression="execution(public * com.mashibing.service.*.*(..))"/>
            <aop:before method="start" pointcut-ref="mypointcut"></aop:before>
            <aop:after-returning method="stop" pointcut-ref="mypointcut" returning="result"></aop:after-returning>
            <aop:after-throwing method="exception" pointcut-ref="mypointcut" throwing="e"></aop:after-throwing>
            <aop:after method="end" pointcut-ref="mypointcut"></aop:after>
            <aop:around method="around" pointcut-ref="mypointcut"></aop:around>
        </aop:aspect>
    </aop:config>

</beans>
```



#### 8、事务管理器 @Transactional

##### 8.1 事务管理器

```java
@Transactional(timeout = 10)
public void checkout(String username,int id){
    bookDao.updateStock(id);
    //int i = 1/0；
    int price = bookDao.getPrice(id);
    bookDao.updateBalance(username,price);
}
```

- 事务：成功则一起成功，保证一致性

##### 8.2 事务管理器的属性

例如：@Transactional(timeout = 10)

| 属性名           | 说明                                                         |
| :--------------- | :----------------------------------------------------------- |
| name             | 当在配置文件中有多个 TransactionManager , 可以用该属性指定选择哪个事务管理器。 |
| propagation      | 事务的传播行为，默认值为 REQUIRED。                          |
| isolation        | 事务的隔离度，默认值采用 DEFAULT。                           |
| timeout          | 事务的超时时间，默认值为-1。如果超过该时间限制但事务还没有完成，则自动回滚事务。 |
| read-only        | 指定事务是否为只读事务，默认值为 false；为了忽略那些不需要事务的方法，比如读取数据，可以设置 read-only 为 true。 |
| rollback-for     | 用于指定能够触发事务回滚的异常类型，如果有多个异常类型需要指定，各类型之间可以通过逗号分隔。 |
| no-rollback- for | 抛出 no-rollback-for 指定的异常类型，不回滚事务。            |

##### 8.3 事务的传播特性

例如：@Transactional(propagation = Propagation.REQUIRED)

| 事务传播行为类型          | 说明                                                         |
| ------------------------- | ------------------------------------------------------------ |
| PROPAGATION_REQUIRED      | 如果当前没有事务，就新建一个事务。如果已经存在一个事务中，加入到这个事务中。(常用) |
| PROPAGATION_REQUIRES_NEW  | 新建事务，如果当前存在事务，把当前事务挂起。                 |
| PROPAGATION_NESTED        | 如果当前没有事务，就新建一个事务。如果已经存在一个事务中，则在**嵌套**事务内执行。 |
|                           |                                                              |
| PROPAGATION_SUPPORTS      | 支持当前事务，如果当前没有事务，就以非事务方式执行。（加不加都一样） |
| PROPAGATION_NOT_SUPPORTED | 以非事务方式执行操作，如果当前存在事务，就把当前事务挂起 （有事务就挂起） |
| PROPAGATION_NEVER         | 以非事务方式执行，如果当前存在事务，则抛出异常。（有事务就抛异常） |
| PROPAGATION_MANDATORY     | 使用当前的事务，如果当前没有事务，就抛出异常。 （必须有事务） |

总结：

```txt
1、事务传播级别是REQUIRED，当checkout()被调用时（假定被另一类中commit()调用），如果checkout()中的代码抛出异常，即便被捕获，commit()中的其他代码都会roll back

2、是REQUIRES_NEW，如果checkout()中的代码抛出异常，并且被捕获，commit()中的其他代码不会roll back；如果commit()中的其他代码抛出异常，而且没有捕获，不会导致checkout()回滚

3、是NESTED，如果checkout()中的代码抛出异常，并且被捕获，commit()中的其他代码不会roll back；如果commit()中的其他代码抛出异常，而且没有捕获，会导致checkout()回滚

    PROPAGATION_REQUIRES_NEW 启动一个新的, 不依赖于环境的 "内部" 事务. 这个事务将被完全 commited 或 rolled back 而不依赖于外部事务, 它拥有自己的隔离范围, 自己的锁, 等等. 当内部事务开始执行时, 外部事务将被挂起, 内务事务结束时, 外部事务将继续执行. 
    另一方面, PROPAGATION_NESTED 开始一个 "嵌套的" 事务,  它是已经存在事务的一个真正的子事务. 嵌套事务开始执行时,  它将取得一个 savepoint. 如果这个嵌套事务失败, 我们将回滚到此 savepoint. 潜套事务是外部事务的一部分, 只有外部事务结束后它才会被提交. 
    由此可见, PROPAGATION_REQUIRES_NEW 和 PROPAGATION_NESTED 的最大区别在于, PROPAGATION_REQUIRES_NEW 完全是一个新的事务, 而 PROPAGATION_NESTED 则是外部事务的子事务, 如果外部事务 commit, 嵌套事务也会被 commit, 这个规则同样适用于 roll back. 
```



### 五、spring整合测试类

#### 1、依赖

```xml
<dependencies>
    <!-- spring -->
    <dependency>
        <groupId>org.springframework</groupId>
        <artifactId>spring-context</artifactId>
        <version>5.3.5</version>
    </dependency>
    <!--spring test测试支持包-->
    <dependency>
        <groupId>org.springframework</groupId>
        <artifactId>spring-test</artifactId>
        <version>5.3.5</version>
    </dependency>
    <!--Junit4单元测试-->
    <dependency>
        <groupId>junit</groupId>
        <artifactId>junit</artifactId>
        <version>4.12</version>
        <scope>compile</scope>
    </dependency>
</dependencies>
```

- 注意spring和spring-test版本必须一样

#### 2、使用

##### 2.1 junit4

```java
@RunWith(SpringJUnit4ClassRunner.class)// 指定测试支持类
@ContextConfiguration("classpath:ioc.xml")// 指定核心配置文件位置
public class Test1 {
    @Autowired
    private User user;
    @Test
    public void testConfig(){
        System.out.println(user);
    }
}
```

##### 2.2 junit5

```java
/*使用ExtentWith和ContextConfiguration注解*/
/*@ExtendWith(SpringExtension.class)
@ContextConfiguration("classpath:applicationContext.xml")*/
// 使用复合注解
@SpringJUnitConfig(locations = "classpath:applicationContext.xml")
public class Test3 {
    @Autowired // 注入要获取的bean
    private  AccountService accountService;
    @Test
    public void testTransaction(){
        int rows = accountService.transMoney(1, 2, 100);
        System.out.println(rows);
    }
}
```



### 六、零xml配置

#### 1、xml配置方式

```xml
applicationContext中,通过AOP实现事务的控制

 <?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:p="http://www.springframework.org/schema/p"
       xmlns:c="http://www.springframework.org/schema/c"
       xmlns:util="http://www.springframework.org/schema/util"
       xmlns:context="http://www.springframework.org/schema/context"
       xmlns:aop="http://www.springframework.org/schema/aop"
       xmlns:tx="http://www.springframework.org/schema/tx"
       xsi:schemaLocation="
       http://www.springframework.org/schema/beans
       http://www.springframework.org/schema/beans/spring-beans.xsd
       http://www.springframework.org/schema/util
       http://www.springframework.org/schema/util/spring-util.xsd
       http://www.springframework.org/schema/context
       http://www.springframework.org/schema/context/spring-context.xsd
       http://www.springframework.org/schema/aop
       http://www.springframework.org/schema/aop/spring-aop.xsd
       http://www.springframework.org/schema/tx
       http://www.springframework.org/schema/tx/spring-tx.xsd
">
    <context:component-scan base-package="com.msb"/>
    <!--读取jdbc配置文件-->
    <context:property-placeholder location="classpath:jdbc.properties"/>
    <!--配置德鲁伊连接池-->
    <bean id="dataSource" class="com.alibaba.druid.pool.DruidDataSource">
        <property name="username" value="${jdbc_username}"></property>
        <property name="password" value="${jdbc_password}"></property>
        <property name="url" value="${jdbc_url}"></property>
        <property name="driverClassName" value="${jdbc_driver}"></property>
    </bean>
    <!--配置JDBCTemplate对象,并向里面注入DataSource-->
    <bean id="jdbcTemplate" class="org.springframework.jdbc.core.JdbcTemplate">
        <!--通过set方法注入连接池-->
        <property name="dataSource" ref="dataSource"></property>
    </bean>
    <!--配置一个事务管理器-->
    <bean id="transactionManager" class="org.springframework.jdbc.datasource.DataSourceTransactionManager">
        <!--将数据源注入事务管理器-->
        <property name="dataSource"  ref="dataSource"></property>
    </bean>
    <!--配置通知-->
    <tx:advice id="txAdvice">
            <!--配置事务参数-->
            <tx:attributes>
                <tx:method name="transMoney" isolation="DEFAULT" propagation="REQUIRED"/>
            </tx:attributes>
    </tx:advice>
    <!--配置AOP-->
    <aop:config>
        <!--配置切入点-->
        <aop:pointcut id="pt" expression="execution(* com.msb.service.AccountService.transMoney(..))"/>
        <!--配置切面-->
        <aop:advisor advice-ref="txAdvice" pointcut-ref="pt"></aop:advisor>
    </aop:config>
</beans>
```

#### 2、配置类方式

```java
package com.msb.config;
import com.alibaba.druid.pool.DruidDataSource;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.ComponentScan;
import org.springframework.context.annotation.Configuration;
import org.springframework.context.annotation.PropertySource;
import org.springframework.jdbc.core.JdbcTemplate;
import org.springframework.jdbc.datasource.DataSourceTransactionManager;
import org.springframework.transaction.PlatformTransactionManager;
import org.springframework.transaction.annotation.EnableTransactionManagement;
import javax.sql.DataSource;

@Configuration  // 配置类标志注解
@ComponentScan(basePackages = "com.msb") // spring包扫描
@PropertySource("classpath:jdbc.properties") // 读取属性配置文件
@EnableTransactionManagement // 开启事务注解
public class SpringConfig {
    //参数
    @Value("${jdbc_driver}")
    private String driver;
    @Value("${jdbc_url}")
    private String url;
    @Value("${jdbc_username}")
    private String username;
    @Value("${jdbc_password}")
    private String password;
    /*创建数据库连接池*/
    @Bean
    public DruidDataSource getDruidDataSource(){ //返回类型决定bean类型，方法明决定bean的id
        DruidDataSource dataSource=new DruidDataSource();
        dataSource.setDriverClassName(driver);
        dataSource.setUrl(url);
        dataSource.setUsername(username);
        dataSource.setPassword(password);
        return dataSource;
    }
    /*创建JdbcTemplate对象*/
    @Bean
    public JdbcTemplate getJdbcTemplate(DataSource dataSource){ //根据类型获取
  //public JdbcTemplate getJdbcTemplate(@Qualifier(value = "getDruidDataSource")DataSource dataSource){ //也可以根据id获取
        JdbcTemplate jdbcTemplate=new JdbcTemplate();
        jdbcTemplate.setDataSource(dataSource);
        return jdbcTemplate;
    }
    /*创建事务管理器*/
    @Bean
    public PlatformTransactionManager getPlatformTransactionManager(DataSource dataSource){
        DataSourceTransactionManager transactionManager =new DataSourceTransactionManager();
        transactionManager.setDataSource(dataSource);
        return transactionManager;
    }
}
```

#### 3、注解方式

全局配置

```java
@ComponentScan(basePackages = "com.msb") // spring包扫描
@PropertySource("classpath:jdbc.properties") // 读取属性配置文件
@EnableTransactionManagement // 开启事务注解
public class SpringConfig {}
```

bean

```java
@Component
public class DruidDataSource{
    //参数
    @Value("${jdbc_driver}")
    private String driver;
    @Value("${jdbc_url}")
    private String url;
    @Value("${jdbc_username}")
    private String username;
    @Value("${jdbc_password}")
    private String password;
}

@Component //bean类型就是类的类型，bean的id是类型的首字母小写
//@Component(value="jdbcTemplate01") //参数value可以设置bean的id
public class JdbcTemplate {
    @Autowired //根据类型获取
    @Qualifier(value = "dataSource01") //根据bean的id获取
    private DataSource dataSource;
}

@Component
public class PlatformTransactionManager {
    @Autowired
    private DataSource dataSource;
}
```



#### 4、测试

```java
@Test()
public void testTransaction3(){
    ApplicationContext context =new AnnotationConfigApplicationContext(SpringConfig.class);
    JdbcTemplate jdbcTemplate = context.getBean(JdbcTemplate.class); //根据类型获取
    JdbcTemplate jdbcTemplate2 = context.getBean(getJdbcTemplate); //根据id获取
    System.out.println(accountService);
}
```

#### 5、总结

配置类方式可以将xml配置更改未java代码，且不用修改原代码。所以，springboot源码中都是使用这个方式，简化了配置。







事务

原子性

隔离性



未授权读取 Lost update    

- 原因：事务B在事务A执行时更改了数据，导致事务B的修改没生效
- 解决方法：行锁，写锁 

脏读 Dirty Reads 

- 原因：事务B在事务A执行时读取数据，但是实际事务A发生了回滚，导致事务B读取的时脏数据
- 解决方法：行锁，读锁（有修改时不可读）

不可重复度 Non-repeatable Reads

- 事务A在事务B执行时，前后读取数据的结果不一样
- 解决方法：行锁，读锁（事务未完成不可读）

幻读

- 原因：统计数据时得到的结果不一致，和不可重复度的不同点在于，他涉及整个表
- 解决方法：表锁，读锁







