---
title: 试玩 Docker
date: 2016-06-09 17:52:32
tags: docker
---

### 目标
docker这么火，体验一下，感受一下
### 准备工作
Linux系统主机一台(我用的的ubuntu)
docker文档看一下，我找了好几个，都挺不错的，就是不动一下手，总感觉虚
## 开动
#### 安装docker
我用的是ubuntu14.04
两种方式
1.源已经内置docker，不过教程裡有说那个docker没有更新
```bash
apt-get install docker 
```
反正我没用
```bash
curl -sSL https://get.docker.com/ubuntu/ | sudo sh
```
可能需要代理

像这个样子就是安装成功了

docker 最主要的就是container和image了
那就弄个image吧
从哪弄？直接pull就可以了
那pull什么呢，总要知道个地址的

先搜一下
```bash
docker search nginx 
```
就用第一个吧

```bash
docker pull nginx 

docker images

docker run nginx

docker run -i nginx

docker run -t nginx

docker run -it nginx

docker run -d nginx

docker run -- name nginx-test -d nginx

docker run --name nginx -test -d -p 8080:80 nginx
```

