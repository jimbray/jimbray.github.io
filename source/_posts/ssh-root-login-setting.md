---
title: VPS 禁止远程ROOT登录
date: 2016-06-09 12:48:17
tags:
---
### 编辑ssh 配置文件
```bash
vi /etc/ssh/sshd_config
```

### 修改关键配置 
```bash
PermitRootLogin no
```

### 重启 ssh 服务
```bash
service /etc/init.d/ssh restart
```
Done.