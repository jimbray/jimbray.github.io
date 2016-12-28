---
title: Facebook ShareDialog 没有回调的问题
date: 2016-06-10 20:24:04
tags: Android
---

完成 facebook 分享功能之后，如果手机安装了Facebook，分享之后会有成功或者失败的 反馈

但是如果没有安装，就会使用 WEB 版本的 进行分享，这个时候，分享完了就是完了，不知道是成功还是失败。

去看了下文档，Facebook提供这个回调。

叫做 CallbakManager ，既然找到了，就上手吧

```java
ShareDialog shareDialog = new ShareDialog(act);

CallbackManager callbackManager = CallbackManager.Factory.create();
shareDialog.registerCallback(callbackManager, new FacebookCallback<Sharer.Result>() {
	@Override
	public void onSuccess(Sharer.Result result) {
		Log.d("tag", "OnSuccess");
	}

	@Override
	public void onCancel() {
		Log.d("tag", "onCancel");
	}

	@Override
	public void onError(FacebookException error) {
		Log.d("tag", "onError");
	}
});

ShareLinkContent content = new ShareLinkContent.Builder()
		.setContentUrl(Uri.parse("http://url.com"))
		.setImageUrl(Uri.parse("http://url.com"))
		.setContentTitle("Title")
		.setContentDescription("description")
		.build();

shareDialog.show(content, ShareDialog.Mode.AUTOMATIC);
```

看上去就是这样的，可是每次都没有回调
最后去Facebook 文档又看了一遍，找到解决办法了，原来之前没有看清楚

Facebook share封装起来之后是用 Activity 做回调的
所以 纯粹的使用 CallbackManager 肯定不会生效啦。

<!-- more -->
每个Activity需要对应的 CallbackManager 
然后需要 override onActivityResult 进行callbackManager 的调用

```java
@Override
protected void onActivityResult(int requestCode, int resultCode, Intent data) {

	callbackManager.onActivityResult(requestCode, resultCode, data);//这个callbackmanager要是之前 create 的那一个，不能是 新 create的
}
```

所以写了一个 辅助类 FacebookHelper

```java
public class FacebookHelper {

    private static final String TAG = FacebookHelper.class.getSimpleName();

    private static CallbackManager mCallbackManager;

    public static CallbackManager initCallbackManager(Activity activity) {
        if (!FacebookSdk.isInitialized()) {
            FacebookSdk.sdkInitialize(activity);
        }

        return mCallbackManager = CallbackManager.Factory.create();
    }

	/**
	* url 需要分享的 url
	* des 需要分享的 description
	**/
    public static void showLink2Facebook(Activity act, String url, String des) {
        if(TextUtils.isEmpty(url)) {
            return ;
        }
        ShareDialog shareDialog = new ShareDialog(act);

        shareDialog.registerCallback(mCallbackManager, new FacebookCallback<Sharer.Result>() {
            @Override
            public void onSuccess(Sharer.Result result) {
                Utils.showToast("share successful");
            }

            @Override
            public void onCancel() {
                Utils.showToast("cancel the share operation");
            }

            @Override
            public void onError(FacebookException error) {
                Utils.showToast("share failed. you can try again later");
            }
        });

        ShareLinkContent content = new ShareLinkContent.Builder()
                .setContentUrl(Uri.parse(url))
                .setImageUrl(Uri.parse(url))
                .setContentTitle("Title)
                .setContentDescription(des)
                .build();

        shareDialog.show(content, ShareDialog.Mode.AUTOMATIC);
    }
}
```

只要在每个需要分享的 Activity OnCreate 时 获取一个 CallbackManager 成员变量

```java
mCallbackManager = FacebookHelper.initCallbackManager(this);
```

然后 重载 onActivityResult 方法中调用 就可以了

```java
mCallbackManager.onActivityResult(requestCode, resultCode, data);
```

为了这事捣鼓了半天，都怪自己文档没看仔细。
看到CallbackManager 就 立马动手写代码了。
以后文档看完再动手，切记。
不然，掉坑。