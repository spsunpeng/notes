### 一、 查看文件

#### 1、查看文件

| 命令 | 解释                   | 常用                    | 解释             |
| ---- | ---------------------- | ----------------------- | ---------------- |
| cat  | 从第一行开始显示       | **cat filename**        | -n可以显示行号   |
| tac  | 从最后一行开始显示     |                         |                  |
| more | 一页一页的显示档案内容 |                         | d：向前翻页      |
| less | 相比more，可以翻页     | shift+g 移动到最后一行  | b：向后翻页      |
| head | 只看头几行             | head-n 20 filename      | 头10页，默认为10 |
| tail | 只看尾巴几行           | **tail -n 20 filename** | 尾10页，默认为10 |

#### 2、 cat

1. 命令格式：

   cat [选项] [文件]...

2. 命令功能：

   cat主要有三大功能：

   1. 一次显示整个文件: cat filename

   2. 从键盘创建一个文件: cat > filename 只能创建新文件,不能编辑已有文件.

   3. 将几个文件合并为一个文件: cat file1 file2 > file

3. 命令参数：

   -n, --number   对输出的所有行编号,由1开始对所有输出的行数编号

   

### 二、过滤 grep

#### 1、使用方法

grep  -参数  "字段"  filename

- 参数

  - -E:  或
  
  - -r ：进入目录搜索
  
  - -n :  显示行号
  - -A/B/C  number 显示匹配内容以及后/前/前后几行内容
- 搜索条件

  - "word" ： 搜索含有word的，支持汉字、特殊字符、空格

  - 支持部分正则表达式 "getDictData.*7," 搜索含有 getDictData和7

  - “word1|word2” ：参数没有-E则|就是原意，若有则表示或

- fileName：可以是文件，也可以是目录(-r)

#### 2、示例

- grep “word” fliename                                   含有word
- grep “汉字| :word” fliename                       含有汉字| :word
- grep -E “word1|word2” fliename               含有word1或word1

- grep "word1.*word2" fliename                  含有word1和word2（有word1...word2）
- grep “word1” fliename grep "word2"        含有word1和word2（有word2的基础上有word1）

- grep -C 5 "word" filename                           含有word以及他前后5行的



### 三、vi

1）上下翻动：向上箭头、向下箭头：每次滚动一条日志，这条日这可能占据多行

2）前后翻页：ctrl+f、ctrl+b：f就是forword，b就是backward

3）首行末行：首行（gg），末行（shift+g）

4）查找：?string：查找字符串，N向前，n向后

5）分页分屏：k8s中的vi也支持分页/分屏操作





### 四、命令

#### 1、linux基本命令

```sh
#基本命令
mv {source} {target}
cp {source} {target}
mkdir {目录}
rm {文件}
rm -ef {目录}

#软件
yum [options] [command] [package ...]
1. 列出所有可更新的软件清单命令：yum check-update
2. 更新所有软件命令：yum update
3. 仅安装指定的软件命令：yum install <package_name>
4. 仅更新指定的软件命令：yum update <package_name>
5. 列出所有可安裝的软件清单命令：yum list
6. 删除软件包命令：yum remove <package_name>
7. 查找软件包命令：yum search <keyword>

wegt {url} #下载
tar -zxf {.tar压缩包} #解压

#进程
ps -ef    #查询进程
jmap -heap pid   #查询堆内存

#网络
ip addr   #查询ip
ifconfig  #查询ip

#其他
uname -v  #查询虚拟机内核版本

#启停防火墙
systemctl status firewalld  # 查看状态
systemctl start firewalld # 开启
systemctl stop firewalld # 关闭 
systemctl is-enabled firewalld # 查看服务是否开机启动
systemctl disable firewalld # 开机禁用
systemctl enable firewalld # 开机启用

#网络配置
cd /etc/sysconfig/network-scripts #网络配置文件位置
vim ifcfg-ens33 #网络配置文件
nmcli c reload #重启网卡
```

#### 2、window基本命令

```sh
ipconfig  #查询ip
ping {ip}
tcping {ip} {port}
```





### 五、环境搭建docker

#### 1.虚拟机安装

- VMware：好像需要破解
- VirtualBox6.3：好像开源，Oracle的，官网下载

#### 2.linux安装

- CentOS-8.0：好像开源，下载地址：http://mirrors.163.com/【网易镜像地址】

  安装：

  - 虚拟机新建操作系统： 新建操作系统 -》设置内存、存储。
  - 加载：选择要加载的镜像 -》加载
  - 操作系统配置：选择存储、root用户设置 -》重启
  - 个性化设置：等等。
  - 网络设置：1.网络设置； 2.网络配置文件修改

#### 3.docker安装

##### 3.1 docker安装

```sh
yum install -y yum-utils device-mapper-persistent-data lvm2 wget
wget -O /etc/yum.repos.d/docker-ce.repo https://download.docker.com/linux/centos/docker-ce.repo
sed -i 's+download.docker.com+mirrors.tuna.tsinghua.edu.cn/docker-ce+' /etc/yum.repos.d/docker-ce.repo
yum -y makecache fast
yum -y install docker-ce-18.09.9
```

##### 3.2 docker启动

```sh
#启动停止虚拟机命令
systemctl start docker
systemctl enable docker
systemctl stop docker

#关闭防火墙
service firewalld status #查看防火墙状态
service firewalld stop #关闭防火墙
```

#### 4.docker容器

##### 4.1.docker配置阿里镜像

- 获取阿里云的容器配置

  - 登录阿里云：https://cr.console.aliyun.com/cn-hangzhou/instances

  - 获取阿里云在docker中配置文件：

    ```json
    {
      "registry-mirrors": ["https://h45068lf.mirror.aliyuncs.com"]
    }
    ```

- 在虚拟集中配置

  - /etc/docker/daemon.json

    ```json
    {
      "registry-mirrors": ["https://h45068lf.mirror.aliyuncs.com"]
    }
    ```

    或者直接执行

    ```sh
    tee /etc/docker/daemon.json <<-'EOF'
    {
      "registry-mirrors": ["https://registry.docker-cn.com"]
    }d
    EOF
    ```

  - 重启docker

    sudo systemctl daemon-reload
    sudo systemctl restart docker

##### 4.2 拉去镜像

```sh
docker search 关键字 #查询可以镜像 eg：docker search mysql
docker pull 镜像名:tag   #拉取镜像 eg：docker pull mysql:5.7
docker images  #查看所有本地镜像
docker rmi id #删除指定的本地镜像
```

##### 4.3 启动容器

```sh
#创建容器
docker run -d --name mytomcat -p 8888:8080 tomcat #新建并启动容器
docker ps #查询容器
docker rm  容器名或id #删除容器
#启动容器
docker start  容器名或id #开启容器
docker ps -a #查询正在运行的容器
docker stop  容器名或id #关闭容器
docker restart  容器名或id #重启容器
#修改容器配置
docker exec -it 容器名或id bash #进入容器
exit #退出
```

#### 5. centos测速

```sh
#安装git
yum -y install git
#git克隆speedtest
git clone https://github.com/sivel/speedtest-cli.git
#进入speedtest-cli
cd speedtest-cli
#修改speedtest.py，由于现在一般python都是python3，所以#!/usr/bin/env python -》 #!/usr/bin/env python3
vi speedtest.py
#执行speedtest-cli
./speedtest-cli
```

可能报错：./speedtest-cli执行失败，没有python

```sh
#检查python版本
python -version
python3 -version
#安装python3
dnf install python3
```

设置全局

```sh
#增加读写执行权限及移动到全局操作目录
chmod +rx speedtest.py
sudo mv speedtest.py /usr/local/bin/speedtest-cli
sudo chown root:root /usr/local/bin/speedtest-cli
speedtest.py #任意位置执行
```



### 六、其他软件安装

#### 1、jdk

```sh
#查看是否安装、安装版本
java -version

#安装
yum install java-1.8.0-openjdk* -y #yum安装方法会自动配置环境变量
#默认安装位置 /usr/lib/jvm
```

#### 2、zookeeper

zookeeper官网：http://zookeeper.apache.org/

##### 2.1 下载

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

##### 2.2 使用

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

##### 2.3 查看数据

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



