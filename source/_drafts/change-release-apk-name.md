---
title: test
tags:
---

android {}外 定义 获取时间的函数

```
def release_time() {
    return new Date().format("yyyy-MM-dd_HH-mm-ss")
}
```

更改apk 名称

```
    //修改release apk名称
    android.applicationVariants.all {
        variant -> variant.outputs.each {
            output -> def outputFile = output.outputFile
                if (outputFile != null && outputFile.name.endsWith('.apk')) {
                //这里修改apk文件名
                def fileName = outputFile.name.replace("app","Application-${defaultConfig.versionName}-${release_time()}")
                output.outputFile = new File(outputFile.parent, fileName)
            }
        }
    }
```

