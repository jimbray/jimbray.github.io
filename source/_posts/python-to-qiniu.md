---
title: 简化markdown写作中的贴图流程（windows简陋版）
date: 2016-06-14 23:59:54
tags: python
---
上次在网上闲逛的时候看到一篇文章

[简化markdown写作中的贴图流程](http://www.jianshu.com/p/7bd4e6ed99be/comments/1794317#comment-1794317)

刚好最近也在学习python，看起来好有趣啊，这篇文章讲的是Mac上的，
我没有Mac，只有Windows，文章里面也写了Windows系统的实现思路，既然作者已经给出来了，作为一个python初学者，动动手罢

### 思路
1. 截图（保存在剪贴板中）
2. 使用七牛sdk 上传剪切板中的图片
3. 获取返回的url，并合成 markdown 贴图语法
应该就可以了吧

### 实现
#### 准备工作
1. 安装 七牛python sdk,我用的是IDE pycharm 方式
![img_1465917613.88.jpg](http://7xv11f.com1.z0.glb.clouddn.com/img_1465917613.88.jpg)
2. 下载安装 [AutoHotKey](https://autohotkey.com/download/) 软件
3. 写 python代码
4. 写AutoHotKey脚本

#### 开始动手
写代码
由于技术太渣，搜到又看不懂win32什么的，最后获取剪贴板截图的功能没能写出来，找到了另一个库 PIL
这个库的截图也是在剪切板的，不过不是其他软件接下来的，是系统截下来的 screencapture 

```python
AK = 'access_key'
SK = 'secret_key'
bucket_name = "bucket_name"
buckey_url = {
    'bucket_name': 'domain_name',
}
```
上面是一些初始化工作
都是七牛的东西
可以在 七牛的 开发者帐号里面找到
access_key 和 secret_key 就不说了
bucket_name 是 空间名称
domain_name 是 空间域名

<!-- more -->
继续:获取图片
```python
def save_clipboard_image():
    pic = ImageGrab.grab()
    saved_path = "img_" + str(time.time()) + ".jpg";
    pic.save(saved_path)
    return saved_path
```
获取系统截图，然后保存在一个文件名为当前时间的jpg文件中
返回保存的文件路径

继续：上传图片
```python
class Qiniu():
    def __init__(self, ak, sk):
        self.access_key = ak
        self.secret_key = sk
        self._q = qiniu.Auth(self.access_key, self.secret_key)

    def upload_file(self, bucket_name, key, file_path):
        #生成上传 Token，可以指定过期时间等
        token = self._q.upload_token(bucket_name, key, 3600)
        ret, info = put_file(token, key, file_path)
        print(info)
        pic_url = self.get_pic_url(bucket_name, file_path)
        add_to_clipboard(pic_url, file_path)

    def get_pic_url(self, bucket_name, file_path):
        result = buckey_url[bucket_name] + file_path
        return result
```
上传图片的 功能都是 七牛sdk 提供的，sdk demo里面都有，也没什么好说的了
上面有个 add_to_clipboard 方法
```python
def add_to_clipboard(pic_url, file_name):
    txt = '![' + file_name +']('+ pic_url +')'
    print(txt)
    command = 'echo ' + txt.strip() + '| clip'
    os.system(command)
```
工作就是把上传完成之后的图片路径 转换成 markdown 语法，并复制到剪贴板
还剩一个入口函数
```python
if __name__ == '__main__':
    saved_path = save_clipboard_image()
    q = Qiniu(AK, SK)
    q.upload_file(bucket_name, saved_path, saved_path)
```
需要导入的库
```python
from PIL import ImageGrab
import time
from qiniu import Auth, put_file
import os
```
好像就写完了
运行以下，去七牛空间上看，就会多一张图片，以一串数字命名
然后新建一个 txt 文档，ctrl-v 以下，就能看到一个字符串啦，是在markdown插入图片的语法

继续：设置触发
接下来要上场的是 windows 下的 神器 AutoHotKey，我也是看前面那个文章才知道的
工作主要是 触发这个python文件的执行
那我就偷懒了一下
使用 ctrl+alt+v触发python

```ahk
^!v::
run,%comspec% /c python D:\python\upload.py
```
^代表ctrl键
!代码alt键
v代表v键（废话）
工作就是按键之后，执行 cmd 然后输入命令
python D:\python\upload.py
运行起来
执行一下，上传成功，可是每张图片都被 cmd 那个黑框框挡住了
google了以下
答案是
```ahk
run,%comspec% /c python D:\python\upload.py ,, hide
```
后面加上一个 hide
就不会出现黑框框了。
到这里应该就是把这个功能的简陋版弄出来了
等之后有时间了，学习更多了，在优化以下，减少硬编码

我的第一个 python 功能就这么完成了。
哈。
试试效果

![img_1465919301.23.jpg](http://7xv11f.com1.z0.glb.clouddn.com/img_1465919301.23.jpg)

快12点了
早睡早起身体好。