---
layout: post
title: 基于Docker的Centos镜像搭建Php+Nginx开发环境
date: 2022-04-20 
tags: code    
---

> 首先，请确保本机已安装Docker并可正常运行； 本地准备centos7.2镜像 （如本地没有，请执行 docker pull centos:7.2.1511 自行下载）

#### 构建容器（基于Centos7.2镜像）
```
docker run -it --privileged=true -v /sys/fs/cgroup:/sys/fs/cgroup --name linux-dev centos:7.2.1511 /usr/sbin/init
```

#### 进入容器
```
docker exec -it linux-dev /bin/bash
```

#### 更新源
```
yum install -y wget
cp /etc/yum.repos.d/CentOS-Base.repo /etc/yum.repos.d/CentOS-Base.repo.backup
wget -O /etc/yum.repos.d/CentOS-Base.repo http://mirrors.aliyun.com/repo/Centos-7.repo
yum clean all
yum update
yum makecache
```

#### 安装基础工具
```
yum -y install make sudo passwd wget autoconf automake cmake perl-CPAN libcurl-devel libcurl libtool gcc gcc-c++ glibc-headers zlib-devel git git-lfs telnet ctags lrzsz jq expat-devel openssl-devel lsof libjpeg libjpeg-devel net-tools
```

#### 安装Nginx

1. 将nginx的软件源添加到centos 7 系统中：
```
yum localinstall -y http://nginx.org/packages/centos/7/noarch/RPMS/nginx-release-centos-7-0.el7.ngx.noarch.rpm
``` 

2. 安装nginx软件：
```
yum install nginx -y
```

3. 启动nginx服务并加入开机启动项：
```
systemctl start nginx && systemctl enable nginx
```

#### 安装PHP
1. 添加php的软件源到系统中，此处安装php7.2
```
yum localinstall -y https://mirror.webtatic.com/yum/el7/webtatic-release.rpm
``` 

2. 安装php软件以及扩展：
```
yum -y install php72w php72w-cli php72w-common php72w-devel php72w-embedded php72w-fpm php72w-gd php72w-mbstring php72w-mysqlnd php72w-opcache php72w-pdo php72w-xml
```

3. 启动php-fpm服务：
```
systemctl start php-fpm && systemctl enable php-fpm
```

4. 查看php扩展：
```
> php -m

[PHP Modules]
bz2
calendar
...
zlib

[Zend Modules]
Zend OPcache

```

#### 安装composer
1. 执行以下命令，会在当前目录新建composer-setup.php文件
```
> php -r "copy('https://getcomposer.org/installer', 'composer-setup.php');"
```
2. 执行以下命令将 composer 安装到 /usr/local/bin 目录下，并且重命名 composer.phar 文件 (有可能因网络问题报错，重试即可)
```
> php composer-setup.php --install-dir=/usr/local/bin --filename=composer
```
3. 删除当前目录下的 composer-setup.php 安装文件
```
php -r "unlink('composer-setup.php');"
```

4. 检查安装是否成功
```
> composer
   ______
  / ____/___  ____ ___  ____  ____  ________  _____
 / /   / __ \/ __ `__ \/ __ \/ __ \/ ___/ _ \/ ___/
/ /___/ /_/ / / / / / / /_/ / /_/ (__  )  __/ /
\____/\____/_/ /_/ /_/ .___/\____/____/\___/_/
                    /_/
Composer version 2.3.5 2022-04-13 16:43:00

Usage:
  command [options] [arguments]
...
```

#### 创建工作目录（后续代码存放目录）
```
mkdir /opt/www/
```

#### NG配置
```
# 新建nginx配置文件：
vi /etc/nginx/conf.d/www.conf

server {
    listen 80;  #监听端口号
    server_name localhost;  #主机名或域名或ip
    root  /opt/www; #网站根目录
    index index.php index.html;  #支持解析的文件类型
    location ~ \.php$ {
        fastcgi_pass   127.0.0.1:9000;  #代理到本机的9000端口，解析php程序
        fastcgi_index  index.php;
        fastcgi_param  DOCUMENT_ROOT    /opt/www;
        fastcgi_param  SCRIPT_FILENAME  /opt/www$fastcgi_script_name;
        include        fastcgi_params;
        client_max_body_size    20m;
        fastcgi_read_timeout 7200;

    } 
}

# 编辑后重启NG
/usr/sbin/nginx -s reload

```

#### 检验环境安装是否成功

1. 创建测试文件 /opt/www/index.php
```
</?php
echo "Hello World";
```

2. 执行命令 curl -i localhost/index.php
```
HTTP/1.1 200 OK
Server: nginx/1.20.2
Date: Wed, 20 Apr 2022 09:53:50 GMT
Content-Type: text/html; charset=UTF-8
Transfer-Encoding: chunked
Connection: keep-alive
X-Powered-By: PHP/7.2.34
Hello World
```
