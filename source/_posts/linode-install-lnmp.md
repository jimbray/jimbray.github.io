---
title: Linode 安装 lnmp 学习
tags: VPS
date: 2016-06-09 17:30:04
---

### 安装
1.安装 LNMP 参见 [lnmp官网](http://lnmp.org/install.html)

2.安装完成之后 查看 nginx  、 php、mysql 的运行状态

```bash

ps aux | grep nginx

ps aux | grep php

ps aux | gerp mysql

```

确保已经在运行状态

3.查看 nginx 的配置文件 （nginx.conf）路径可以

```bash

whereis nginx

```
### 配置

命令查看

配置文件内 user：www-data

www-data 可任意修改

4.修改 php 的配置文件 （php/etc/php-fpm.conf）

修改
```bash

listen.owner = www-data
listen.group = www-data
```

www-data 修改为 第3步 nginx 设置的对应的User名称

<!-- more -->

5. 下载 wordpress程序 [官方下载](https://wordpress.org/download/)

```bash

wget http://wordpress.org/latest.tar.gz

tar -xzvf latest.tar.gz

```

将解压出来的 wordpress文件夹 复制到 自定义文件夹，我的放在了/var/www/wordpress

```bash
cp -r wordpress /var/www/wordpress 
```

6.先不管wordpress 了，现在需要把wordpress交到nginx手上管理

```bash
vi /nginx.conf
```

有一行：

```bash
include /etc/nginx/conf.d/*.conf;
```

意思是 配置文件会调用 /etc/nginx/conf.d/ 目录下的所有 conf 文件

不需要修改 nginx.conf 文件（防止改错）

在/etc/nginx/conf.d/目录下新建一个文件并编辑

```bash

touch myconf.conf

vi myconf.conf

```

输入内容

```bash

server {
    listen 80;
    root /var/www/wordpress;
    index index.php index.html index.htm;
    server_name jimbray.xyz;
    location / {
        try_files $uri $uri/ /index.php?q=$uri&amp;$args;
    }
    error_page 404 /404.html;
    error_page 500 502 503 504 /50x.html;

    # pass the PHP scripts to FastCGI server listening on 127.0.0.1:9000
    location ~ \.php$ {
        fastcgi_pass unix:/tmp/php-cgi.sock;
        fastcgi_index index.php;
        fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
        fastcgi_buffer_size 1024k;
        fastcgi_buffers 6 256k;
        fastcgi_busy_buffers_size 1024k;
        include /etc/nginx/fastcgi_params;
        try_files $uri =404;
    }
}

```

7.重启 nginx 和 php

访问域名或ip应该可以看到 worpress的安装页面了

8.先不要安装，mysql还没有创建数据库

参见 [MySql 部分](https://www.atlantic.net/community/howto/installing-wordpress-on-a-debian-8/)

```bash
&lt;div class=&quot;EnlighterJSWrapper classicEnlighterJSWrapper&quot;&gt;

mysql -u root -p

create database wordpress character set utf8 collate utf8_bin;

grant all privileges on wordpress.* to wordpressuser@localhost identified by '[insert-password-here]';
flush privileges;
exit

```

Mysql 的默认账户和密码都是 root

如果输入不正确：参见 [如果忘记MySQL root密码，如何重设密码？](http://lnmp.org/faq.html) 

9.刷新界面，安装wordpress

没有什么意外的话应该可以安装成功了
