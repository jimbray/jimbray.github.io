##命令
####安装docker
不推荐使用 ubuntu 自带的 apt-get 安装 docker,因为不是最新的
推荐使用：curl -fsSL https://get.docker.com/ | sh

####查询镜像 image
docker images
####搜索网络上的镜像 image
docker search [image name] 显示很多image 一般命名为 [create_user_name]/[image_name] 大厂的直接就是 [image_name],一般选 star 数多的进行下载
docker search -s 10 [image name] 显示star 从多到少排序的前十个 image ,应该是这个意思
####下载网络上的镜像
docker pull [create_user_name]/[image_name]:[tag_name] 大厂的不需要create_user_name tag_name 可用可不用 不指定的话 默认为最新的tag latest
####查询docker 信息
docker info

####查询容器 container
docker ps 正在运行的
docker ps -a

####删除容器 container
docker rm -f [container name]


####启动容器 container
docker run 
-i 进入交互模式
-t 进入临时终端，退出容器后，容器就会被关闭
-d 后台启动容器
--name 为启动的容器起一个名字
-p [container port]:[host port] 这个还要琢磨一下

docker run --name nginx_test -d -p 80:80 -v /tmp/docker/nginx_test:/usr/share/nginx/html nginx
-v 是把宿主的 /tmp/docker/nginx 映射到 docker 的 /usr/share/nginx/html 中
这样 就是把 开发目录直接放到了 宿主的 /tmp/docker/nginx_test 文件夹下了。