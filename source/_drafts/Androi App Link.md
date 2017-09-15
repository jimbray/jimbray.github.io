https://github.com/nordnet/cordova-universal-links-plugin/issues/88

官网链接 https://developer.android.com/training/app-links/deep-linking.html
https://firebase.google.com/docs/app-indexing/android/public-content
1. 添加 intent filter 来响应进来的 链接
2. deeplink 

    <intent-filter android:autoVerify="true">
        <action android:name="android.intent.action.VIEW"/>
        <category android:name="android.intent.category.DEFAULT"/>
        <category android:name="android.intent.category.BROWSABLE"/>
        <data android:scheme="http" android:host="cloud-mypoof.com"/>
        <data android:scheme="https"/>
        <data android:host="cloud-mypoof.com"/>
        <data android:host="www.cloud-mypoof.com"/>
        <data android:pathPrefix="/web_register"/>
        <!--<data android:pathPrefix="/v"/>
        <data android:pathPrefix="/bitmoji"/>
        <data android:pathPrefix="/discover"/>
        <data android:pathPrefix="/unlock"/>
        <data android:pathPrefix="/odg"/>
        <data android:pathPrefix="/partner"/>
        <data android:pathPrefix="/search"/>
        <data android:pathPrefix="/share"/>
        <data android:pathPattern="/xxx/..*"/>-->
    </intent-filter>

3. 测试 deeplink 
adb shell am start-W -a android.intent.action.VIEW -d http://cloud-mypoof.com com.poofer.poof

成功
这个时候 点击 cloud-mypoof.com 不会弹出 使用哪个浏览器 的 choose dialog 了
但是 还是会弹出 一个 open with 的选项
而 一个 Android App link 就是一个 基于网站认证的 deeplink
不应该是选择，应该直接跳转到 POOF 

4. Android App Link 需要做多一些事情
a. intent-filter 加入 android:autoVerify="true" 
b. 上传 assetlinks.json 到 https://hostname/.well-known/assetlinks.json 路径下
生成 assetlinks.json 的方法

https://developers.google.com/digital-asset-links/tools/generator

App package fingerprint 生成命令

	keytool -list -v -keystore my-release-key.keystore

上面的网址还可以 进行检测

5. 发布 assetlinks.json 文件
需要做到以下

* The assetlinks.json file is served with content-type application/json.
* The assetlinks.json file must be accessible over an HTTPS connection, regardless of whether your app's intent filters declare HTTPS as the data scheme.
* The assetlinks.json file must be accessible without any redirects (no 301 or 302 redirects) and be accessible by bots (your robots.txt must allow crawling /.well-known/assetlinks.json).
* If your app links support multiple host domains, then you must publish the assetlinks.json file on each domain. See Supporting app linking for multiple hosts.
* Do not publish your app with dev/test URLs in the manifest file that may not be accessible to the public (such as any that are accessible accessible only with a VPN). A work-around in such cases is to configure build variants to generate a different manifest file for dev builds.
Test app links

以上我都做了，但是用命令验证的时候

    adb shell dumpsys package d

可以看到 
 App Verification  status
 com.poofer.poof status 为 undefined

> it shows up under "App verification status" header but status is "undefined", it doesn't show up under "App linkages for user 0" header at all.


https://developer.android.com/studio/write/app-link-indexing.html
使用 android studio 2.3 简直方便得不行

