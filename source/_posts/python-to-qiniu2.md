---
title: 简化markdown写作中的贴图流程（windows基本版）
date: 2016-06-19 21:12:36
tags: python
---
接上一篇文章 [简化markdown写作中的贴图流程（windows简陋版）](http://jimbray.xyz/2016/06/14/python-to-qiniu/)

上一篇文章是 做了很久，除了对python不熟，还有懒，所以最终的结果不尽人意，可以说做出来一个半成品。能用，但用起来不爽。
所以，现在完成了基本版，基本版基本可用了

上次的版本是 获取 全屏截图，这样的话像QQ截图这些功能都用不上了，而且还是显示很多不必要的元素在截图里面，比如说下面这个图啊
![img_1466342265.26.jpg](http://7xv11f.com1.z0.glb.clouddn.com/img_1466342265.26.jpg)
箭头所指的地方，根本不应该出现在截图里面的，是多余的元素。
废话不多说，开干。
思路跟实现跟上次一点区别都没有
代码也只是修改了一个函数，其他的基本没有变化
主要记录遇到的几个问题

### 代码
先把代码贴出来，再上问题
```python
def save_clipboard_image():
    im = ImageGrab.grabclipboard()
    if im is None:
        print('Error: No image data in clipboard')
        return None

    if isinstance(im, Image.Image):
        print im.format, im.size, im.mode
        width, height = im.size
        saved_path = "img_" + str(time.time()) + ".jpg"
        # im.resize((30, 30), Image.ANTIALIAS).load()
        im.save(saved_path)

        print saved_path
        return saved_path
    else:
        print 'the object in clipboard is not a image.'
        return None
```

这个函数是修改的，之前是获取全屏截图，现在修改为获取剪贴板中的图片
#### 遇到的问题
##### 问题1
```bash
Unsupported BMP bitfields layout
```
这个是 Pillow 的bug，搜索到解决办法：
<!-- more -->
作者：青南
链接：[https://zhuanlan.zhihu.com/p/21303461](https://zhuanlan.zhihu.com/p/21303461)
来源：知乎

这个问题从Pillow 2.8.0开始，一直到3.2.0都没有被官方解决。目前有一个间接的解决办法。
请打开Python安装目录下的\Lib\site-packages\PIL\BmpImagePlugin.py文件，将以下代码：
```python
if file_info['bits'] in SUPPORTED:
    if file_info['bits'] == 32 and file_info['rgba_mask'] in SUPPORTED[file_info['bits']]:
        raw_mode = MASK_MODES[(file_info['bits'], file_info['rgba_mask'])]
        self.mode = "RGBA" if raw_mode in ("BGRA",) else self.mode
    elif file_info['bits'] in (24, 16) and file_info['rgb_mask'] in SUPPORTED[file_info['bits']]:
        raw_mode = MASK_MODES[(file_info['bits'], file_info['rgb_mask'])]
    else:
        raise IOError("Unsupported BMP bitfields layout")
else:
    raise IOError("Unsupported BMP bitfields layout")
```
修改为：
```python
if file_info['bits'] in SUPPORTED:
    if file_info['bits'] == 32 and file_info['rgba_mask'] in SUPPORTED[file_info['bits']]:
        raw_mode = MASK_MODES[(file_info['bits'], file_info['rgba_mask'])]
        self.mode = "RGBA" if raw_mode in ("BGRA",) else self.mode
    elif file_info['bits'] in (24, 16) and file_info['rgb_mask'] in SUPPORTED[file_info['bits']]:
        raw_mode = MASK_MODES[(file_info['bits'], file_info['rgb_mask'])]
    '''新增内容开始'''
    elif file_info['bits'] == 32 and file_info['rgb_mask'] == (0xff0000, 0xff00, 0xff):
        pass
    '''新增内容结束'''
    else:
        raise IOError("Unsupported BMP bitfields layout")
else:
    raise IOError("Unsupported BMP bitfields layout")
```
就能解决本问题。

##### 问题2
```bash
IOError: image file is truncated (1044 bytes not processed)
```
搜索到解决办法 [Python：IOError: image file is truncated 的解决办法](http://www.cnblogs.com/hongfei/p/4436767.html)
```python
from PIL import ImageFile
ImageFile.LOAD_TRUNCATED_IMAGES = True
```

问题解决。在 ahk脚本中修改以下 新python文件的路径就可以愉快地玩耍了

```ahk
^!v::
run,%comspec% /c python D:\python\clipboard_to_qiniu.py
```

reload 以下 ahk 脚本，使用方法跟之前一模一样，不过现在支持像QQ这类切图软件了。

### 改进
每次截图之后，就会发现 python文件所在文件夹就会多一张图片，都是已经上传到七牛的了，留下来没有什么用处，还浪费我空间。
那就再加一个逻辑吧：上传到七牛后删除本地的图片

```python
def deleteLocalImage(path):
    if(os.path.isfile(path)):#文件存在
        os.remove(path)
```

主函数也修改以下
```python
if __name__ == '__main__':
    saved_path = save_clipboard_image()
    if saved_path == None:
        pass
    else:
        q = Qiniu(AK, SK)
        q.upload_file(bucket_name, saved_path, saved_path)
        deleteLocalImage(saved_path)
```

搞定收工。
这次用起来比较普通了。
![img_1466343760.76.jpg](http://7xv11f.com1.z0.glb.clouddn.com/img_1466343760.76.jpg)
