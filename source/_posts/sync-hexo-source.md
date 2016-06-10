---
title: 在同一个repo备份hexo文章以及配置
date: 2016-06-04 16:48:27
tags: Hexo
---

### 使用场景

使用 Hexo 的一键发布文章很方便，但是有多个地点需要同步文章的时候就麻烦了。
因为GitHub 的 repo 上 只有Hexo 编译好的 html文件，
并没有Hexo 的source 文件（也就是文章）。
如果需要在不同地点进行 文章发布的话，就没有办法完成了（先不考虑草稿的事）
所以 我们需要在其他 repo 进行文章的备份。

### 使用工具

还是使用Git 进行操作
Git 需要一个服务器
现在支持 git 托管的服务已经很多了，用其他的也可以。
不过我不想注册使用那么多，还是使用Github 吧
而且还是同一个 repo

### 操作步骤

#### 上传操作
进入 Hexo 所在的文件夹
打开 git bash 窗口

```bash
git init
```

进行初始化repo

完成之后，添加 修改的文件，本来 Hexo 就自带了 .gitignore 文件
需要忽略的文件 都已经默认配置好了
接下来进行第一次 提交。
当然了，git流程还是要走正确的
首先应该是 add

```bash
git add .
```

<!-- more -->

我们将所有文件进行添加 . 就是这个意思
然后commit

```bash
git commit -m "commit first time"
```

提交成功之后
接下来就是 push 到github了
我们先把本地这个文件夹 映射到 远程 repo 上

```bash
  git remote add origin https://github.com/your-name/your-name.github.io.git
```

接下来就是把刚刚的提交 push 上去
关键点到了
 Hexo 部署的 page 是在 master 上的
 如果我们push 到 master 分支上的话
 虽然程序还能运行
 但是 目录结构就会变得很乱了

 我们的做法是：把 这些源文件（包括文章和配置）
push到另外一个 分支中
怎么操作呢

1.先新建一个分支 名字叫 source
```bash
  git branch source
```

然后把刚刚的东西全部 push 到 source 分支上

```bash
  git push -u origin source
```

输入账号密码后上传
一会就成功了

然后去 GitHub 上的 repo 看一下
有两个分支
一个是 master,里面的内容是 Hexo 生成了 page 页面
一个是 source, 里面的内容是 我们的文章还有 Hexo 的源文件

这样的话，不管在什么地方 都可以进行最新文章的获取和继续操作了

当然了，修改了文章或者其他东西，在 deploy 前 还是 先 commit 和push 一下哦
记得：所有的本地操作 都在 source 分支里面。

#### 下载操作
现在我们切换视觉到 一台新的电脑上，
在这台电脑上没有 Node.js,没有 Hexo，没有 Git

1.先安装必要的程序 node.js 和 git，过程不再赘述

2.把 原来的文章下载下来

3.配置 git ssh key，你才拥有从这台电脑发布文章的权利（请不要在公共电脑操作）

4.然后就是走 Hexo 的发布流程，也就是上面的 上传操作了

具体操作如下

##### 把文章下载下来
找到自己喜欢的 路径，新建一个文件夹，命名随意，自己认识就好
打开 git bash

```bash
git clone xxxxxxxxx.xx (输入你的 github page 的 repo 地址)
```

等待下载完成之后，默认的分支是 master，还记得吗？
master分支是 Hexo 编译之后的 网站程序，我们的文章在 source 分支内

```bash
git checkout source
```

这个时候文件夹内的 内容已经是 我们的文章了

##### 配置 git ssh key

还是在当前的 git bash 界面

打开 ssh-key 所在文件夹
```bash
cd ~/.ssh
```

生成 ssh-key
```bash
ssh-keygen -t rsa -C "your_email@youremail.com"
```

弹出一下提问
我们不修改生成路径，不设置密码短语，直接回车	
生成成功之后，找到 刚刚升车 ssh-key 的路径
找到 id_rsa.pub 文件
右键-使用文本编辑器打开，里面有一大串的英文字母，不需要管，从头开始复制，直到最后
注意：最后的空格不需要 复制到（可能会引起一些问题）

然后就是去 github setting 里面 设置 ssh-key 了
具体参照 [如何创建公钥](https://gist.github.com/yisibl/8019693)

配置完成之后，测试一下

```bash
ssh -T git@github.com
```

如果返回 Hello <username> (username 表示你的用户名)
表示成功

##### 然后就是配置 Hexo 程序了
安装 hexo
```bash
npm install hexo-cli -g

npm install hexo --save
```

检查 hexo 是否安装成功

```bash
hexo -v
```

这个时候就不做 初始化了
这个时候就不做 初始化了
这个时候就不做 初始化了

自动安装需要的组件
```bash
npm install
```

安装 git 部署的 插件

```bash
npm install hexo-deployer-git --save
```

操作完成
现在这台新电脑 就跟原来那台电脑一样操作就可以发布文章了。