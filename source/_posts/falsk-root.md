---
title: Flask 直接显示根目录文件内容
date: 2016-06-09 12:37:25
tags: python
---
### 搜索结果

```python
@app.route('/<path>')
def info(path):
    resp = make_response(open(path).read())
    resp.headers["Content-type"]="application/json;charset=UTF-8"
    return resp
```

按道理应该是可以生效的，但我在用的时候却报错了

### 出现错误

```bash
IOError: [Errno 2] No such file or directory: u'readme.json'
```

居然找不到这个文件

### 解决办法
使用绝对路径
```python
@app.route('/<path>')
def today(path):
    base_dir = os.path.dirname(__file__)
    resp = make_response(open(os.path.join(base_dir, path)).read())
    resp.headers["Content-type"]="application/json;charset=UTF-8"
    return resp
```	

访问 127.0.0.1:5000/readme.json

Done.