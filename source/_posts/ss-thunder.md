---
title: 使用迅雷下载后 shadowsocks失效 解决办法
date: 2016-06-09 12:08:50
tags: Shadowsocks
---

今天用的好好的 ss 突然之间失效了，看了下日志：

```bash
	failed to recv data in handshakeReceive2Callback
```

网上搜罗下，出现这个情况的不在少数，主要是因为使用了迅雷下载导致

### 解决办法

1.在服务中找到XLServicePlatform把这个服务禁用了。现在应该可以使用ss了

2.重启电脑后发现，ss又不能用了，继续查看服务，XLServicePlatform又起来了，还把启动方式变成了自动。

3.关掉 服务，打开 C:\Program Files\Common Files\Thunder Network\ServicePlatform\XLSP.dll ，把里面的内容清空

问题解决。

XLServicePlatform：这个服务主要的功能是迅雷的网络诊断，并不是迅雷下载的基础服务，应该不会影响迅雷的下载使用