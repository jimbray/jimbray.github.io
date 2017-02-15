---
title: 最简单方式拥有自己的SS服务器
date: 2017-02-15 12:20:20
tags: [Shadowsocks, VPS]
---

### 购买VPS

我买的是 [openvzvps-小尾巴](https://www.50vz.net/aff.php?aff=656) 国人弄的,虽然配置低点，但我又不用来做其他东西，就搭个ss，够用就行。
虽然带了小尾巴，但是为了避免广告嫌疑，我就不贴图了

### 安装shadowsocks

```
$ sudo apt-get update
$ sudo apt-get install python-gevent python-pip
$ sudo pip install shadowsocks
$ apt-get install python-m2crypto
```



### 配置shadowsocks


创建 shadowsocks.json 配置文件

```
vi /etc/shadowsocks.json
```



修改shadowsocks.json 配置文件

```
{
"server":"0.0.0.0",
"server_port":8388,
"local_port":1080,
"password":"password",
"timeout":600,
"method":"aes-256-cfb"
}
```

填写自定义的端口和密码就可以了

### 运行shadowsocks

```
ssserver -c /etc/shadowsocks.json -d start

ssserver -c /etc/shadowsocks.json -d stop
```

就搞定了。
一劳永逸，自从开了之后就没动过它。一直正常。
看了一下Youtube4K视频，略卡，不过1080无压力，日常使用完全没问题。
![](http://7xv11f.com1.z0.glb.clouddn.com/SSSPEED.png)