---
title: Nginx相关
date: 2018-06-06 21:32:24
categories: Java
tags:
  - 分布式
  - nginx
top_img: https://tva1.sinaimg.cn/large/006tNbRwgy1gash4k2x0uj31900u0b1g.jpg
---

## Nginx安装配置

https://www.runoob.com/linux/nginx-install-setup.html

**正向代理**：正向代理的对象是客户端	举例：访问Youtube挂VPN

**反向代理**：反向代理的对象是服务器	举例：各大电商平台的服务器

**常用的web服务器**：Apache，Nginx，Tomcat，Jetty，Netty，Jboss，iis等

**nginx的常用命令**

- ./nginx   启动
- ./nginx -t  检查nginx.conf文件格式
- ./nginx -s stop  停止
- ./nginx -s reload  重启nginx

<!-- more -->

## nginx的核心配置文件

main、events、http

**nginx.conf**

```xml
worker_processes  2; #工作进程数
error_log  /usr/local/webserver/nginx/logs/nginx_error.log warn;
worker_rlimit_nofile 10240; #最大打开文件的限制
events {
    worker_connections  10240; #不能超过上面的数字
}
http {
    include       mime.types; #请求头的映射关系表
    default_type  application/octet-stream; #上面找不到的话会用这个
    sendfile        on; #用来开启高效传输的模式
    keepalive_timeout  65; #http连接的超时时间

    include /etc/nginx/conf.d/*.conf;
}
```

**proxy.conf**

```xml
server{
    listen 80;
    server_name  localhost;
    location / {
        root   html;
        index  index.html index.htm;
    }
}
server{
    listen 8080;
    server_name  sjaylee;
    location / {
        proxy_pass http://mysvr;
        root   html;
        index  index.html index.htm;
    }
}
```

**upstream.conf**

```xml
upstream.conf
upstream mysvr{
    server 10.211.55.4:8081;
    server 10.211.55.4:8082;
}
```

## location的匹配规则

基础语法有三种：

- location pattern {}	一般匹配
- location = pattern {}	精准匹配
- location ~pattern {}	正则匹配

**nginx的负载均衡和负载均衡的调度算法**

upstream：ip_hash、轮询(默认)、weight、fair、url_hash。

ip_hash算法是对服务器的IP地址进行hash运算来获取对应代理服务器，好处是不需要考虑session跨域。

weight算法是根据权重来合理分配各个代理服务器的调用比例。

## nginx日志切分

第一步：分析如何去实现日志切分，编写shell脚本，记得要给脚本赋予可执行权限。

第二步：启动定时任务对脚本进行调度：crontab -e

定时任务格式：

```bash
*/1 * * * * sh /usr/local/webserver/nginx/logs/access.log
```

第三步：新增split_log.sh文件，并给其增加权限，chmod 777 文件名

**split_log.sh**

![](https://tva1.sinaimg.cn/large/006tNbRwgy1garbdepmokj30ik0mm4ev.jpg)