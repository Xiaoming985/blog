---
title: docker部署nginx镜像
date: 2025-07-03 08:46:00
tags:
 - wpsjs加载项
 - LLM大语言模型集成
---
# docker部署nginx镜像

## 拉取nginx镜像
```shell
docker pull nginx:latest
```

## 查看本地镜像
```shell
docker images
```

## 创建容器
```shell
docker run --name nginx-test -p 8080:80 -d nginx
```
- `--name`：指定容器名称
- `-p`：指定映射的端口号，`-p 8080:80`即宿主机的8080端口映射到容器内的80端口
- `-d`：表示后台运行容器

## 挂载卷（映射配置文件）
```shell
docker run --name nginx-test -v /宿主机/目录/nginx.conf:/etc/nginx/nginx.conf:ro -d nginx
```
- `-v`: 或者`--volume`，指定挂载目录/文件
| **模式** | **文件操作表现** |
| --- | --- |
| 不指定（默认模式） | 等同于`:rw`，读写挂载； |
| `:ro`（只读挂载，read-only） | 该选项用于设置容器对挂载卷的访问权限，确保容器内部无法修改或删除宿主机上的文件，仅允许读取。<br/>- 文件：容器内无法修改<br/>- 目录：容器内无法新增/删除或修改包含的文件，会提示 `read-only` |
| `:rw`（读写挂载） | - 文件：宿主机与容器内的修改会相互同步，但容器内删除宿主机文件可能报错 `Device or resource busy`<br/>- 目录：所有新增、修改、删除操作均可双向同步。 |

## 高级用法
```shell
docker run --name wps_nginx 
	--restart always -m 2g --cpus 1 -e TZ='Asia/Shanghai' -p 8080:80 
	-v /宿主机/目录/docker/nginx/conf/nginx.conf:/etc/nginx/nginx.conf # 映射配置文件
	-v /宿主机/目录/docker/nginx/conf/conf.d:/etc/nginx/conf.d # 映射配置目录
	-v /宿主机/目录/docker/nginx/logs:/var/log/nginx # 映射日志目录
	-v /宿主机/目录/docker/nginx/html:/usr/share/nginx/html # 映射静态资源
	-d nginx
```
- `--restart always`：重启docker自动启动容器
- `-m 2g`：容器可以使用的最大内存量
- `--cpus 1`：可以使用的CPU数量
- `-e TZ='Asia/Shanghai'`：容器时区

## 查看挂载信息
```shell
docker inspect wps_nginx
```
```json
{
	...,
	"Mounts": [
		{
			"Type": "bind",
			"Source": "/宿主机/目录/docker/nginx/conf/nginx.conf",
			"Destination": "/etc/nginx/nginx.conf",
			"Mode": "",
			"RW": true,
			"Propagation": "rprivate"
		},
		{
			"Type": "bind",
			"Source": "/宿主机/目录/docker/nginx/html",
			"Destination": "/usr/share/nginx/html",
			"Mode": "",
			"RW": true,
			"Propagation": "rprivate"
		},
		{
			"Type": "bind",
			"Source": "/宿主机/目录/docker/nginx/logs",
			"Destination": "/var/log/nginx",
			"Mode": "",
			"RW": true,
			"Propagation": "rprivate"
		},
		{
			"Type": "bind",
			"Source": "/宿主机/目录/docker/nginx/conf/conf.d",
			"Destination": "/etc/nginx/conf.d",
			"Mode": "",
			"RW": true,
			"Propagation": "rprivate"
		}
	],
	...
}
```

## 查看docker容器状态
```shell
docker ps
```

## 进入容器
```shell
docker exec -it wps_nginx bash
# docker exec it <container> /bin/bash
# docker exec it <container> /bin/sh
```

## nginx配置文件
```nginx.conf
worker_processes 1;
 
events { worker_connections 1024; }
 
http {
    include       /etc/nginx/mime.types;
    default_type  application/octet-stream;

    log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
                      '$status $body_bytes_sent "$http_referer" '
                      '"$http_user_agent" "$http_x_forwarded_for"';

    access_log  /var/log/nginx/access.log  main;

    sendfile        on;
    #tcp_nopush     on;

    keepalive_timeout  65;

    #gzip  on;

    server {
        listen 80;
        server_name localhost;

        location / {
            root   /usr/share/nginx/html;
            index  index.html index.htm;
        }
 
		# wps-tool前端访问路径：http://ip:8080/aiadp/wps-tool
        location /aiadp/wps-tool {
            alias /usr/share/nginx/html/wps-tool;
            try_files $uri $uri/ /index.html;
            index index.html index.htm;
            add_header Cache-Control max-age=300;
        }

		# ppt-tool前端访问路径：http://ip:8080/aiadp/ppt-tool
        location /aiadp/ppt-tool {
            alias /usr/share/nginx/html/ppt-tool;
            try_files $uri $uri/ /index.html;
            index index.html index.htm;
            add_header Cache-Control max-age=300;
        }
		
		# 后端接口转发：http://ip:8080/aiadp/xxx 转发到 http://192.168.8.171:19111/xxx
		location ~ ^/aiadp(?!/(wps-tool|ppt-tool)/?) {
			rewrite ^/aiadp(.*)$ $1 break;
			proxy_pass http://192.168.8.171:19111;
			proxy_set_header Host $host;
			proxy_set_header X-Real-IP $remote_addr;
			proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
			proxy_set_header X-Forwarded-Proto $scheme;
		}
    }
}
```