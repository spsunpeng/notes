# 一、jackson

### 1、ObjectMapper

#### 1.1 转换

```java
ObjectMapper objectMapper= new ObjectMapper()
User user = new User();
String string = ""
List<User> userList = new ArrayList<>();
```

| source   | target   | to                                                           |
| -------- | -------- | ------------------------------------------------------------ |
| String   | Object   | objectMapper.readValue(string, User.class) <br />objectMapper.readValue(string, new TypeReference<List<User>>() { })<br/>**注意1：string中的字段许少不许多** |
| String   | JsonNode | objectMapper.readTree(string);<br/>**补充：通过jsonNode.get(string)方法也可以取出字段值** |
| Object   | String   | objectMapper.writeValueAsString(user)                        |
| Object   | JsonNode | objectMapper.valueToTree(string);                            |
| JsonNode | Object   | 目前没用到                                                   |
| JsonNode | String   | toString                                                     |

#### 1.2 配置

```java
//忽略josnString中存在，对象中不存在的情况
objectMapper.configure(DeserializationFeature.FAIL_ON_UNKNOWN_PROPERTIES, false);
```



### 2、ObjectNode

#### 2.1 构建ObjectNode

```java
ObjectNode objectNode = objectMapper.createObjectNode();
User user = new User();

//设置string
objectNode.put("name", "suoeng");
//设置objectNode/jsonNode/arrayNode
objectNode.set("user", objectMapper.valueToTree(user)); //作为整体存入
objectNode.setAll(objectMapper.valueToTree(user));      //每个字段单独存入
```



### 3、JsonNode

```java
//取值
JsonNode jsonNode = objectNode;
jsonNode.get(string)
```



### 4、总结

#### 4.1 异常

##### 4.1.1 编译异常

JsonProcessingException

- 运行异常，所以必须处理
- json定义的异常，之所以定义是因为一些方法处理不了null，所以这类方法使用时要校验参数是否未空，比如readValue，反之未见此类异常说明参数可以穿空

##### 4.1.2 String不是json格式

#### 4.2 类型

##### 4.2.1 类型

```java
User user = new User();
//Object类型
System.out.println(User.class);       //class com.example.json.model.User
System.out.println(user.getClass());  //class com.example.json.model.User

//List类型
List<User> userList = new ArrayList<>();
System.out.println(userList.getClass()); //class java.util.ArrayList
System.out.println(List.class);          //interface java.util.List
System.out.println(List<User>.class);       //错误

//List<User>类型 json提供
TypeReference<List<User>> typeReference = new TypeReference<List<User>>() {};
System.out.println(typeReference.getType());//java.util.List<com.example.json.model.User>
```

由此可见，new TypeReference<T>() {} 可以获取类型

##### 4.2.2 类型传递

| 实参                    | 形参                  | 场景                          |
| ----------------------- | --------------------- | ----------------------------- |
| User.class              | Class<T> clazz        | 1.获取实例；2.获取类型clazz   |
| new TypeReference<User> | TypeReference<T> type | 获取类型，如 User、List<User> |
|                         |                       |                               |
