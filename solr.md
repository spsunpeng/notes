# 概述

## 1、使用

```sh
###solr使用
./solr start -force #启动
#访问：10.1.20.122:8983


```

## 2、目前

目前只安装了solr，后续有需要再学习



# 一、solr

## 1、安装

```sh
###1.资源
#本机资源在/usr/local/tmp/solr-7.7.2.tgz中，也可另寻资源
tar zxf solr-7.7.2.tgz
cp -r solr-7.7.2 ../solr

###2.安装
cd /usr/local/solr/bin
vim solr.in.sh  
	SOLR_ULIMIT_CHECKS=false	#修改启动参数，否则启动时报警告。提示设置SOLR_ULIMIT_CHECKS=false

###3.启动
./solr start -force 
#solr默认不推荐root账户启动，如果是root账户启动需要添加-force参数
#Solr内嵌Jetty，直接启动即可。监听8983端口


###4、使用
#访问：10.1.20.122:8983
```

