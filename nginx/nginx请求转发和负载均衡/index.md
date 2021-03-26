# nginx 反向代理和负载均衡
## 反向代理
反向代理经常用于服务器集群中，用于隐藏真实服务器地址。
### 实例
目的是把 a.localhost.com 的请求转发给 192.168.2.101 服务器。  

修改 nginx/conf/vhost/a.localhost.com.conf  
(如果是直接访问ip则修改 nginx/conf/nginx.conf)
```
server {
	listen 80;
	server_name  a.localhost.com;

	# http://a.localhost.com:80 => http://192.168.2.101:80
	location / {
		# 要指向的服务器地址 可以写端口号和请求路径
		proxy_pass http://192.168.2.101:80;
	}
}
```

## 负载均衡
负载均衡用于把请求分发给不同的服务器。  
### 实例
目的是把 a.localhost.com 的请求转发给 192.168.2.101 192.168.2.102 192.168.2.103 服务器。  
nginx 负载均衡常用模式  

#### 1. 权重轮询  
平均分发给服务器。如果带有 weight 参数，根据参数比例分发。默认 weight 为 1。
```
upstream vm_server1 {
	# 分发给的服务器
	server 192.168.2.101:80 weight=2;
	server 192.168.2.102:80;
	server 192.168.2.103:80;
}
server {
	listen 80;
	server_name  a.localhost.com;
	
	location / {
		proxy_pass http://vm_server1;
	}
}
```

#### 2. ip_hash  
根据请求者 ip 分发，同 ip 一直都会访问同一个服务器。
```
upstream vm_server1 {
	# ip
	ip_hash;
	# 分发给的服务器
	server 192.168.2.101:80;
	server 192.168.2.102:80;
	server 192.168.2.103:80;
}
server {
	listen 80;
	server_name  a.localhost.com;
	
	location / {
		proxy_pass http://vm_server1;
	}
}
```


#### 3. 最少连接
自动分发给负载最低的服务器
```
upstream vm_server1 {
	# 最少连接
	least_conn;
	# 分发给的服务器
	server 192.168.2.101:80;
	server 192.168.2.102:80;
	server 192.168.2.103:80;
}
server {
	listen 80;
	server_name  a.localhost.com;
	
	location / {
		proxy_pass http://vm_server1;
	}
}
```
### 参考
https://docs.nginx.com/nginx/admin-guide/web-server/reverse-proxy/  
https://docs.nginx.com/nginx/admin-guide/load-balancer/http-load-balancer/  
