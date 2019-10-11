# Nginx

 负载均衡 请求转发 停止&开始


## start nginx.exe && nginx.exe -s stop

## 请求装发 proxy_pass
```xml
server {
        listen       80;         ## 监听端口
        server_name  www.abc.cn; ## 服务名称
        location / {
            proxy_pass http://101.91.127.179:8082; ##访问路径
            index  index.html index.htm;
        }
```


## 负载均衡作用
>* 是为了解决高并发，负载均衡器拦截到左右请求，在采用负载均衡算法分配到不同的真实服务器上
## Nginx 配置负载均衡
>* Upstream Server 上游服务器（表示 使用负载均衡器访问真实服务器）
>* Nginx 内置核心功能 
>故障转移 
>重试机制
>心跳检测
>* 负载均衡配置 轮训

```xml
## 定义上游服务器及 负载均衡器
    upstream backServer{
        server 42.123.80.182:8090 weight=1;
        server 101.91.127.179:8083 weight=2;
    }
    server {
        listen       80;
        server_name  www.abc.cn;

        #charset koi8-r;

        #access_log  logs/host.access.log  main;

        location / {
            ##root   html;
            proxy_pass http://backServer/;
            index  index.html index.htm;
        }

      
        error_page   500 502 503 504  /50x.html;
        location = /50x.html {
            root   html;
        }

    }
```
>* 负载均衡配置 ip绑定
```xml
## IP绑定
    upstream backServer{
        server 42.123.80.182:8090;
        server 101.91.127.179:8083;
        ip_hash;
    }
    server {
        listen       80;
        server_name  www.abc.cn;

        #charset koi8-r;

        #access_log  logs/host.access.log  main;

        location / {
            ##root   html;
            proxy_pass http://backServer/;
            index  index.html index.htm;
        }

      
        error_page   500 502 503 504  /50x.html;
        location = /50x.html {
            root   html;
        }

    }
```

>* 故障转移
```xml
    server {
        listen       80;
        server_name  www.abc.cn;

        #charset koi8-r;
    
        #access_log  logs/host.access.log  main;
    
        location / {
            ##root   html;
            proxy_pass http://backServer/;
            
            ## nginx与真实服务器超时时间 后端服务器链接的超时时间 发起捂手等候响应超时时间
            proxy_connect_timeout 1s;
            ##nginx发送给正式服务器超时时间
            proxy_send_timeout 1s;
            ##nginx接受真实服务器超时时间
            proxy_read_timeout 1s;
            
            index  index.html index.htm;
        }
```
## Rewrite全局变量
变量|含义
--|:--:|
$args    |这个变量等于请求行中的参数，同$query_string
$content length|    请求头中的Content-length字段。
$content_type    |请求头中的Content-Type字段。
$document_root|    当前请求在root指令中指定的值。
$host|    请求主机头字段，否则为服务器名称。
$http_user_agent|    客户端agent信息
$http_cookie|    客户端cookie信息
$limit_rate    |这个变量可以限制连接速率。
$request_method|    客户端请求的动作，通常为GET或POST。
$remote_addr    |客户端的IP地址。
$remote_port    |客户端的端口。
$remote_user    |已经经过Auth Basic Module验证的用户名。
$request_filename|    当前请求的文件路径，由root或alias指令与URI请求生成。
$scheme    |HTTP方法（如http，https）。
$server_protocol|    请求使用的协议，通常是HTTP/1.0或HTTP/1.1。
$server_addr|    服务器地址，在完成一次系统调用后可以确定这个值。
$server_name|    服务器名称。
$server_port|    请求到达服务器的端口号。
$request_uri|    包含请求参数的原始URI，不包含主机名，如”/foo/bar.php?arg=baz”。
$uri|    不带请求参数的当前URI，$uri不包含主机名，如”/foo/bar.html”。
$document_uri|    与$uri相同。
