### 2020.12.23

### 项目整体框架搭建，整体设计一

#### 1. 课堂记录

1. 项目框架

2. git\sourcetree\gitee

   git@gitee.com:cpf100/msb-dongbao-mall.git

3. mysql和duck

4. lombok与代码生成器与mybatis-plu

5. @MapperScan("com.msb.msbdongbaoums.mapper")

6. maven的使用，父项目<parent>



### 2020.12.30

1.导入依赖

群组-服务名-版本号：确定唯一jar包

parent依赖的版本号：是否可以不同

2.导入依赖

如何识别其他模块的，install什么情况下用

3.扫包

springbootApplication、mapperscan还可以扫其他模块的包

@SpringBootApplication(scanBasePackages = {"com.msb"}) 还是 @Service 或者 @Component

4.时间

linux设置时间，代码自动修改时间，国际时间标准



#### 扫描、注入、组件

##### 1.扫：扫描其他类

```java
//扫描本模块下的类
@SpringBootApplication
//扫描本模块"com.msb.dongbao.ums"包 ，和"com.msb.dongbao.portal"包 下的类
@SpringBootApplication(scanBasePackages = {"com.msb.dongbao.ums", "com.msb.dongbao.portal"})
//合并，可以写做扫描包"com.msb.dongbao"
@SpringBootApplication(scanBasePackages = {"com.msb.dongbao"})
//那么，是不是也可以扫描包 "com", "org"，这样会扫描所有的引用的jar包下的类
@SpringBootApplication(scanBasePackages = {"com.msb.dongbao"}, "com", "org")
```

##### 2.注入：扫描时为了注入

```java
//注意，扫描是为了注入，扫描其他模块的包"com.msb.dongbao.ums"，就可以其他模块的组件注入
@SpringBootApplication(scanBasePackages = {"com.msb.dongbao"})
@Autowired
private UmsMemberService umsMemberService;

//如果，不需要注入，那么只要引入jar包就行了
private UmsMemberService umsMemberService;
/*
		<dependency>
            <groupId>com.msb</groupId>
            <artifactId>msb-dongbao-ums</artifactId>
            <version>1.0-SNAPSHOT</version>
        </dependency>
*/
```

##### 3.组件：接口和实现类只需要接口时组件

```java
public interface UmsMemberService {
}

@Service
public class UmsMemberServiceImpl implements UmsMemberService {
}

/*
只有UmsMemberServiceImpl需要添加 @Component 组件的注解
*/
```



版本1：

- 场景：

  api下无 springboot-pulin

  application无springboot-pulin ， 依赖api 和 service

  service 有 springboot-pulin， 依赖 api

- 结果

  仓库中有 api 包

  service 确定依赖 api 

  application 确定依赖 api 和 service

  

版本3

- 更改

  删除service 中的 springboot-pulin

  ```xml
  <build>
      <plugins>
          <plugin>
              <groupId>org.springframework.boot</groupId>
              <artifactId>spring-boot-maven-plugin</artifactId>
          </plugin>
      </plugins>
  </build>
  ```

- 结果

  没有任何影响



版本4

- 修改

  service 增加 pom

  ```xml
  <packaging>pom</packaging>
  ```





spring-boot-maven-plugin打包出来的jar是不可依赖的
我们现在整合后的maven项目有一个parent工程，打包类型为pom，下面多个spring-boot工程作为它的module，分别为子模块和moduleA,moduleB。假如moduleA依赖于子模块。如果你在子模块中使用了spring-boot-maven-plugin的默认配置build，或者在parent工程中使用spring-boot-maven-plugin的默认配置build。那么在clean package的时候会发现moduleA找不到子模块中的类。原因就是默认打包出来的jar是不可依赖的。



### 2021.01.05

1.时间（遗留）、公司与宿舍环境、宿舍mysql

2.环境、密码脱敏、注册登录



### 2020.01.06

#### 一、错误

##### 错误1：请求方式出错

报错：postman {“status": 405,  "error": "Method Not Allowed"}

原因：代码get接口，请求post接口

##### 错误2：找不到Mapper的配置文件

报错：org.apache.ibatis.binding.BindingException: Invalid bound statement (not found): com...ums...selectByName

原因：在service-ums模块下配置的mapper-locations没有起作用，必须在web模块下配置

#### 二、知识点

##### 知识点1：配置文件不在rescourse下打包

比如Mapper.xml文件不在rescourse目录下，而在java目录下，所以在pom.xml文件下需要加入build标签

```xml
<build>
    <resources>
        <resource>
            <directory>src/main/resources</directory>
        </resource>
        <resource>
            <directory>src/main/java</directory>
            <includes>
                <include>**/*.xml</include>
            </includes>
        </resource>
    </resources>
</build>
```

##### 知识点2：密码脱敏

依赖

```xml
<dependency>
    <groupId>org.springframework.security</groupId>
    <artifactId>spring-security-crypto</artifactId>
    <scope>test</scope>
</dependency>
<dependency>
    <groupId>org.springframework.security</groupId>
    <artifactId>spring-security-crypto</artifactId>
    <scope>compile</scope>
</dependency>
```

使用

```java
BCryptPasswordEncoder bCryptPasswordEncoder = new BCryptPasswordEncoder();
//加密
umsMember.setPassword(bCryptPasswordEncoder.encode(umsMember.getPassword()));
//密码验证
boolean bool = passwordEncoder.matches("密码", "加密过的密码");

//md5加密法，但是由于每次同一个密码每次加密的结果都是一样的，所以通过彩虹吗可以破解md5加密
DigestUtils.md5DigestAsHex(sourceString.getBytes());
```





### 2021.01.12

#### 课堂笔记

1.序列化

2.@Builder 注解 链式调用

set返回void

builder返回对象

404 资源找不到，url出错

把json当成页面去查了，没有就报了404？

如果是controller spring默认找的是界面

405 请求方法错误，get/post

3.integer可以表示 没有，这在接口中就比较重要

4.springboot异常处理：解决最终抛到到controller层的异常

#### 主要内容

1.统一返回

2.异常处理

3.参数校验



### 2021.01.16

#### 1. 统一返回

使用lombok注解@Builder

- 作用：简化set，由于设置字段时，需要多次set，而.builder()和.build()方法可以简化这种操作，因为返回的是原对象，即链式调用

```java
@Data
@Builder
public class ResultWrapper<T> implements Serializable {

    private Integer code;
    private String msg;
    private T data;

    //使用lombok注解@Builder
    public static ResultWrapper.ResultWrapperBuilder getSuccess(){
        return ResultWrapper.builder().code(StatusErrorEnums.SUCCESS.getCode()).msg(StatusErrorEnums.SUCCESS.getMsg());
    }
    
    //不使用lombok注解@Builder
    public static ResultWrapper.ResultWrapperBuilder getSuccess2(){
        ResultWrapper result = ResultWrapper();
        result.setCode(StatusErrorEnums.SUCCESS.getCode());
        result.setMsg(tatusErrorEnums.SUCCESS.getMsg());
        return result;
    }  
}
```

#### 2. 异常处理

- 方法1：在类上使用注解表明此类是处理异常的类，在方法上使用注解表示处理哪个类

```java
//表明这个类是处理异常的类
@RestControllerAdvice
public class GlobalExceptionHandle {

    //处理哪一种异常注解
    @ExceptionHandler(ArithmeticException.class)
    public ResultWrapper customException(){
        return ResultWrapper.builder().code(301).msg("服务异常").build();
    }
}
```

- 方法2：在类上使用注解表明此类是处理异常的类，同类还继承第三方方法，再重写需要更改的方法

```java
@ControllerAdvice
public class ValidateHandler extends ResponseEntityExceptionHandler {

    @Override
    protected ResponseEntity<Object> handleMethodArgumentNotValid(
            MethodArgumentNotValidException ex
            , HttpHeaders headers
            , HttpStatus status
            , WebRequest request) {

        //把第一个字段错误取出返回，ex.getBindingResult().getFieldErrors()是参数校验的错误
        String msg = "";
        for(FieldError fieldError : ex.getBindingResult().getFieldErrors()){
            msg = fieldError.getDefaultMessage();
            break;
        }

        return new ResponseEntity(ResultWrapper.builder().code(102).msg(msg).build(), HttpStatus.OK);
    }

}
```

#### 3. 参数校验

坐标

```xml
<!--spring validation 校验-->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-validation</artifactId>
</dependency>
```

使用

```java
@Valid
@NotNull
@Size(min="", max="", message="")
```



eyJhbGciOiJIUzI1NiJ9.eyJzdWIiOiLlrZnpuY8iLCJleHAiOjE2MTExMDY4Nzh9.h_Sl4EqS3wvZe5iSL7N-CIg5ZQQQNGpGnG_gxvDBekp



### 2021.01.20

#### 1. Jwt

##### 1.1 pom依赖

```xml
<dependency>
    <groupId>io.jsonwebtoken</groupId>
    <artifactId>jjwt</artifactId>
    <version>0.7.0</version>
</dependency>
```

##### 1.2 创建与解析token

```java
public class JwtUtil {

    //加密的“盐”
    private static final String secret = "abcdef";

	//创建
    public static String createToken(String subject){

        String token = Jwts.builder().setSubject(subject)
                .setExpiration(new Date(System.currentTimeMillis() + 1000 * 60))
                .signWith(SignatureAlgorithm.HS256, secret)
                .compact();

        return token;
    }

	//解析
    public static String parseToken(String token){
        Claims body = Jwts.parser().setSigningKey(secret).parseClaimsJws(token).getBody();
        String subject = body.getSubject();
        return subject;
    }
}
```

- 翻译：token【代币、符记、象征】、secret【秘密】、parse【解析】、subject【主题】、Expiration【过期】、compact【契约】

- 加解密分析：

  eyJhbGciOiJIUzI1NiJ9                                                                                 --》HS256通过base64编码的结果

  .eyJzdWIiOiJ4aWFvaG9uZyIsImV4cCI6MTYxMTEwNzYyM30              --》subject和Expiration通过base64编码的结果

  .R6cl12yY4x59F4rBph-Lu3SCcK0jhH6XN5w0wXd1Lys                       --》根据secret对subject加密的结果，无法解析

##### 1.3 使用

```java
public static void main(String[] args) throws InterruptedException {
    String name = "孙鹏";
    String token = createToken(name);
    System.out.println(token);

    TimeUnit.SECONDS.sleep(2);

    String subject = parseToken(token);
    System.out.println(subject);
}
```

token过期

异常：ExpiredJwtException

##### 1.4 Jwt的特点与作用

- 作用：为登录的用户返回一个token，后续访问就不需要密码。此外，还有其他解决方案，比如为登录的用户分配一个对应的id，但这里就需要存储对应关系了。
- 特点/优势：自加密解析，不用存储，可以保证token自身的安全，但无法保证携带的信息的安全
- 遗留：如何保护携带的信息的安全

#### 2 加密

base64：编解码，不是加密，所以可以反解码

散列算法：md5【不可反解码，但由于对于主题的加密结果一样，可以通过彩虹表（对应表）破解】

对称加密：两方持有相同的密钥，加解密只需要密钥即可

非对称加密：公加私解，私加公解



### 2020.1.28

#### 1、token校验

##### 1、拦截器

###### 1.1 拦截器

```java
public class AuthInterceptor implements HandlerInterceptor {

    @Override
    public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler) throws Exception {

        System.out.println("进入拦截器");
        //有没有token
        String token = request.getHeader("token");
        if(null == token){
            throw new TokenException("token为空");
        }

        HandlerMethod handlerMethod = (HandlerMethod) handler;
        Method method = handlerMethod.getMethod();
        //有没有注解
        if(method.isAnnotationPresent(TokenCheck.class)){
            TokenCheck annotation = method.getAnnotation(TokenCheck.class);
            //注解中的请求是否为true
            if(annotation.required()) {
                try {
                    //校验token
                    JwtUtil.parseToken(token);
                    return true;
                } catch (Exception e) {
                    throw new TokenException("token校验错误");
                }
            }
        }
        return true;
    }

    @Override
    public void postHandle(HttpServletRequest request, HttpServletResponse response, Object handler, ModelAndView modelAndView) throws Exception {

    }

    @Override
    public void afterCompletion(HttpServletRequest request, HttpServletResponse response, Object handler, Exception ex) throws Exception {

    }
}
```

###### 1.2 拦截器配置

```java
@Configuration
public class InterceptorConfig implements WebMvcConfigurer {

    @Override
    public void addInterceptors(InterceptorRegistry registry) {
        registry.addInterceptor(authInterceptor()).addPathPatterns("/**").excludePathPatterns("/user-member/login");
    }

    @Bean
    public AuthInterceptor authInterceptor(){
        return new AuthInterceptor();
    }
}
```

##### 2、注解

```java
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.METHOD)
public @interface TokenCheck {
    //检验是否校验token
    boolean required() default false;
}
```

##### 3、使用

```java
@PostMapping("/edit")
@TokenCheck(required = true)
public ResultWrapper edit(@RequestBody UmsMember umsMember){
    return umsMemberService.edit(umsMember);
}
```

##### 4、异常处理

###### 4.1 定义异常

```java
public class TokenException extends  Exception{
    public TokenException(String msg) {
        super(msg);
    }
}
```

###### 4.2 统一异常返回

```java
@RestControllerAdvice
public class GlobalExceptionHandle {
    @ExceptionHandler(TokenException.class)
    public ResultWrapper loginException(Exception e){
        return ResultWrapper.getFailBuilder().code(501).msg(e.getMessage()).build();
    }
}
```

##### 5、补充

问：这里功能的实现可以不使用注解吗？

答：可以

- 首先，拦截器配置中可以添加需要校验的类

- 然后，拦截器实现中如果不去解析注解的话

- 那么，不适用注解可以完成功能

  

#### 2、日志切面

```java
@Aspect
@Componet
public class WebLogAscept{
    
    //不加注解
    @Pointcut("execution(* com.sinosun.bizmate.backend.basedata.endpoint..*(..))")
    //加注解
    @Pointcut("@annotation(* com.sinosun.bizmate.backend.aspect.WebLog)")
    public void weblog(){}
    
    @Before("webLog()")
    public void doBefore(JoinPoint joinPoint) throws Throwable{
        //方法前执行
    }
    
    @Around("webLog()")
    public object doAround(ProceedingJoinPoint proceedingjoinPoint) throws Throwable{
        //方法后执行
    }
    
    @After("webLog()")
    public void doAfter() throws Throwable{
        //方法后执行
    }
}
```

```java
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.METHOD)
public @interface WebLog {
}
```











































