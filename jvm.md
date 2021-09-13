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