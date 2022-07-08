---
layout:     post
title:      "nginx常用配置"
subtitle:   "nginx配置反向代理、负载均衡、https服务、静态资源服务器、防盗链"
date:       2022-06-08 12:00:00
author:     "LiuYang"
catalog:    false
header-style: text
tags:
  - nginx
---

## Nginx简介

>Nginx (engine x) 是一个高性能的HTTP和反向代理web服务器，同时也提供了IMAP/POP3/SMTP服务



## 一、反向代理
>反向代理（Reverse Proxy）方式是指以代理服务器来接受Internet上的连接请求，然后将请求转发给内部网络上的服务器，并将从服务器上得到的结果返回给Internet上请求连接的客户端，此时代理服务器对外就表现为一个服务器。

<img style="border-radius: 20px;" src="/img/in-post/nginx/fxdl.png"  alt="反向代理" width="600px" >

* 例：使用反向代理http://liuyang1.com:8080 直接跳转到 http://localhost:9527/，配置如下：
```
server {
    listen       8080;
    server_name  liuyang1.com;
    location / {
        #代理服务器地址
        proxy_pass http://localhost:9527/;
    } 
}
```


## 二、负载均衡
>负载均衡，英文名称为Load Balance，其含义就是指将负载（工作任务）进行平衡、分摊到多个操作单元上进行运行，例如FTP服务器、Web服务器、企业核心应用服务器和其它主要任务服务器等，从而协同完成工作任务。
>如果其中一个节点挂了，nginx会自动转发到其他正常节点，使服务高可用。

**负载均衡策略**

* 1、轮询【默认】：
    每个请求按时间顺序逐一分配到不同的后端服务器，如果后端服务器down掉，能自动剔除
* 2、指定权重：
    指定轮询几率，weight和访问比率成正比，用于后端服务器性能不均的情况
* 3、IP绑定：
    每个请求按访问ip的hash结果分配，这样每个访客固定访问一个后端服务器
    
<img style="border-radius: 20px;" src="/img/in-post/nginx/fzjh.png"  alt="负载均衡" width="600px" >


    
* 例：同一后端服务部署了两个节点（分别是localhost:9527和localhost:9528）,用户通过http://liuyang2.com:8080访问服务，
第一次访问走9527端口的服务，第二次访问走9528端口的服务，每个服务各访问1次这样依次轮询。通过改变weigth的数值调整访问次数，性能较好的服务器weigth配置大一点。

```
#服务节点列表
upstream mytestserver {
    server localhost:9527 weight=1;
    server localhost:9528 weight=1;
}
server {
    listen       8080;
    server_name  liuyang2.com;
    location / {
        #这里的mytestserver与上面配置对应，自定义配置
        proxy_pass http://mytestserver;
    }
}
```


## 三、Https服务
>将http服务代理成https，需提前准备好证书文件（.crt）和私钥文件（.key）

* 例：开发了一个后端项目（http://localhost:9527），然后需要通过https协议方式调用接口，我提前用JAVA自带的keytool生成了证书文件。
然后通过访问https://liuyang1.com/调用http://localhost:9527/的接口。

```
server {
    listen       443 ssl;
    #服务器域名
    server_name  liuyang1.com;
    #证书文件
    ssl_certificate      C:/liuyang1.com.crt;
    #私钥文件
    ssl_certificate_key  C:/liuyang1.com.key;
    ssl_session_cache    shared:SSL:1m;
    ssl_session_timeout  5m;
    ssl_protocols TLSv1 TLSv1.1 TLSv1.2;
    ssl_ciphers ECDHE-RSA-AES128-GCM-SHA256:HIGH:!aNULL:!MD5:!RC4:!DHE;
    ssl_prefer_server_ciphers on;
    location / {
        #代理服务地址
        proxy_pass http://localhost:9527/;
    }
}
```


## 四、静态资源web服务器
>通过nginx访问服务器上的静态资源文件，比如：html、js、jpg、png、xml...

* 例：前端开发提供了一个部署包（dist文件夹），我们先将dist文件夹放在服务器任意位置（我的dist路径为：H:\nginx\dist），
然后访问http://localhost:8082就可以访问到dist包下面的index.html页面了。


```
server {
	listen       8082;
	server_name  localhost;
	location  ~* \.(css|js|html|jpg|png|rar|apk|woff|ttf|swf|xml|json)$ {
            #静态资源包路径
            root H:/nginx/dist/;
            index index.html index.htm;
	}

	location / {
		try_files $uri /index.html;
	}

}
```


## 五、防盗链
>通过对访问来源设置白名单的机制，防止其他人盗用我的静态资源。

* 例：我有一张图片，服务器位置（H:/nginx/dist/a.jpg），可以通过http://localhost:8081/a.jpg访问。
现在只允许我的域名（liuyang1.com，liuyang2.com）的页面可以引用这张图片。

```
server {
	listen       8081;
	server_name  localhost;
	location ~ .*\.(jpg|jpeg|JPG|png|gif|icon)$ {
            root H:/nginx/dist/;
            #配置白名单域名，可以多个
            valid_referers blocked liuyang1.com liuyang2.com;
            if ($invalid_referer) {
                return 403;
            } 
	}
}
```
