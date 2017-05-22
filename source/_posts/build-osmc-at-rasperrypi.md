---
title: 使用树莓派搭建OSMC家庭影院
date: 2017-05-22 12:39:36
tags: 树莓派
---
手上有一台Raspberry Pi 3 ，为了避免吃灰，还是折腾一下好了。
 选个系统安装一下，~~经过长时间的纠结~~ 
选择了 OSMC,还是插电视用吧。

### 安装OSMC 系统

1.  在 [osmc官网][1]先下载镜像
2.  因为是在 win 下干活的，所以用的是 [win32diskimager][2]
3.  直接选择 osmc img文件烧录就行了。
4.  整个过程都是自动的，完成之后就会进入 系统了

![][3] ![][4]

### 修改系统语言为中文

1.  找到 system–>settings–>apparence–>skin，把 fonts 改成 Arial based；
2.  找到 skin 下面的 international，把 language 改成 Chinese； 一定要先修改字体，否则的话切换成中文之后会乱码。

### 不识别NTFS硬盘

系统安上了，也有中文了，手里移动硬盘插上去，准备看电影了。 但是插上去死活没反应，我硬盘不是坏了吧。 Google 老师告诉我，OSMC 不认识我的硬盘。（不识别NTFS硬盘）

1.  安装 ntfs-3g ，让系统支持NTFS硬盘 `sudo apt-get install ntfs-3g` 安装完成之后重启

2.  查看插入的硬盘对应的分区名称 `sudo fdisk -l | grep NTFS` 我硬盘里放电影的分区 对应的是 /dev/sda2

3.  挂在指定的分区到系统 把硬盘分区挂在到指定的文件夹（可以新建） 比如我就新建了一个文件夹 叫 Movies

`sudo mount -t ntfs-3g /dev/sda2 /home/osmc/Movies` 现在应该可以愉快地观看硬盘的视频了。
<!-- more -->

### 开机自动挂载硬盘

但是每次重启都发现 硬盘又要手动挂载一遍，真麻烦。 那就设置自动挂载吧，反正这个硬盘就是拿来看电影的了。

`sudo vi /etc/fstab`

在尾部加上 `/dev/sda2 /home/osmc/Movies/ auto defaults 0 0`

### 安装迅雷远程下载

硬盘里的电影虽然很多，但总有一天会看完的，我可不想坐吃山空。 迅雷会员也买了，作为一个互联网服务，我觉得还是很值得的。 迅雷有一个远程下载的 Xware 服务，可以在 [这里][5] 下载最新版 我下载的是 [Xware1.0.31_armel_v5te_glibc.zip][6] 这个版本 用SSH 登录到 OSMC

1.  解压Xware1.0.31_armel_v5te_glibc.zip后的文件放在 指定目录里面 `unzip Xware1.0.31_armel_v5te_glibc.zip` 我是新建了一个 /xunlei 文件夹
2.  然后修改 /xunlei 文件夹的权限 `chmod 777 * -R /xunlei`
3.  进入/xunlei文件夹，进行激活操作 `./portal` 就会得到一个迅雷远程下载的 激活码，然后去 [迅雷][7] ，激活下载机。 这样，OSMC 就是一台远程下载机器了。

### 安装ftp

硬盘放着也是放着，也可以做一下文件的中转站 `sudo apt-get install vsftpd` 然后进行编辑 `sudo vi /etc/vsftpd.conf` 找到以下行，定义一下 `anonymous_enable=NO 表示：不允许匿名访问
local_enable=YES 设定本地用户可以访问。
write_enable=YES 设定可以进行写操作
local_umask=022 设定上传后文件的权限掩码。` 新增一行 `local_root=/home/osmc/Movies` `sudo service vsftpd restart` 完成，在Android上用ES文件浏览器配合使用 还有 MX Player ，可以在手机上播放 硬盘里的视频了。

### 安装 samba

既然是个 服务器，又可以放电影，那手机上也想看硬盘上的电影，这个时候，samba装起来 `sudo apt-get install samba` samba相关支持工具安装 `sudo apt-get install samba-common-bin` 配置samba `/etc/samba/smb.conf` 先备份smb.conf文件 `cp smb.conf cmb.conf.bak` 修改smb.conf文件 `sudo nano smb.conf` 我的文件配置是： 
``` 
[global] 
log file = /var/log/samba/log.%m 
[movie] 
comment = Temporary file space path = /home/pi
read only = no 
public = yes

```

增加samba用户 `sudo smbpasswd -a pi，#输入两次密码` 重启samba服务 `sudo service samba restart` 如果不行，使用 `sudo systemctl restart smbd.service` 验证 在windows开始中输入：192.168.1.111看你osmc具体分配到的地址 输入密码（如果需要进行ssh连接，最好在设置里面设置静态IP） ， 就能访问共享的文件夹了

这样一个完整的 家庭影院系统就完成了。 
1. 可以在任意地点远程下载电影 

2. 可以在电视上直接播放下载完成的电影 

3. 可以在手机上直接播放下载完成的电影 

4. 基于OSMC自带的UPNP服务，还可以进行手机视频的投放 

5. 可以做一个文件中转站，抛弃数据线，各端传输数据

折腾完毕，还有地方未完善 迅雷远程下载服务没有开机自启，后续完善，因为到现在还没关机过。

 [1]: https://osmc.tv/
 [2]: https://sourceforge.net/projects/win32diskimager/
 [3]: http://7xv11f.com1.z0.glb.clouddn.com/formatting.jpg
 [4]: http://7xv11f.com1.z0.glb.clouddn.com/installing.jpg
 [5]: http://luyou.xunlei.com/forum-51-1.html
 [6]: http://luyou.xunlei.com/forum.php?mod=attachment&aid=MjUxNDB8NGQ1OWUzZmN8MTQ4ODcxNzg0MnwxMjk3ODl8MTI1NDU%3D
 [7]: http://yuancheng.xunlei.com/