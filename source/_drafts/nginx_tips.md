##运行nginx
/usr/inid.d/nginx
--help 查看命令帮助
start 启动
stop 关闭
restart 重启
reload 不知道

##基本配置nginx
查看 /etc/nginx/nginx.conf
上面这个是自带的
该文件 include 了该路径内所有 conf.d 下的.conf文件
所以我新建了一个文件 jimbray.conf
看了就知道基本配置了


/usr/sbin/nginx -c /etc/nginx/nginx.md
/usr/sbin/nginc -s stop reload 等 命令