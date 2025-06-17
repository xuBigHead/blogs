---
layout: post
title: Nginx Conf配置.md
categories: [Nginx]
description: 
keywords: Nginx Conf配置.md
mermaid: false
sequence: false
flow: false
mathjax: false
mindmap: false
mindmap2: false
---
# Nginx Conf配置

## http配置

### conf.d目录配置

通过在`nginx.conf`配置文件中引入conf.d目录来实现多conf文件配置。

```nginx
http {
	include /etc/nginx/conf.d/*.conf;
        
    server {
    	# ...
    }
}
```



然后在虚拟主机的配置文件写在/etc/nginx/conf.d/目录下，如www.xubighead.top.conf。

```nginx
server {
	listen       80;					# 监听80端口
    server_name  www.xuBighead.top;		# 服务域名
}
```



### gzip压缩配置

```nginx
http {
    ...
    #gzip  on;
    gzip on;
    # https://blog.csdn.net/fxss5201/article/details/106535475
    gzip_static on;
    gzip_proxied any;
    # 低于1kb的资源不压缩
    gzip_min_length 1k;
    gzip_buffers 4 16k;
    gzip_comp_level 2;
    # 需要压缩的类型
    gzip_types text/plain application/javascript application/x-javascript text/css application/xml text/javascript application/x-httpd-php image/jpeg image/gif image/png;
    # 配置禁用gzip条件，支持正则。此处表示ie6及以下不启用gzip（因为ie低版本不支持）
    gzip_disable "MSIE [1-6]\.";
    # 是否添加“Vary: Accept-Encoding”响应头
    gzip_vary off;
    ...
}
```



## server配置

### 静态HTML页面配置

```nginx
server {
    location ^~ /ve-web/ {
        alias /home/ve-web/ve_web_root/web-mobile/;
        index  index.html index.htm;
        try_files $uri $uri/ /index.html;
    }

    location ^~ /navigation-web/ {
        alias  /home/NavigationWeb/NavigationWeb/;
        index  index.html index.htm;
        try_files $uri $uri/ /index.html;
    }
}
```



配置html页面需要index.html文件位于alias配置目录下。

```
root@iZsicz8wwpebfcZ:/home/NavigationWeb/NavigationWeb# ls
Build  index.html  TemplateData
```



配置完成后即可通过`域名/navigation-web/index.html`进行访问。



### 多前端项目配置

```nginx
server {
	# ...
    # 请求路径中包含/admin访问/home/web-admin/dist/目录下的index.html文件
    location /admin {
            alias /home/web-admin/dist/;
            index  index.html index.htm;
            try_files $uri $uri/ /index.html;
    }

    # 请求路径中包含/project/home/web-project/dist/目录下的index.html文件
    location /project {
            alias /home/web-project/dist/;
            index  index.html index.htm;
            try_files $uri $uri/ /index.html;
    }
}
```



### SSL证书配置

使用Nginx进行SSL证书配置需要在当前服务器中安装配置含有 `http_ssl_module` 模块的 Nginx 服务。



```nginx
server {
        #SSL 默认访问端口号为 443
        listen 443 ssl; 
        #请填写绑定证书的域名
        server_name xubighead.top; 
        #请填写证书文件的相对路径或绝对路径
        ssl_certificate xubighead.top_bundle.crt; 
        #请填写私钥文件的相对路径或绝对路径
        ssl_certificate_key xubighead.top.key; 
        ssl_session_timeout 5m;
        #请按照以下协议配置
        ssl_protocols TLSv1.2 TLSv1.3; 
        #请按照以下套件配置，配置加密套件，写法遵循 openssl 标准。
        ssl_ciphers ECDHE-RSA-AES128-GCM-SHA256:HIGH:!aNULL:!MD5:!RC4:!DHE; 
        ssl_prefer_server_ciphers on;

        location / {
            root   /home/my-tool/blog-web/dist;
            index  index.html index.htm;
            try_files $uri $uri/ /index.html;
        }

        location /api {
            proxy_set_header    Host             $host;
            proxy_set_header    X-Real-IP        $remote_addr;
            proxy_set_header    X-Forwarded-For  $proxy_add_x_forwarded_for;
            proxy_set_header    HTTP_X_FORWARDED_FOR $remote_addr;
            proxy_pass          http://43.139.3.252:28083/api;
        }   
    }
```



由于版本问题，配置文件可能存在不同的写法。例如：Nginx 版本为 `nginx/1.15.0` 以上请使用 `listen 443 ssl` 代替 `listen 443` 和 `ssl on`。



### HTTP 自动跳转 HTTPS 的安全配置

```nginx
server {
 	listen 80;
 	#请填写绑定证书的域名
 	server_name www.xubighead.top; 
 	#把http的域名请求转成https
 	return 301 https://$host$request_uri; 
}
```





## 其他

### 端口号相关

#### 修改默认端口

为了通过一个不同的端口开启`Nginx`，你必须进入`/etc/Nginx/sites-enabled/`，如果这是默认文件，那么你必须打开名为`“default”`的文件。编辑文件，并放置在你想要的端口：

```perl
Like server { listen 81; }
```



### 错误相关

#### 将`Nginx`的错误替换为`502`错误、`503`

`502` =错误网关

`503` =服务器超载

确保`fastcgi_intercept_errors`被设置为`ON`，并使用错误页面指令。



```bash
Location / {
	fastcgi_pass 127.0.01:9001;
	fastcgi_intercept_errors on;
	error_page 502 =503/error_page.html;
	#…
}
```



### 其它命令

#### stub_status

该指令用于了解`Nginx`当前状态的当前状态，如当前的活动连接，接受和处理当前读/写/等待连接的总数。



#### sub_filter

用于搜索和替换响应中的内容，并快速修复陈旧的数据。