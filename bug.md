# 翻车日记

### 1、2022-05-11 system服务启动失败

mallbbg2sit环境system服务启动失败，infor日志下无任何报错：排查过程



#### 1.1 对比分析

- 测试

  dev环境可以启动成功

  sit环境上个版本代码也部署失败

- 分析

  猜测是sit环境造成



#### 1.2 分析k8s和其他

- 测试

  k8s节点部署详情、查看部署文档、修改的代码

- 分析

  没有结果（并且因为不熟悉服务监听网络端口：8080/9999，导致走了好多弯路）



#### 1.3 临时修改

- 测试

  删除deploy文档中的探针（每隔60s监测8080端口）

- 结果

  服务日志打印完后间隔10分钟才正常使用

- 分析

  当时分析：认为8080端口有问题，隔了10分钟才监听上

  事后分析：服务10分钟之后才启动成功（对探针而言它代表60内没有监听到8080，表示任务服务没起来），那么这十分钟里他一直在做什么



#### 1.4 将日志级别改为debug

- 测试

  一顿胡思乱想后，终于将日志级别改为debug

- 结果

  sit环境：不断打印rabbit 和xxx-job相关日志

  dev环境：没有此现象，且dev环境关于declaring queue(断言队列)明显和sit环境不一致

  sit环境rabbit：有个队列bbc_admin_queue的状态和其它队列不一致

- 分析

  sit环境的队列bbc_admin_queue状态有问题，导致此队列的断言一直通不过，十分钟后系统才跳过



#### 1.5 事后分析

- 服务注册到nacos才算启动成功
- 10分钟才启动成功是因为服务重复declaring queue

- 没有认清服务启动成功的本质导致方向错误
- 没有仔细查看debug日志导致分析不出问题





### 2、zsa-adapter服务启动失败

zsa-adapter本地可以起成功，但是部署到k8s中启动失败，并且pod直接销毁，无法查看日志。

#### 1.1 事后分析

- k8s中static先于bean加载，而本地bean先于static加载
- 服务一开始就加载失败，pod销毁，所以看不了日志。（应该有办法让pod存货一段时间）

#### 1.2 分析日志

pod销毁无法查看日志，但物理机上日志还是存在的

```sh
#查看部署文件deploy得知：日子存放方式是hostPath(持久化但不共享)，所以需要链接对应的物理机。
kubectl get pods mallbbcg2-system-service-78c6cd544b-dgs4g -n mallbbcg2sit -o wide 
#得到system部署的主机名：xa-dev-k8s-n02
#方法1.公司服务器连公司服务器，主机名肯定可以自动映射；方法2.本地连公司服务器，需要先查到ip
ssh root@xa-dev-k8s-n02 #密码sinosun 
cd /usr/local/SINO/LOG/mallbbcg2sit
vim servicelog-system-service.log
```

#### 1.3 解决

没有办法控制static 和bean的加载顺序，最后用其他方式解决



















