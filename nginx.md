nginx 2004年 俄罗斯人 2019年被硬件厂商F5收购



安装

```sh
#=============================================安装====================================================
yum install yum-utils
#配置nginx的下载源
vim /etc/yum.repos.d/nginx.repo
<--
[nginx-stable]
name=nginx stable repo
baseurl=http://nginx.org/packages/centos/$releasever/$basearch/
gpgcheck=1
enabled=1
gpgkey=https://nginx.org/keys/nginx_signing.key
module_hotfixes=true
[nginx-mainline]
name=nginx mainline repo
baseurl=http://nginx.org/packages/mainline/centos/$releasever/$basearch/
gpgcheck=1
enabled=0
gpgkey=https://nginx.org/keys/nginx_signing.key
module_hotfixes=true
-->
yum install nginx
nginx -v #nginx/1.22.0


#=============================================启动====================================================
cd /usr/sbin
./nginx #启动
./nginx -s stop #停止，kill命令也可以停止
./nginx -s reload #重新加载配置文件
#验证
ps -ef | grep nginx 
curl localhost:80   #虚拟你访问nginx：Welcome to nginx!
http://10.1.20.236/ #宿主机访问nginx: Welcome to nginx!



cat /var/run/nginx.pid #查看nginx的pid
```



nginx命令位置：/usr/sbin/nginx

nginx配置文件位置：/etc/nginx/nginx.conf



nginx: [warn] conflicting server name "localhost" on 0.0.0.0:80, ignored



```conf
    server{
        listen    81;
	server_name    localhost;
        location / {
	    proxy_pass http://localhost:8080;
        }    
    }
#    server {
#	    listen       80;      
#            listen       443 ssl;
#            server_name  www.baidu.com;
#            ssl_certificate      /data/cert/server.crt;
#            ssl_certificate_key  /data/cert/server.key;
#     }
```



tomcat /usr/share/tomcat/webapps/ROOT





负载均衡

```conf
# server list 
upstream myServers { 
    #random two least_conn; #负载均衡算法：随机，默认（不配置）轮询
    server localhost:8081; 
    server localhost:8082; 
}
server { 
    listen 9002; 
    server_name www.cpf.com; 
    location / { 
        proxy_pass http://myServers; 
    } 
}

```

负载均衡算法

least_conn;
ip_hash;
hash $request_uri consistent;
random two least_conn; #随机



















