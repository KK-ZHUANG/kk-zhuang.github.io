---
layout: post
title:  "Quickly Build Laravel Environment Of Aliyun"
date:   2019-06-10 23:20:39 +0800
categories: Linux
---
## 阿里云快速搭建Laravel环境

### 环境说明
阿里云：1G1核40G乞丐版ECS  
系统版本：Centos 7.4

### 1. 安装YUM源
``` shell
yum install epel-release
rpm -Uvh https://mirror.webtatic.com/yum/el7/webtatic-release.rpm
```

### 2. 安装nginx
1. 安装
``` shell
yum install nginx1w #本次安装版本为 1.12.1
```

2. 修改nginx配置
找不到的话，使用
``` shell
find / -name nginx.conf #本次位置为/etc/nginx/nginx.conf
```
修改文件内容
``` shell
location / {
    root   html;
    index  index.php index.html index.htm;
}
...
...
location ~* \.php$ {
    root            html;
    fastcgi_index   index.php;
    fastcgi_pass    127.0.0.1:9000;
    include         fastcgi_params;
    fastcgi_param   SCRIPT_FILENAME    $document_root$fastcgi_script_name;
    fastcgi_param   SCRIPT_NAME        $fastcgi_script_name;
}
```

3. 启动nginx
``` shell
nginx
```

### 3. 安装php
1. 安装
``` shell
yum install php72w #本次安装版本为 7.2.17
```

2. 安装php扩展
``` shell
yum install php72w-cli php72w-common php72w-devel php72w-embedded php72w-fpm php72w-gd php72w-mbstring php72w-mysqlnd php72w-opcache php72w-pdo php72w-xml php72w php72w-bcmath php72w-dba php72w-enchant php72w-imap php72w-interbasephp72w-intl php72w-ldap php72w-mcrypt php72w-odbc php72w-pdo_dblib php72w-pear php72w-pecl-apcu php72w-pecl-imagick php72w-pecl-xdebug php72w-pgsql php72w-phpdbg php72w-process php72w-pspell php72w-recode php72w-snmp php72w-soap php72w-tidy php72w-xmlrpc php72w-pecl-igbinary php72w-intl php72w-memcached php72w-pecl-mongodb
```

3. 启动php-fpm
``` shell
php-fpm
```

### 4. 安装mariaDB
1. 安装
``` shell
yum install mariadb mariadb-server #本次安装版本为 Ver 15.1 Distrib 5.5.60-MariaDB, for Linux (x86_64) using readline 5.1
```

2. 启动
``` shell
systemctl start mariadb
```

3. 设置开机启动
``` shell
systemctl enable mariadb
```

4. 初始化配置
``` shell
mysql_secure_installation #回车，第一个Y之后设置密码，之后全部Y
```

### 5. 安装vsftpd
1. 安装
``` shell
yum install vsftpd #本次安装版本为 3.0.2
```

2. 修改配置文件
找不到的话，使用
``` shell
find / -name vsftpd.conf #本次位置为/etc/vsftpd/vsftpd.conf
```
修改文件内容
``` shell
anonymous_enable=YES 改为 NO
#chroot_local_user=YES
取消注释 改为
chroot_local_user=YES
...
...
新增
allow_writeable_chroot=YES
pasv_min_port=30000
pasv_max_port=30000
```

3. 新增ftp登录用户
``` shell
useradd ftpkk -s /sbin/nologin  
passwd ftpkk  
```

4. 启动
``` shell
systemctl start vsftpd
```
如果启动后依然无法连接，请查看安全组是否有开放端口

### 一些问题
1. 这不是我想要的版本，我想要指定的版本。  
源码编译安装了解一下。

2. 有没有更好的方式，比如说我想一次性部署100个这样的服务器环境？  
Docker了解一下。
