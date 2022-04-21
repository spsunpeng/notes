#### zookeeper

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























