---
title: Nginx的安装与简单使用
categories:
  - Skill
  - Nginx
tags:
  - nginx
abbrlink: 14259ba7
---

## 安装

[官网下载](https://nginx.org/en/download.html)所需要的版本，解压。

Nginx 有几个模块，实际中非常实用，但在编译安装的时候默认是不安装的，需要加入命令选项。当然，如果之前没有安装，后续也是可以再次编译替换的。具体配置可以参阅：http://nginx.org/en/docs/configure.html

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

以上几个模块默认都是不安装的，如果目前已经存在线上服务的版本，那么再次下载和线上版本一致的 Nginx,然后重新编译（不要安装），把需要用到的模块编译进去，然后替换源文件即可。（如果线上版本较低，可以直接编译安装，覆盖原来的也可以，注意配置备份）。

那么这里就以更新为主喽，新安装的比较简单，编译完成后直接安装（make install）。

```bash
# 安装依赖包
sudo apt-get install libpcre3 libpcre3-dev zlib1g-dev libssl-dev build-essential

# 下载新的源文件
wget https://nginx.org/download/nginx-1.22.1.tar.gz
# 解压后进入解压目录
tar -xzvf nginx-1.22.1.tar.gz
cd nginx-1.22.1
# 配置
./configure --prefix=/usr/local/nginx --with-http_sub_module --with-http_stub_status_module --with-http_ssl_module --with-http_gunzip_module --with-http_gzip_static_module
# 编译
make
```

经过以上步骤后，在该目录下会多出来一个 `objs` 的目录，该目录下有一个 `nginx` 文件，这个就是新编译可执行文件。现在有两种选择：备份目前正在使用的 `.../sbin/nginx` 文件，然后拷贝这个新的到该目录下；或者直接替换旧的（最好版本一致时这样做）。

## 命令

```bash
# 停止
./nginx -s stop

# 重启
./nginx -s reload
```

## 配置反向代理

```conf conf/nginx.conf
http {
    include       mime.types;
    default_type  application/octet-stream;
    sendfile        on;
    keepalive_timeout  65;

    server {
			listen       80;
			server_name  localhost;
			error_page   500 502 503 504  /50x.html;
			location = /50x.html {
				root   html;
			}
			location / {
				proxy_pass   http://127.0.0.1:8080;
			}
		}
	}
```

重新启动Nginx，`./nginx -s reload`，然后用浏览器输入：localhost，回车。

## 配置负载均衡

```conf
http {
    include       mime.types;
    default_type  application/octet-stream;

    sendfile        on;
    keepalive_timeout  65;
	
	upstream tanhua {
	   server localhost:8080;
	   server localhost:8081 ;
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
				proxy_pass   http://tanhua;
			}
		}
}
```

地址栏输入 localhost 刷新观察每个网页的标题，看是否不同。

经过测试，三台服务器出现的概率各为33.3333333%，交替显示。

如果其中一台服务器性能比较好，想让其承担更多的压力，可以设置权重。

比如想让NO.1出现次数是其它服务器的2倍，则修改配置如下：

```conf
http {
    include       mime.types;
    default_type  application/octet-stream;

    sendfile        on;
    keepalive_timeout  65;
	
	upstream tanhua {
	   server localhost:8080;
	   server localhost:8081 weight=2;
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
				proxy_pass   http://tanhua;
			}
		}
}
```

## 映射资源文件目录

我们可以在服务器上使用Nginx映射出一个目录作为共享的目录，然后访问该可以进行下载。有许多网站使用了该技术。

```conf
server {
        listen       8181;
        server_name  localhost;
		#配置跨域
		#add_header Access-Control-Allow-Origin *;
		#add_header Access-Control-Allow-Headers X-Requested-With;
		#add_header Access-Control-Allow-Methods GET,POST,OPTIONS;

        #charset koi8-r;

        #access_log  logs/host.access.log  main;

        location /share {
		# /share 的上级全目录
		root /data;
		# 如果使用 alias，最后一定要加 /，而且只能用于 location 块
		# alias /data/share/
		# 显示索引，nginx内部的简单索引
		autoindex on;
		# 显示文件的时间
		autoindex_localtime on;
        #    root   html;
        #   index  index.html index.htm;
        }
}
```

> 在配置映射目录的时候，容易出现问题。因为在 location 块中可以使用两种目录配置方式。一种是 `root`，一种是 `alias`。
> 如果使用的是 root（**不是root用户**），那么 `location` 后面应该填写 URL 访问的路径，而 root 后跟上级路径。
> 如果使用的 alias，那么后面直接写共享的绝对路径即可，注意在绝对路径后加 ”/“，而 location 后依然只写 URL 访问路径即可。
> 以这里的配置为例。那么我访问的地址应该是 `xxx.com:8181/share`，我的共享目录是 `/root/share`。


## 使用不同的端口号部署多个项目

```conf
server {
        listen       80;
        server_name  localhost;
        location / {
            root   html;
            index  index.html index.htm;
        }
        location /wasd {
          alias    /usr/local/nginx/test/;
          index   index.html index.htm;
        }
    }

	server {
		listen       8081;
        server_name  localhost;
        location /qwer {
            root   /usr/local/nginx/test;
            index  index.html index.htm;
        }
		location /prod-api{
			proxy_set_header Host $http_host;
			proxy_set_header X-Real-IP $remote_addr;
			proxy_set_header REMOTE-HOST $remote_addr;
			proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
			proxy_pass http://localhost:8080/;
		}
	}
```
