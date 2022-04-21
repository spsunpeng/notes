

# 当前

```sh
#========================================版本=================================================
docker -version #19.03.12
docker-compose -version #1.29.2
kubectl version #1.14.1

#=========================================启动==============================================
#docker
systemctl status docker
systemctl start docker
#k8s
systemctl status kubelet
systemctl start kubelet
#nfs：共享数据
systemctl start nfs.service #master
systemctl start rpcbind.service #master
mount 10.1.20.235:/usr/local/data/www-data /mnt #node
#rinetd：端口转发
netstat -tulpn | grep rinetd #查看映射，具体产看看三.2
```



# 一、Centos

- 新建虚拟机
  - 选择系统安装位置，系统类型，系统版本：red hat(64-bit)
  - 选择内存大小
  - 现在创建虚拟硬盘，文件类型选择VDI，动态分配，虚拟硬盘位置不要选在C盘

- 配置安装
  - 存储 -》没有盘片 -》分配光驱 -》选择本地的iso
  - 系统 -》启动顺序 -》把光驱排在第一个

- 安装
  - 确定安装，安装语言选择中文（这里设置的仅仅是安装语言）
  - 安装位置 -》 本地标准盘 -》 自动分配分区 -> 完成 -》 开始安装
  - root密码 -》 root:root -》完成 -》重启

```sh
#网络：桥接模式
#网卡:开机启动，将动态获取ip改为静态获取
ip addr #10.1.20.89
vi /etc/sysconfig/network-scripts/ifcfg-enp0s3
TYPE=Ethernet 						
	PROXY_METHOD=none
	BROWSER_ONLY=no
	BOOTPROTO=static
	IPADDR=10.1.20.89
	NETMASK=255.255.255.0
	GATEWAY=10.1.20.1
	DNS1=114.114.114.114
	DEFROUTE=yes
	IPV4_FAILURE_FATAL=no
	IPV6INIT=yes
	IPV6_AUTOCONF=yes
	IPV6_DEFROUTE=yes
	IPV6_FAILURE_FATAL=no
	IPV6_ADDR_GEN_MODE=stable-privacy
	NAME=enp0s3                           
	UUID=b351fca8-3596-4168-a35f-cc927c09e2c4
	DEVICE=enp0s3
	ONBOOT=yes                       
nmcli connection reload #centos7重载网卡
systemctl restart network #centos8重载网卡

#关闭防火墙
systmectl stop firewalld    # 关闭
systmectl disable firewalld # 关闭开机启用

#下载vim
yum -y install vim*

#版本
cat /etc/redhat-release #CentOS Linux release 7.9.2009 (Core)
uname -r  #3.10.0-1160.el7.x86_64
#内存：2g
#存储：50g
#网络：桥接模式

vim ~/.bashrc
	PS1='\[\e[35;40m\][\u@\h \w]\$: \[\e[m\]'
		\d ：#代表日期，格式为weekday month date，例如："Mon Aug 1"
		\H ：#完整的主机名称
		\h ：#仅取主机的第一个名字
		\t ：#显示时间为24小时格式，如：HH：MM：SS
		\T ：#显示时间为12小时格式
		\A ：#显示时间为24小时格式：HH：MM
		\u ：#当前用户的账号名称
		\v ：#BASH的版本信息
		\w ：#完整的工作目录名称
		\W ：#利用basename取得工作目录名称，所以只会列出最后一个目录
		\# ：#下达的第几个命令
		\$ ：#提示字符，如果是root时，提示符为：# ，普通用户则为：$
		\[\e[35;40m\]: 颜色，其中“F“为字体颜色，编号为30-37，“B”为背景颜色，编号为40-47
		\[\e[m\]：后面不设置
source /root/.bashrc
```

- 导出虚拟机

  - 选择模板（centos7）

  - 开放式虚拟化格式1.0，导出位置，仅包含NAT网卡的MAC地址，写入Manifest地址
  - 导出成ova格式

- 导入虚拟机

  - 选择系统和存储的安装路径
  - 重命名虚拟机

- 设置静态网络
  - 将动态ip改为静态或为静态ip重新选个ip

```sh
timedatectl set-timezone Asia/Shanghai
hostnamectl set-hostname [new-hostname] #vim /etc/hostname
vim /etc/hosts
	10.1.20.235  master
	10.1.20.236  node1
	10.1.20.237  node2
reboot #重启虚拟机
```





# 二、docker

https://docs.docker.com/

https://hub.docker.com/search

## 1、安装

```sh
# 安装
yum install -y yum-utils device-mapper-persistent-data lvm2 wget
wget -O /etc/yum.repos.d/docker-ce.repo https://download.docker.com/linux/centos/docker-ce.repo
sed -i 's+download.docker.com+mirrors.tuna.tsinghua.edu.cn/docker-ce+' /etc/yum.repos.d/docker-ce.repo
yum -y makecache fast
yum -y install docker-ce-18.09.9
docker -version

# 配置阿里镜像
vim /etc/docker/daemon.json
	{
  		"registry-mirrors": ["https://h45068lf.mirror.aliyuncs.com"]
	}
systemctl daemon-reload #重启
systemctl restart docker #重启

#启动
systemctl start docker #启动
systemctl stop docker  #停止
systemctl status docker #状态
systemctl enable docker #自启动
systemctl disable docker #取消自启动
systemctl is-enabled docker #是否自启动

#docker信息
/var/lib/docker/image #镜像位置
/var/lib/docker/containers #容器位置
docker -version #19.03.12
docker-compose -version #1.29.2
```



## 2、docker命令

```sh
#==========================================一、镜像===============================================
docker search [keyword] #搜索镜像 eg：docker search mysql
docker pull [image]:[tag]   #拉取镜像 eg：docker pull mysql:5.7
docker images  #查
docker rmi id  #删除
docker build -t [image-name] [image-source-path] #构建镜像，-t:重命名


#==========================================二、容器===============================================
#创建并运行容器
docker run -d --name m1 -p 3307:3306 -e MYSQL_ROOT_PASSWORD=root mysql:5.7
	-d: 后台运行容器，并返回容器ID；
	-p: 指定端口映射，格式为：主机(宿主)端口:容器端口
	--name=[name]: 为容器指定一个名称；
	-e [环境变量key]=[环境变量value]: 设置环境变量；
docker logs [docker-id|docker-name] #查看容器日志

#增删
docker create [image]:[tag] #创建容器，一般都被run替代
docker ps -a #查询容器，-a:所有镜像，包括没有运行的镜像
docker rm -f id #删除容器，-f:强制删除容器（删除正在运行的容器）

#启停
docker start [docker-id|docker-name] #开启容器
docker stop [docker-id|docker-name] #关闭容器
docker restart [docker-id|docker-name] #重启容器
docker ps #查询正在运行的容器
docker inspect [id|name] #查询容器详情，比如 port, ip

#进入容器
docker exec -it [docker-id|docker-name] /bin/bash #进入容器。-it:以交互的模式进入
#tomcat404原因可能版本问题，需要将webapps.dist下的内容全部复制到webapps中



#==========================================三、网络===============================================
#网络：docker会为每个容器分配虚拟地址，容器间网络本身就互通，但是由于容器创建销毁频繁简单，虚拟ip不断变化，所以希望通过容器名联通网络，所以就需要网桥
docker network ls
docker network  create --driver bridge [new-bridge-name]#创建网桥
docker run -d --name [new-docker-name] -p 8888:8080 --network [bridge-name] tomcat
#测试：进入容器 ping 其他容器的名字


#==========================================四、共享地址===============================================
#容器直接映射地址
docker run -p 8000:8080 --name v1 -v /usr/local/dockershare:/usr/local/tomcat/webapps tomcat 
#通过共享容器映射地址
docker create --name webpage -v /usr/local/dockershare:/usr/local/tomcat/webapps tomcat /bin/true 
docker run --name t3 -d --volumes-from webpage -p 8005:8080 tomcat
```



## 3、dockerfile

用户构建镜像：docker build -t [image-name] [image-source-path]，此命令执行时会执行Dockerfile文件

```Dockerfile
#==========================================一、命令===============================================
FROM tomcat:latest #制作基准镜像
WORKDIR /usr/local/tomcat/webapps #切换工作目录，没有就会创建
ADD docker-web ./docker-web #ADD&COPY 将宿主机文件复制到容器中，当前容器路径是（./）就是WORKDIR目录
ENV [] #设置环境变量
EXPOSE [port] #暴露端口，现有程序的默认端口
RUN [shell] #构建镜像时执行shell命令
ENTRYPOINT [shell] #容器启动时执行shell命令，只会执行最后一个，且不会被命令行替换
CMD [shell]  #容器启动时执行shell命令，只会执行最后一个，但会被命令行替换

#=========================================二、示例：redis=============================================
FROM centos
RUN ["yum" , "install" , "-y" ,"gcc","gcc-c++","net-tools","make"]
WORKDIR /usr/local
ADD redis-4.0.14.tar.gz .
WORKDIR /usr/local/redis-4.0.14/src
RUN make && make install
WORKDIR /usr/local/redis-4.0.14
ADD redis-7000.conf .
EXPOSE 7000
CMD ["redis-server","redis-7000.conf"]
```



## 4、docker-compose

微服务有多个镜像和容器，Dockerfile是启动配置，compose是自动化部署

- app-Dockerfile

```Dockerfile
FROM openjdk:8u222-jre
WORKDIR /usr/local/bsbdj
ADD bsbdj.jar .
ADD application.yml .
ADD application-dev.yml .
EXPOSE 80 #暴露端口
CMD ["java","-jar","bsbdj.jar"]
```

- db-Dockerfile

```Dockerfile
FROM mysql:5.7
WORKDIR /docker-entrypoint-initdb.d
ADD init-db.sql .
```

- docker-compose.yaml

```yaml
version: '3.3'
services:
  db:
    build: ./bsbdj-db/ #构建镜像
    restart: always    #容器挂了就重启
    environment:
      MYSQL_ROOT_PASSWORD: root #容器启动时的环境变量
  app:
    build: ./bsbdj-app/
    depends_on:       #依赖的容器，让app可以访问db
      - db
    ports:
      - "80:80"
    restart: always
```

```sh
cd /usr/local/demo/bsbdj
docker-compose up -d #运行
docker-compose logs [docker-id|docker-name] #查看日志
docker-compose down #下线
```





# 三、k8s

容器编排工具

- 自动化容器的部署和复制
- 随时扩展或收缩容器的规模
- 容器分组group，并且提供容器间的负载均衡
- 实时监控，即使故障发现，自动替换

名词

- pod pause 数据卷
- container 容器
- label 标签
- replication controller 复制控制器
- service 服务
- node 节点

## 1、安装

```sh
#=============================================1.准备===================================================
#1.1关闭交换区
swapoff -a #查看交换分区
vi /etc/fstab 
   #swap 注释
#1.2 配置网桥
cat <<EOF >  /etc/sysctl.d/k8s.conf
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
EOF
sysctl --system #查看

#====================================2.安装 kubelet kubeadm kubectl======================================
#kubeadm：容器集群部署工具，kubectl：命令
yum install -y kubelet-1.14.1 kubeadm-1.14.1 kubectl-1.14.1 
rpm -qa | grep kube #查看
systemctl start kubelet
systemctl enable kubelet

#====================================3.docker拉去k8s镜像=================================================
coredns-1-3-1.tar  kube-apiserver-1-15.tar   kube-proxy-1-15.tar      myflannel-11-0.tar
etcd-3-3-10.tar    kube-controller-1-15.tar  kube-scheduler-1-15.tar  pause-3-1.tar


#====================================4.构建集群：master==================================================
#构建集群
kubeadm init --kubernetes-version=v1.14.1 --pod-network-cidr=10.244.0.0/16 --ignore-preflight-errors=NumCPU
#--ignore-preflight-errors=NumCPU：忽略内存限制，不报错可以不加
#执行kubeadm init打印出的命令
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config

#安装flannel网络组件
kubectl create -f kube-flannel.yml #安装flannel网络组件
kubectl get pod --all-namespaces #查看

#开启dashboard
kubectl apply -f kubernetes-dashboard.yaml
kubectl apply -f admin-role.yaml
kubectl apply -f kubernetes-dashboard-admin.rbac.yaml
kubectl -n kube-system get svc #查看
#访问http://192.168.163.132:32000 

#===========================================5.加入集群：node=============================================
#执行master节点构建集权式的输出
kubeadm join 10.1.20.235:6443 --token f5jryy.c9uut9rxhokzlj9o \
    --discovery-token-ca-cert-hash sha256:ed4005bc79fd7be8d7d53eb06fd00e1833d79b127d8e06bfaa080ef1a148282e 
#如果忘记，在master上执行查看，再在node中加入
kubeadm token list #master执行：查看
kubeadm join 192.168.163.132:6443 --token aoeout.9k0ybvrfy09q1jf6 --discovery-token-unsafe-skip-ca-verification #node执行：无token方式加入集群

```



## 2、kubectl

```sh
#=============================================namespace==============================================
kubectl get ns

#=============================================node==============================================
kubectl get nodes

#=============================================deploy==============================================
kubectl get deploy
kubectl create -f [deploy.yml]
kubectl apply -f [deploy.yml]
kubectl delete deploy [deploy-name]
kubectl describe deploy [deploy-name]

#=============================================pod==============================================
kubectl get pod --all-namespaces
kubectl get pod -n [namespace]
kubectl get pod -o wide
kubectl describe pod [pod-name]
kubectl logs [pod-name] #容器启动日志
kubectl exec -it [pod-name] #进入容器

#=============================================service==============================================
kubectl get service
kubectl create -f [service.yml]
kubectl delete service [service-name]
kubectl describe service [service-name]
curl [service-ip]:[service-port]/test/index.jsp #使用service的ip:port范文命令

#=============================================nfs 共享数据==============================================
#master安装
yum install -y nfs-utils rpcbind
mkdir data
cd data
mkdir www-data
vim /etc/exports
	/usr/local/data/www-data 10.1.20.235/24(rw,sync)
systemctl start nfs.service
systemctl start rpcbind.service
systemctl enable nfs.service
systemctl enable rpcbind.service
exportfs #验证

#node安装
yum install nfs-utils
showmount -e 10.1.20.235 #查询
#并未拉去，只是像本地一样访问远程，实际访问中还是要用网络
mount 10.1.20.235:/usr/local/data/www-data /mnt #关机需要重新执行
#测试：查看/mnt下文件

#启动/重启
systemctl start nfs.service #master
systemctl start rpcbind.service #master
mount 10.1.20.235:/usr/local/data/www-data /mnt #node

#结合k8s
kubectl create -f [yml]
kubectl exec -it [pod-name] #查看挂在文件

#=============================================rinetd 端口转发==========================================
#安装
yum install -y gcc #环境，有gcc可以不执行
cd /usr/local
tar -zxvf 
cd rinetd
sed -i 's/65536/65535/g' rinetd.c
mkdir -p /usr/man
make && make install

#映射
vim /etc/rinetd.conf
	0.0.0.0 8000 10.104.210.162 8000
rinetd -c /etc/rinetd.conf #执行映射配置
netstat -tulpn | grep rinetd #查看映射
#当映射文件更新时，再此执行rinetd -c /etc/rinetd.conf只会新建映射，可能造成端口重复，所有需要删除原来的
ps ax | grep rinetd #查看映射的进程
kill -9 [pid] #删除

#测试
kubectl create -f [yml]
#http://[ip]:[serivice-port]/test/index.jsp 
```

## 3、deploy

```yaml
#基本
apiVersion: extensions/v1beta1
kind: Deployment
metadata:
  name: tomcat-deploy
spec:
  replicas: 2
  template:
    metadata:
      labels:
        app: tomcat-cluster-pod
    spec:
      containers:
      - name: tomcat-cluster-container
        image: tomcat
        ports:
        - containerPort: 8080

#基本信息 元数据 容器 共享数据 内存限制
apiVersion: extensions/v1beta1
kind: Deployment
metadata:
  name: tomcat-deploy
spec: 
  replicas: 2
  template:
    metadata:
      labels:
        app: tomcat-cluster-pod
    spec: 
      volumes:
      - name: web-app
        hostPath:
          path: /mnt
      containers:
      - name: tomcat-cluster-container
        image: tomcat
        resources:
          requests:
            cpu: 0.5
            memory: 200Mi
          limits:
            cpu: 1
            memory: 512Mi
        ports:
        - containerPort: 8080
        volumeMounts:
        - name: web-app
          mountPath: /usr/local/tomcat/webapps
```

## 4、service

```yml
apiVersion: v1
kind: Service
metadata:
  name: tomcat-service
  labels:
    app: tomcat-service   
spec:
  type: NodePort
  selector:
    app: tomcat-cluster-pod
  ports:
  - port: 8000
    targetPort: 8080
    nodePort: 32500
```





# 四、jenkins





# 五、devops





















