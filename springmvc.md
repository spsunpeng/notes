### springmvc

​		MVC是模型(Model)、视图(View)、控制器(Controller)的简写，是一种软件设计规范。就是将业务逻辑、数据、显示分离的方法来组织代码。MVC主要作用是**降低了视图与业务逻辑间的双向偶合**。MVC不是一种设计模式，**MVC是一种架构模式**。当然不同的MVC存在差异。

​		**Model（模型）：**数据模型，提供要展示的数据，因此包含数据和行为，可以认为是领域模型或JavaBean组件（包含数据和行为），不过现在一般都分离开来：Value Object（数据Dao） 和 服务层（行为Service）。也就是模型提供了模型数据查询和模型数据的状态更新等功能，包括数据和业务。

​		**View（视图）：**负责进行模型的展示，一般就是我们见到的用户界面，客户想看到的东西。

​		**Controller（控制器）：**接收用户请求，委托给模型进行处理（状态改变），处理完毕后把返回的模型数据返回给视图，由视图负责展示。 也就是说控制器做了个调度员的工作。

​		其实在最早期的时候还有model1和model2的设计模型0

**最典型的MVC就是JSP + servlet + javabean的模式。**

![springmvc执行流程](springmvc.assets/springmvc执行流程.png)

#### 1、入门demo

- 多个web.xml文件
- 多了在idea中使用时的一堆配置：导包、启动tomcat、tomcat识别prom（tomcat是真的麻烦）

##### 1.1 新建项目

- 方法一：新建maven项目  --》 选择 org.apache.maven.archetypes:maven-archetype-webapp
- 方法二：
  - 新建maven项目
  - 右键项目名称 --》 添加框架支持 --> 选择web（出现web文件夹）

##### 1.2 导包

```xml
<dependencies>
    <!-- https://mvnrepository.com/artifact/org.springframework/spring-context -->
    <dependency>
        <groupId>org.springframework</groupId>
        <artifactId>spring-context</artifactId>
        <version>5.2.3.RELEASE</version>
    </dependency>
    <!-- https://mvnrepository.com/artifact/org.springframework/spring-web -->
    <dependency>
        <groupId>org.springframework</groupId>
        <artifactId>spring-web</artifactId>
        <version>5.2.3.RELEASE</version>
    </dependency>
    <!-- https://mvnrepository.com/artifact/org.springframework/spring-webmvc -->
    <dependency>
        <groupId>org.springframework</groupId>
        <artifactId>spring-webmvc</artifactId>
        <version>5.2.3.RELEASE</version>
    </dependency>
</dependencies>
```

##### 1.3 编写web.xml文件

```xml
	<!--配置DispatcherServlet-->
    <servlet>
        <servlet-name>springmvc</servlet-name>
        <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
        <!--关联springmvc的配置文件-->
        <init-param>
            <param-name>contextConfigLocation</param-name>
            <param-value>classpath:springmvc.xml</param-value>
        </init-param>
    </servlet>
    <!--匹配servlet的请求，/标识匹配所有请求-->
    <servlet-mapping>
        <servlet-name>springmvc</servlet-name>
        <!--/*和/都是拦截所有请求，/会拦截的请求不包含*.jsp,而/*的范围更大，还会拦截*.jsp这些请求-->
        <url-pattern>/</url-pattern>
    </servlet-mapping>
```

##### 1.4 配置文件：springmvc.xml

```xml
<bean id="/hello" class="com.mashibing.controller.HelloController"></bean>

<!--视图解析器-->
<bean id="internalResourceViewResolver" class="org.springframework.web.servlet.view.InternalResourceViewResolver">
    <!--配置前缀-->
    <property name="prefix" value="/WEB-INF/jsp/"></property>
    <!--配置后缀-->
    <property name="suffix" value=".jsp"></property>
</bean>
```

##### 1.5 HelloController.java

###### 1.5.1 导包

![idea导包方式3](D:\Git\gitRepository\studyNotes\myNotes\pictures\javaWeb\idea导包方式3.png)

###### 1.5.1 HelloController

```java
public class HelloController implements Controller {
    @Override
    public ModelAndView handleRequest(HttpServletRequest httpServletRequest, HttpServletResponse httpServletResponse) throws Exception {
        System.out.println("aaaaaaaaaaaaaa");
        //创建模型和视图对象
        ModelAndView mv = new ModelAndView();
        //将需要的值传递到model中
        mv.addObject("msg","hello world");
        //设置要跳转的视图，
        mv.setViewName("hello");
        return mv;
    }
}
```

##### 1.6 创建hello.jsp页面

```jsp
<%@ page contentType="text/html;charset=UTF-8" language="java" %>
<html>
  <head>
    <title>$Title$</title>
  </head>
  <body>
  $END$
  </body>
</html>
```

##### 1.7 启动：tomcat

###### 1.7.1 启动：tomcat

![启动服务器tomcat](D:\Git\gitRepository\studyNotes\myNotes\pictures\javaWeb\启动服务器tomcat.png)

###### 1.7.2 web编译时导包

tomcat启动时不能找到prom.xml导入的包，需要为编译后的字节码文件标明位置

![web编译时导包](springmvc.assets/web编译时导包.png)



#### 2、request

```java
@Controller
public class RequestController {

    @RequestMapping(value = "/params", method = RequestMethod.GET)
    public String params(String name){
        return "success";
    }

    //参数必传，如果不穿：400，参数不匹配
    @RequestMapping(value = "/params2", method = RequestMethod.GET, params = {"name=sunpeng","id"})
    public String params2(String name){
        return "success";
    }

    //参数必传，如果不穿：400，Required String parameter 'name' is not present
    @RequestMapping(value = "/params3")
    public String params3(@RequestParam(value = "name", required = true) String name){
        return "success";
    }

    //使用对象接受参数
    @RequestMapping(value = "/body", method = RequestMethod.GET)
    public String body(String id, Person person){
        return "success";
    }

    //调用放参数必须是json格式，且get方法不支持
    @RequestMapping(value = "/jsonBody", method = RequestMethod.POST)
    public String jsonBody(@RequestBody(require = true) Person person){
        return "success";
    }
    
}
```

#### 3、response

```java
@Controller
public class ResponseController {

    /*
     * return String
     * 1.跳转到视图层
     * 2.如果返回是string类型但没有此静态资源：404，web/string.jsp
     * 3.如果返回不是string类型：500，类型不匹配
     */
    @RequestMapping(value = "/request1", method = RequestMethod.GET)
    public String response1(){
        return "success"; 
    }

     /*
     * return @ResponseBody
     * 1方法的返回值不在作为界面跳转依据,而已直接作为返回的数据
     * 2将方法的返回的数据自动使用ObjectMapper转换为JSON
     */
    @ResponseBody
    @RequestMapping(value = "/response2", method = RequestMethod.GET)
    public String response2(){
        System.out.println("response2");
        return "success";
    }

    //请求转发
    @RequestMapping(value = "/forward")
    public String forward(){
        System.out.println("forward");
        return "forward:WEB-INF/jsp/success.jsp";
    }

    //重定向
    @RequestMapping(value = "/redirect")
    public String redirect(){
        System.out.println("redirect");
        return "redirect:index.jsp";
    }

    //View
    @RequestMapping(value = "/view")
    public View viewFun(HttpServletRequest request){
        System.out.println("view");
        //View view = new InternalResourceView("/forwardPage.jsp");  //请求转发
        View view = new RedirectView(request.getContextPath() + "/redirectPage.jsp"); //重定向
        return view;
    }

    //ModelAndView
    @RequestMapping(value = "/mv")
    public ModelAndView mvFun(HttpServletRequest request){
        System.out.println("view");
        ModelAndView mv = new ModelAndView();
        //请求转发
        mv.setViewName("forward:/forwardPage.jsp");
//        mv.setView(new InternalResourceView("/forwardPage.jsp"));
//        //重定向
//        mv.setViewName("redirect:/redirectPage.jsp");
//        mv.setView(new RedirectView(request.getContextPath() + "/redirectPage.jsp"));
        return mv;
    }

}
```

#### 4、rest

```java
@RestController //@RestController = @Controller + @ResponseBody
public class RestTypeController {

  //所有参数必传: http://localhost:8090/springmvc_first_war_exploded/rest/params/sunpeng/20
    @GetMapping("/restParams/{name}/{id}")
    private String params(@PathVariable("id") int myId, String name){
        System.out.println("name:"+name+"id:"+myId);
        return "success";
    }

    @PostMapping("/restTest")
    private String post(){
        System.out.println("post");
        return "post";
    }
}
```

#### 5、static 静态资源

```xml
<!--静态资源放行，让静态资源不用通过前端控制器-->
<mvc:resources mapping="/static/**" location="/static/"></mvc:resources>
```

#### 6、HandlerInterceptor

##### 6.1 代码

```java
public class MyInterceptor implements HandlerInterceptor {

    @Override
    public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler) throws Exception {
        System.out.println("MyInterceptor preHandle");
        /*设置请求和响应的乱码 */
        request.setCharacterEncoding("UTF-8");
        response.setCharacterEncoding("UTF-8");
        //token校验等等
        return true;
    }

    @Override
    public void postHandle(HttpServletRequest request, HttpServletResponse response, Object handler, ModelAndView modelAndView) throws Exception {
        System.out.println("MyInterceptor postHandle");

        //过滤返回的敏感字
        Map<String, Object> model = modelAndView.getModel();
        String msg = (String)model.get("msg");
        String newMsg = msg.replace("脏话", "**");
        model.put("msg", newMsg);

        //新的跳转页面
//        modelAndView.setViewName("/index.jsp");
    }

    //无论controller是否出现异常,都会执行的方法，一般来说都做一些资源释放工作
    @Override
    public void afterCompletion(HttpServletRequest request, HttpServletResponse response, Object handler, Exception ex) throws Exception {
        System.out.println("MyInterceptor afterCompletion");
        System.out.println(ex);
    }
}
```

##### 6.2 配置

```xml
<!--注册拦截器，谁先注册（在上）谁先执行（在外层），-->
<mvc:interceptors>
    <mvc:interceptor>
        <mvc:mapping path="/login"/>
        <bean id="myInterceptor" class="com.msb.interceptor.MyInterceptor"></bean>
    </mvc:interceptor>

    <mvc:interceptor>
        <mvc:mapping path="/login"/>
        <bean id="myInterceptor2" class="com.msb.interceptor.MyInterceptor2"></bean>
    </mvc:interceptor>
</mvc:interceptors>
```

#### 7、exception

##### 7.1 方法1

```java
@Controller
public class ExceptionController {

    @Autowired
    private ExceptionService exceptionService;

    @GetMapping("/test1")
    public String text1(){
        int i = 1/0;
        return "success";
    }
    @GetMapping("/test2")
    public String text2(){
        String str = null;
        int length = str.length();
        return "success";
    }
    @GetMapping("/test3")
    public String text3(){
        exceptionService.text3();
        return "success";
    }

    /**
    * 只能处理这个类下中的异常
    */
    @ExceptionHandler(value ={ArithmeticException.class,NullPointerException.class} )
    public ModelAndView handelException(){
        ModelAndView mv =new ModelAndView();
        mv.setViewName("error1.jsp");
        return mv;
    }

}
```

##### 7.2 方法2

```java
@ControllerAdvice
public class GloableException1 {
    @ExceptionHandler(value ={ArithmeticException.class,NullPointerException.class} )
    public ModelAndView handelException(){
        ModelAndView mv =new ModelAndView();
        mv.setViewName("error1.jsp");
        return mv;
    }

}
```

##### 7.3 方法3

###### 7.3.1 xml配置

```xml
<bean id="simpleMappingExceptionResolver" class="org.springframework.web.servlet.handler.SimpleMappingExceptionResolver">
    <property name="exceptionMappings">
        <props>
            <prop key="java.lang.ArithmeticException">redirect:/error1.jsp</prop>
            <prop key="java.lang.NullPointerException">redirect:/error2.jsp</prop>
        </props>
    </property>
</bean>
```

###### 7.3.2 @bean方法

```java
@Configuration
public class GlobalException2 {

    @Bean
    public SimpleMappingExceptionResolver getSimpleMappingExceptionResolver(){
        SimpleMappingExceptionResolver resolver = new SimpleMappingExceptionResolver();
        Properties prop = new Properties();
        prop.put("java.lang.NullPointerException","error1.jsp");
        prop.put("java.lang.ArithmeticException","error2.jsp");
        resolver.setExceptionMappings(prop);
        return resolver;
    }

}
```

##### 7.4 方法4

```java
@Configuration
public class GlobalException3 implements HandlerExceptionResolver {
    @Override
    public ModelAndView resolveException(HttpServletRequest httpServletRequest, HttpServletResponse httpServletResponse, Object o, Exception e) {
        ModelAndView mv = new ModelAndView();
        if(e instanceof NullPointerException){
            mv.setViewName("/error2.jsp");
        }else if(e instanceof ArithmeticException){
            mv.setViewName("/error1.jsp");
        }
        /*Map<String, Object> model = mv.getModel(); 
        model.put("error", e);*/
        mv.addObject("error",e);
        return mv;
    }
}
```

#### 8、总结

##### 8.1 springmvc.xml配置

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:context="http://www.springframework.org/schema/context"
       xmlns:mvc="http://www.springframework.org/schema/mvc" xmlns:mv="http://www.springframework.org/schema/mvc"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
       http://www.springframework.org/schema/beans/spring-beans.xsd
       http://www.springframework.org/schema/context
       https://www.springframework.org/schema/context/spring-context.xsd
       http://www.springframework.org/schema/mvc
       http://www.springframework.org/schema/mvc/spring-mvc.xsd">

    <!--自动扫描包，由IOC容器进行控制管理-->
    <context:component-scan base-package="com.msb"></context:component-scan>


    <!--配置处理器映射器-->
    <!-- <bean class="org.springframework.web.servlet.mvc.method.annotation.RequestMappingHandlerMapping"></bean>-->
    <!--配置处理器适配器-->
    <!-- <bean class="org.springframework.web.servlet.mvc.method.annotation.RequestMappingHandlerAdapter"></bean>-->
    <!--一个注解替换上面的两个配置-->
    <mvc:annotation-driven/>

   <!--配置视图解析器：加前缀和后缀-->
    <bean class="org.springframework.web.servlet.view.InternalResourceViewResolver">
        <property name="prefix" value="/WEB-INF/jsp/"></property>
        <property name="suffix" value=".jsp"></property>
    </bean>

    <!--静态资源放行，让静态资源不用通过前端控制器-->
    <mvc:resources mapping="/static/**" location="/static/"></mvc:resources>

    <!--注册拦截器，谁先注册（在上）谁先执行（在外层），-->
    <mvc:interceptors>
        <mvc:interceptor>
            <mvc:mapping path="/login"/>
            <bean id="myInterceptor" class="com.msb.interceptor.MyInterceptor"></bean>
        </mvc:interceptor>

        <mvc:interceptor>
            <mvc:mapping path="/login"/>
            <bean id="myInterceptor2" class="com.msb.interceptor.MyInterceptor2"></bean>
        </mvc:interceptor>
    </mvc:interceptors>

    <!--异常-->
	<bean id="simpleMappingExceptionResolver" class="org.springframework.web.servlet.handler.SimpleMappingExceptionResolver">
        <property name="exceptionMappings">
            <props>
                <prop key="java.lang.ArithmeticException">redirect:/error1.jsp</prop>
                <prop key="java.lang.NullPointerException">redirect:/error2.jsp</prop>
            </props>
        </property>
    </bean>

</beans>
```

##### 8.2 web.xml配置

```xml
<?xml version="1.0" encoding="UTF-8"?>
<web-app xmlns="http://xmlns.jcp.org/xml/ns/javaee"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://xmlns.jcp.org/xml/ns/javaee http://xmlns.jcp.org/xml/ns/javaee/web-app_4_0.xsd"
         version="4.0">

    <!--配置前端控制器-->
    <servlet>
        <servlet-name>springmvc</servlet-name>
        <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
        <init-param>
            <param-name>contextConfigLocation</param-name>
            <!--需要控制的bean对象-->
            <param-value>classpath:springmvc.xml</param-value>
        </init-param>
    </servlet>
    <servlet-mapping>
        <servlet-name>springmvc</servlet-name>
        <!--请求资源：/代表除过jsp文件的所有资源，/*包裹jsp的所有资源-->
        <url-pattern>/</url-pattern>
    </servlet-mapping>

    <!--jsp没有能力发送post/delete请求，此过滤器就是将jsp的post请求转换成post/put/delete请求-->
    <filter>
        <filter-name>hiddenHttpMethodFilter</filter-name>
        <filter-class>org.springframework.web.filter.HiddenHttpMethodFilter</filter-class>
    </filter>
    <filter-mapping>
        <filter-name>hiddenHttpMethodFilter</filter-name>
        <url-pattern>/*</url-pattern>
    </filter-mapping>
</web-app>
```























