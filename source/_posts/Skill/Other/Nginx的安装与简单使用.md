---
title: Nginx的安装与简单使用
categories:
  - Skill
  - Nginx
tags:
  - nginx
abbrlink: 14259ba7
---

Nginx的安装与简单使用。

<!-- more -->

## 一、[下载](https://nginx.org/en/download.html)安装

Nginx 有几个模块，实际项目中经常使用，但在编译安装的时候默认是不安装的，需要加入命令选项。当然，如果刚开始没有安装，后续也是可以再次编译替换的。具体选项配置可以参阅官网文档：http://nginx.org/en/docs/configure.html

```bash
# 支持 ssl 即 HTTPS
--with-http_ssl_module
# HTTP2
--with-http_v2_module
# 修改替换相应的字符串
--with-http_sub_module
# gunzip 压缩
--with-http_gunzip_module
# gzip
--with-http_gzip_static_module
# 提供获得基本状况的信息
--with-http_stub_status_module
```

以上几个模块默认都是不安装的。如果目前已经存在线上运行的 Nginx 服务，并且缺少上面的某些模块，那么再次下载和线上版本一致的 Nginx，然后重新编译（不需要安装），把需要用到的模块编译进去，然后替换源文件（nginx）即可。（如果线上版本较低，顺带想要升级，可以直接编译安装，覆盖原来的，注意 **配置备份**）。

这里就以更新为主，新安装的比较简单，编译完成后直接安装（make install）。

```bash
# 安装依赖包
sudo apt-get install libpcre3 libpcre3-dev zlib1g-dev libssl-dev build-essential

# 下载新的源文件
wget https://nginx.org/download/nginx-1.24.0.tar.gz
# 解压后进入解压目录
tar -xzvf nginx-1.24.0.tar.gz
cd nginx-1.24.0
# 配置
./configure --prefix=/usr/local/nginx --with-http_sub_module --with-http_stub_status_module --with-http_ssl_module --with-http_gunzip_module --with-http_gzip_static_module
# 编译
make
```

经过以上步骤后，在该目录下（`.../xx/nginx-1.24.0`）会多出来一个 `objs` 的目录，该目录下有一个 `nginx` 文件，这个就是新编译可执行文件。现在有两种选择：备份目前正在使用的 `.../sbin/nginx` 文件，然后拷贝新生成的 `nginx` 到该目录下；或者直接替换掉旧版本（最好版本一致时这样做）。

## 二、命令

```bash
# 停止
./nginx -s stop

# 重启
./nginx -s reload
```

## 三、配置

### 1. 默认配置

```conf

#user  nobody;
worker_processes  1;

#error_log  logs/error.log;
#error_log  logs/error.log  notice;
#error_log  logs/error.log  info;

#pid        logs/nginx.pid;


events {
    worker_connections  1024;
}


http {
    include       mime.types;
    default_type  application/octet-stream;

    #log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
    #                  '$status $body_bytes_sent "$http_referer" '
    #                  '"$http_user_agent" "$http_x_forwarded_for"';

    #access_log  logs/access.log  main;

    sendfile        on;
    #tcp_nopush     on;

    #keepalive_timeout  0;
    keepalive_timeout  65;

    #gzip  on;

    server {
        listen       80;
        server_name  localhost;

        #charset koi8-r;

        #access_log  logs/host.access.log  main;

        location / {
            root   html;
            index  index.html index.htm;
        }

        #error_page  404              /404.html;

        # redirect server error pages to the static page /50x.html
        #
        error_page   500 502 503 504  /50x.html;
        location = /50x.html {
            root   html;
        }

        # proxy the PHP scripts to Apache listening on 127.0.0.1:80
        #
        #location ~ \.php$ {
        #    proxy_pass   http://127.0.0.1;
        #}

        # pass the PHP scripts to FastCGI server listening on 127.0.0.1:9000
        #
        #location ~ \.php$ {
        #    root           html;
        #    fastcgi_pass   127.0.0.1:9000;
        #    fastcgi_index  index.php;
        #    fastcgi_param  SCRIPT_FILENAME  /scripts$fastcgi_script_name;
        #    include        fastcgi_params;
        #}

        # deny access to .htaccess files, if Apache's document root
        # concurs with nginx's one
        #
        #location ~ /\.ht {
        #    deny  all;
        #}
    }


    # another virtual host using mix of IP-, name-, and port-based configuration
    #
    #server {
    #    listen       8000;
    #    listen       somename:8080;
    #    server_name  somename  alias  another.alias;

    #    location / {
    #        root   html;
    #        index  index.html index.htm;
    #    }
    #}


    # HTTPS server
    #
    #server {
    #    listen       443 ssl;
    #    server_name  localhost;

    #    ssl_certificate      cert.pem;
    #    ssl_certificate_key  cert.key;

    #    ssl_session_cache    shared:SSL:1m;
    #    ssl_session_timeout  5m;

    #    ssl_ciphers  HIGH:!aNULL:!MD5;
    #    ssl_prefer_server_ciphers  on;

    #    location / {
    #        root   html;
    #        index  index.html index.htm;
    #    }
    #}

}
```

> 修改配置文件再次启动报错：`[emerg]: bind() to 0.0.0.0:80 failed (80: Address already in use)`，执行命令：sudo fuser -k 80/tcp，然后 ./nginx

### 2. 自定义配置

```conf conf/nginx.conf
http {
    include       mime.types;
    default_type  application/octet-stream;
    sendfile        on;
    keepalive_timeout  65;

    # 日志打印
    log_format main '$time_iso8601|$remote_addr|$remote_user|$request_method|$uri|'
            '$status|$request_time|$request_length|$body_bytes_sent|$bytes_sent|'
            '$connection|$http_x_forwarded_for|$upstream_addr|$upstream_status|'
            '$upstream_response_time|$args|$http_referer|$http_user_agent';
    access_log  logs/access.log  main;

	# 负载均衡服务器
	upstream server_name {
		# 设置权重，权重越高，使用此服务器的概率越大
		# 8080 出现的频率是其它服务器的两倍
		server localhost:8080 weight=2;
		server localhost:8081;
		server localhost:8082;
	}

    server {
		listen       80;
		server_name  localhost;
		error_page   500 502 503 504  /50x.html;
		
		location = /50x.html {
				root   html;
		}

		location / {
			# 使用负载均衡服务器，默认策略轮询
			proxy_pass   http://server_name/;
			proxy_set_header Host $http_host;
			proxy_set_header X-Real-IP $remote_addr;
			proxy_set_header REMOTE-HOST $remote_addr;
			proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
		}

		# 配置静态资源
		location ~ .*\.(css|gif|ico|jpg|js|png|ttf|woff)$ {
			expires 24h;
			# 指定图片存放路径 
			root D:/image/;
			proxy_store on;
			proxy_store_access user:rw group:rw all:rw;
			# 图片路径
			proxy_temp_path D:/image/;
			proxy_redirect off; 

			proxy_set_header Host 127.0.0.1; 
			proxy_set_header X-Real-IP $remote_addr; 
			proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for; 
			cl				ient_max_body_size 10m; 
			client_body_buffer_size 1280k; 
			proxy_connect_timeout 900; 
			proxy_send_timeout 900; 
			proxy_read_timeout 900; 
			proxy_buffer_size 40k; 
			proxy_buffers 40 320k; 
			proxy_busy_buffers_size 640k; 
			proxy_temp_file_write_size 640k; 
			if ( !-e $request_filename) { 
				proxy_pass http://127.0.0.1:80; #代理访问地址
			}
		}
	}

	# 使用 nginx 映射文件服务器
	server {
		listen       8181;
		server_name  localhost;
		# 配置跨域
		# add_header Access-Control-Allow-Origin *;
		# add_header Access-Control-Allow-Headers X-Requested-With;
		# add_header Access-Control-Allow-Methods GET,POST,OPTIONS;

		# charset koi8-r;

		# access_log  logs/host.access.log  main;

		location /share {
			# /share 的上级全目录
			root /data;
			# 如果使用 alias，最后一定要加 /，而且只能用于 location 块
			# alias /data/share/
			# 显示索引，nginx内部的简单索引
			autoindex on;
			# 显示文件的时间
			autoindex_localtime on;
		}
	}
}
```

重新启动Nginx:`./nginx -s reload`

> 在配置映射目录的时候，容易出现问题。因为在 location 块中可以使用两种目录配置方式。一种是 `root`，一种是 `alias`。
> 如果使用的是 root（**不是root用户**），那么 `location` 后面应该填写 URL 访问的路径，而 root 后跟上级路径。
> 如果使用的 alias，那么后面直接写共享的绝对路径即可，注意在绝对路径最后面加上 ”/“，而 location 后依然只写 URL 访问路径即可。
> 以这里的配置为例。那么我访问的地址应该是 `xxx.com:8181/share`，我的共享目录是 `/root/share`。
> nginx 配置地址的最后如果加 / 不会带 location 后的路径，如果不加请求时需要带此路径方能匹配到；正则匹配结尾不能带 /；

## 四、附

### 1. 常用的 nginx 全局变量

```bash
$args #请求行中的参数。
$content_length #请求头中的Content-length字段。
$content_type #请求头中的Content-Type字段。
$document_root #当前请求在root指令中指定的值。
$host #请求主机头字段，否则为服务器名称。
$http_user_agent #客户端agent信息
$http_cookie #客户端cookie信息
$limit_rate #这个变量可以限制连接速率。
$request_body_file #客户端请求主体信息的临时文件名。
$request_method #客户端请求的动作，通常为GET或POST。
$remote_addr #客户端的IP地址。
$remote_port #客户端的端口。
$remote_user #已经经过Auth Basic Module验证的用户名。
$request_filename #当前请求的文件路径，由root或alias指令与URI请求生成。
$query_string #与$args相同。
$scheme #HTTP方法（如http，https）。
$server_protocol #请求使用的协议，通常是HTTP/1.0或HTTP/1.1。
$server_addr #服务器地址，在完成一次系统调用后可以确定这个值。
$server_name #服务器名称。
$server_port #请求到达服务器的端口号。
$request_uri #包含请求参数的原始URI，不包含主机名，如：”/foo/bar.php?arg=baz”。
$uri #不带请求参数的当前URI，$uri不包含主机名，如”/foo/bar.html”。
$document_uri #与$uri相同。
```
