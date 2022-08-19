### 1、 JVM

- java  ->  java 8.0

- jdk -> 1.8
- jre

- jvm ：java虚拟机，跨语言跨平台
  - 相当于操作系统，执行class文件，只要是能编译成class文件的都可以在jvm上运行
  - 是一种规范 HotSpot是主流用的，由oracle提供，8.0后估计要收费



### 2、class文件

- 打开：

  - 2进制文件

  - 普通文本文件打开时乱码的，需要专门的2进制文件查看器才能查看，比如 ：

    idea的插件：BinED

    notepag的插件：HEX-Editor

  - 反编译：idea生成的out目录下的文件就是2进制文件，但双击打开并不是直接打开，二十反编译再打开

    常用的反编译软件还有，idea...



严镇涛







其实并不是因为有javac将Java源码编译成class文件，才说Java属于编译+解释语言，因为在这个编译器编译之后，生成的类文件不能直接在对应的平台上运行。 那为何又说Java是编译+解释语言呢？因为class文件最终是通过JVM来翻译才能在对应的平台上运行，而这个翻译大多数时候是解释的过程，但是也会有编译，称之为运行时编译，即JIT(Just In Time)。 综上所述，Java是一门编译型+解释型的高级语言。





notepad++ 查看二级制文件

安装插件

notepad++ --> 插件 --> Plugin Manager --> show plugin manager

![image-20220817105053588](jvm.assets/image-20220817105053588.png)



使用插件

notepad++打开二级制文件 --> 插件 --> HEX-Editor --> View in HEX

切换2进制/8进制/16进制

notepad++ --> View in --> 切换格式





## java命令

```sh
javac Person.java #编译，得到二级制文件
java Person #执行，注意执行的不是Person.class文件，而是Person的全限定名
javap -v -p Person.class >Person.txt #转化为类汇编语言
```



java二级制文件（.class）文件查看方式

- 用相关软件或插件打开，得到二进制或16机制文件
- 转化为类汇编语言打开，用javap -v命令，它保留了机器语言的加载方式和顺序，同时让程序员可以大致阅读所以汇编语言是对机器友好的语言。
- 用反编译软件打开。相比源代码进行了优化，专业的反编译软件会保留行号，而idea加载的jar包不会，排查bug需要注意。





声明周期

装载  链接  初始化  使用  卸载





类加载-准备阶段

static final: ConstantValue 

static 只开辟空间，为0

变量：在对象初始化时才分配空间



类加载：装载，链接，初始化

虚拟机把Class文件加载到内存 并对数据进行校验，转换解析和初始化 形成可以虚拟机直接使用的Java类型，即java.lang.Class



处理：类，对象（入口），字段，方法（程序），静态量（类名，字段，方法名）



- 装载
  - 通过类的全限定名获取二级制字节流，在jvm外部实现，这个模块叫类加载器
  - 将字节流所代表的静态存储结构转化为方法区运行时的数据结构。（加载到内存对应的位置上）
  - 在java堆中生成一个代表这个类的java.lang.class对象，作为方法区中这些数据的访问入口
- 链接
- 初始化
- 使用
- 卸载



获取类的二进制字节流的阶段是我们JAVA程序员最关注的阶段，也是操控性最强的一个阶段。因为这个阶段我 们可以对于我们的类加载器进行操作，比如我们想自定义类加载器进行操作用以完成加载，又或者我们想通过 JAVA Agent来完成我们的字节码增强操作。





1）Bootstrap ClassLoader 负责加载$JAVA_HOME中 jre/lib/rt.jar 里所有的class或Xbootclassoath选项指定的jar包。由C++实现，不是ClassLoader子类。

 2）Extension ClassLoader 负责加载java平台中扩展功能的一些jar包，包括$JAVA_HOME中jre/lib/*.jar 或 -Djava.ext.dirs指定目录下的jar包。 

3）App ClassLoader 负责加载classpath中指定的jar包及 Djava.class.path 所指定目录下的类和jar包。

4）Custom ClassLoader 通过java.lang.ClassLoader的子类自定义加载class，属于应用程序根据自身需要自定义的ClassLoa





## 总结

- 编译（.java -> .class）

  - 词法解析，语法解析，语义解析
  - 代码生成器，得到得到二级制文件.class：版本信息，常量池，字段表集合，方法表集合

- 装载

  - 过一个类的全限定名获取定义此类的二进制字节流

  - 将这个字节流所代表的静态存储结构转化为方法区的运行时数据结构，方法区：类信息，静态变量，常量

  - 在Java堆中生成一个代表这个类的java.lang.Class对象。 堆：代表被加载类的java.lang.Class对象

- 链接

  - 校验（校验程序合法性）
  - 准备（为所有数据初始化为0，对比C++，rodata在编译阶段，data在此阶段，但是java并不初始化，和bos一样）
  - 解析（为符号分配内存地址）

- 初始化

  - 使用时初始化

- 运行

  - 使用：使用时初始化
  - 卸载 
    - 该类所有的实例都已经被回收，也就是java堆中不存在该类的任何实例。 
    - 加载该类的ClassLoader已经被回收。 
    - 该类对应的java.lang.Class对象没有任何地方被引用，无法在任何地方通过反射访问该类的方 法。





























