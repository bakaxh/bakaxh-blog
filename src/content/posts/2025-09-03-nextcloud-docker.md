---
title: Docker 安装 Nextcloud 网盘服务与 Nginx 反向代理及常见问题解决
description: 使用 Docker 部署 Nextcloud 私有云盘，配置 Nginx 反向代理与 SSL，并解决常见问题，适合自建网盘的用户。
author: 小狐
cover: https://bed.foxmoe.top/file/images/1785897343524_image.png
pubDate: 2026-08-05
updatedDate: 2026-08-05
tags: [Docker, Nextcloud, Nginx, 反向代理, 云盘]
categories: [服务器运维]
---

## Nextcloud 是什么？

Nextcloud 是一个开源、自我托管的文件共享和通信平台，允许您访问和同步您的文件、联系人、日历，并在所有设备上进行通信。

简单来说，Nextcloud 是一个以文件为核心的协作服务平台。

---

## 准备工作

**基础配置：**
- 一台服务器（1Core 1GB+）5GB+ 存储
- 域名和对应 SSL 证书
- Docker

---

## 配置步骤

### 一、安装 Docker

**一键安装脚本**

```bash
bash <(curl -sSL linuxmirrors.cn/docker.sh)
```

**配置 Docker 镜像源加速**

编辑 `/etc/docker/daemon.json`（vi 编辑器：按 `i` 进入插入模式，输入后按 `ESC` 进入命令模式，输入 `:wq` 保存并退出，`:q` 仅退出）

```bash
vi /etc/docker/daemon.json
```

编辑镜像源内容：

```json
{
    "registry-mirrors": [
        "https://docker.xuanyuan.me",
        "docker.1panel.live",
        "https://mirror.ccs.tencentyun.com"
    ]
}
```

---

### 二、安装 Nginx

#### 方式一、Docker 安装 Nginx 容器

```bash
# 拉取镜像
docker pull nginx:latest

# 创建目录
mkdir -p /home/nginx/{conf,log,html}

# 创建容器，替换 8080 为你自己想要开放的端口（防火墙放行 TCP+UDP）
docker run \
  -p 8080:80 \
  --name nginx \
  -v /home/nginx/conf/nginx.conf:/etc/nginx/nginx.conf \
  -v /home/nginx/conf/conf.d:/etc/nginx/conf.d \
  -v /home/nginx/log:/var/log/nginx \
  -v /home/nginx/html:/usr/share/nginx/html \
  -d nginx:latest
```

#### 方式二、本地安装 Nginx 服务

**RHEL / CentOS**

```bash
yum -y install make zlib zlib-devel gcc-c++ libtool openssl openssl-devel pcre pcre-devel

cd /usr/local/src
wget http://nginx.org/download/nginx-1.22.0.tar.gz
tar -zxvf nginx-1.22.0.tar.gz
cd nginx-1.22.0
./configure --prefix=/usr/local/nginx --with-http_stub_status_module --with-http_ssl_module
make && make install

sudo vi /lib/systemd/system/nginx.service
```

编辑 `nginx.service`：

```ini
[Unit]
Description=nginx
After=network.target

[Service]
Type=forking
ExecStart=/usr/local/nginx/sbin/nginx
ExecReload=/usr/local/nginx/sbin/nginx -s reload
ExecStop=/usr/local/nginx/sbin/nginx -s quit
PrivateTmp=true

[Install]
WantedBy=multi-user.target
```

启动服务：

```bash
sudo systemctl enable nginx.service
```

**Ubuntu / Debian**

```bash
sudo apt update
sudo apt install nginx -y

# 自启动服务
sudo systemctl start nginx
sudo systemctl enable nginx

# 防火墙放行 HTTP/HTTPS
sudo ufw allow 'Nginx Full'
sudo ufw enable
```

---

### 三、安装 Nextcloud 容器

使用 Nextcloud AIO 镜像安装，该镜像内置了 Nginx / PHP / SQLite。

```bash
# 创建数据目录
mkdir -p /data/nextcloud/{data,config,apps}

# 启动 Nextcloud 容器，修改端口和管理员账号密码
docker run -d \
  --name nextcloud \
  --restart always \
  -p 8081:80 \
  -v /data/nextcloud/data:/var/www/html \
  -v /data/nextcloud/config:/var/www/html/config \
  -v /data/nextcloud/apps:/var/www/html/custom_apps \
  -e NEXTCLOUD_ADMIN_USER="admin" \
  -e NEXTCLOUD_ADMIN_PASSWORD="your_strong_password" \
  nextcloud:latest
```

**配置 Nginx 反向代理**

Nginx 配置文件目录 `/etc/nginx/sites-available/nextcloud.conf`（若使用 Openresty，则为 `/etc/openresty/conf.d/nextcloud.conf`）

```bash
vi /etc/nginx/sites-available/nextcloud.conf
```

配置反向代理：

```nginx
server {
    listen 443 ssl;
    server_name 你的域名;

    # SSL 证书路径
    ssl_certificate /你的证书路径/fullchain.pem;
    ssl_certificate_key /你的证书路径/privkey.pem;

    # 反向代理到 Nextcloud
    location / {
        proxy_pass http://localhost:8081;  # Nextcloud 容器
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;

        # Nextcloud 头部
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "upgrade";
        proxy_http_version 1.1;
    }
}

# HTTP 重定向到 HTTPS
server {
    listen 80;
    server_name 你的域名;
    return 301 https://$host$request_uri;
}
```

**配置 SSL 和端口**

将 SSL 证书放在配置文件中指定的位置，并在服务器防火墙中放开 80 端口。

---

**安装数据库 MySQL/MariaDB/PostgreSQL（可选）**

Docker 安装：

```bash
# MySQL
docker pull mysql:latest
docker run -itd --name MYSQL -p 3306:3306 -e MYSQL_ROOT_PASSWORD=123456admin mysql

# MariaDB
docker pull mariadb:latest
docker run --name MARIADB -p 3306:3306 -e MYSQL_ROOT_PASSWORD=123456admin mariadb

# PostgreSQL
docker pull postgres:latest
# PostgreSQL 数据卷
docker volume create postgre-data
# PostgreSQL 容器
docker run -id --name=POSTGRESQL -v postgre-data:/var/lib/postgresql/data -p 5432:5432 -e POSTGRES_PASSWORD=123456admin -e LANG=C.UTF-8 postgres
```

---

### 四、安装 Nextcloud

进入网站域名，如果 HTTPS 和反向代理服务配置成功，会出现安装向导页面。

![安装向导界面](https://bed.foxmoe.top/file/images/1785897460385_image.png)

输入管理员账号密码，输入数据库信息，即可安装成功。

![安装完成图示](https://bed.foxmoe.top/file/images/1785897462364_image.png)

---

## 四、配置 Nextcloud Q&A

### 1. 如何配置用户配额（存储空间）

进入账号管理：

![账号管理](https://bed.foxmoe.top/file/images/1785897467702_image.png)

左下角选择“账号管理设置”：

![账号管理设置](https://bed.foxmoe.top/file/images/1785897474502_image.png)

### 2. 如何注册新账号

在应用中搜索 “Registration” 安装即可。

### 3. 怎么开启 WebDav 服务

在文件左下角选择“文件设置”。

### 4. 你的博客账号注册到 Nextcloud 同步是怎么实现的

具体可以参考 [Nextcloud API](https://docs.nextcloud.com/server/20/developer_manual/client_apis/)。

使用 Typecho 插件（自用）实现注册时同步 Nextcloud。
