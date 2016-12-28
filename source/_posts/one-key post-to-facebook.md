---
title: Android 一键发布 Facebook 帖文
date: 2016-12-28 13:00:00
tags: Android
---

一般遇到的需求都是做分享操作，现在国内已经有很多一键分享的 SDK了，而且还封装得不错，就是权限要得比较多。

今天需要做的是  跟 Path APP 一样，在自己的 APP 发布post时，同步发送到 ~~其他平台~~Facebook。

### 先把基本配置做了

> - 將 **Facebook Android SDK** 新增至您的行動開發環境
> - 取得正確設定並連結至 Android 應用程式的 **Facebook 應用程式編號**。請參閱 [Android 新手指南，新增 Facebook 應用程式編號](https://developers.facebook.com/docs/android/getting-started#app_id)
> - 產生 **Android 金鑰雜湊**並新增至[開發人員個人檔案](https://developers.facebook.com/settings/developer/contact/)
> - **新增 Facebook Activity** - 請將本項目加至 `AndroidManifest.xml

###  开始分享

#### 建立内容模板

模板有几种方式

* Link
* Photos
* Videos
* Multimedia

参见[Facebook官方文档](https://developers.facebook.com/docs/sharing/android?locale=en_US)

#### 最终使用

```
ShareApi.share(content, null);
```

即可发送成功

但是发送的样式 跟 使用那些 一键分享的 SDK 是一样的

都是那种感觉像 转发 的形式。（会看到 发送的对话框，类似于 ShareDialog）

<!-- more -->

#### 后台发送

仔细看了文档，发现了一个 [Graph API ](https://developers.facebook.com/docs/graph-api)

而且还提供了 [案例](https://developers.facebook.com/docs/graph-api/common-scenarios)，也有各个平台的SDK使用，用起来很方便。

##### Are two people Facebook friends? （判断两人是否为朋友）

* [`/{user-a-id}/friends/{user-b-id}`](https://developers.facebook.com/docs/graph-api/reference/user/friends/#readmodifiers) - this API Read modifier will let you check via API.

##### Publishing new Status Updates （发布新的近况）

- [`/{user-id}/albums`](https://developers.facebook.com/docs/graph-api/reference/user/albums/#publish) to create empty photo albums for people.
- [`/{user-id}/photos`](https://developers.facebook.com/docs/graph-api/reference/user/photos/#publish) to add individual photos for people.
- [`/{page-id}/albums`](https://developers.facebook.com/docs/graph-api/reference/page/albums#publish) to create empty photo albums for Facebook Pages.
- [`/{page-id}/photos`](https://developers.facebook.com/docs/graph-api/reference/page/photos/#publish) to add individual photos for Facebook Pages.
- [`/{album-id}/photos`](https://developers.facebook.com/docs/reference/api/album/#photos) to add photos to an existing album for people or for Pages.
- [`/{user-id}/videos`](https://developers.facebook.com/docs/graph-api/reference/user/videos/) to update an user's videos.
- [`/{page-id}/videos`](https://developers.facebook.com/docs/graph-api/reference/page/videos/) to update an page's videos.
- [`/{event-id}/videos`](https://developers.facebook.com/docs/graph-api/reference/event/videos/) to update an event's videos.

##### Sharing Links （分享链接）

- [`/{user-id}/feed`](https://developers.facebook.com/docs/graph-api/reference/user/feed/#publish) for people on Facebook, using the `link` field.
- [`/{page-id}/feed`](https://developers.facebook.com/docs/graph-api/reference/page/feed/#publish) for Facebook Pages, using the `link` field.



等等......具体参见[文档](https://developers.facebook.com/docs/graph-api/common-scenarios)

### 权限申请

根据需求写了一个方法，用于发布 文字近况 和 图片近况

```
public static void share2FBInBackground(String img_url, String des) {
        if(TextUtils.isEmpty(img_url)) {
            if(!TextUtils.isEmpty(des)) {
                Bundle params = new Bundle();
                params.putString("message", des);
                /* make the API call */
                new GraphRequest(
                        AccessToken.getCurrentAccessToken(),
                        "/me/feed",
                        params,
                        HttpMethod.POST,
                        new GraphRequest.Callback() {
                            public void onCompleted(GraphResponse response) {
                                DebugUtils.LOG_D(TAG, "post text complete");
                                /* handle the result */
                            }
                        }
                ).executeAsync();
            }

        } else {
            Bundle params = new Bundle();
            params.putString("url", img_url);
            if(!TextUtils.isEmpty(des)) {
                params.putString("caption", des);
            }
            /* make the API call */
            new GraphRequest(
                    AccessToken.getCurrentAccessToken(),
                    "/me/photos",
                    params,
                    HttpMethod.POST,
                    new GraphRequest.Callback() {
                        public void onCompleted(GraphResponse response) {
                        /* handle the result */
                            DebugUtils.LOG_D(TAG, "post photos complete");
                        }
                    }
            ).executeAsync();
        }
    }
```

可是

每次都不成功！

原来 Facebook 发布还需要权限，还需要申请（publish_actions）

那就在 发布之前 加一个 判断吧

是否有权限发布

```
public static boolean hasPublishPermission() {
    AccessToken accessToken = AccessToken.getCurrentAccessToken();
    return accessToken != null && accessToken.getPermissions().contains("publish_actions");
}
```

那，怎么申请权限呢？

```
public static void loginWithPublishPermission(Activity activity, final 			FacebookCallback<LoginResult> callback) {
        // 允许使用的权限
        String permission[] = new String[]{"publish_actions"};
        LoginManager.getInstance().logInWithPublishPermissions(activity, Arrays.asList(permission));
        LoginManager.getInstance().registerCallback(mCallbackManager, callback);
    }
```

做了做些操作，应该可以发布成功了。

但是， `publish_actions` 这个权限是需要申请并审核的。

那就去申请好了。

进入 开发者账号--选中应用--选择 App-Review --选择 Submit Items for Approval--添加 需要的权限

然后 Edit Notes 看下 审核需要做哪些事情。

可严格了

* My app does not auto-populate the user message field
* My app does not use the Facebook [Feed Dialog](https://developers.facebook.com/docs/sharing/reference/feed-dialog/) or the Facebook [Share Dialog](https://developers.facebook.com/docs/sharing/reference/share-dialog) to publish content
* The share button has clear Facebook branding
* My app does not post to Facebook automatically

还要录制一个视频

You must show:

1. How a person starts your app and logs in to Facebook login
2. How a person shares to Facebook from your app
3. How the post looks on the person's Facebook timeline after posting

比写代码还麻烦。

还有一个问题，我想测试我写的代码是否能发布出去呢，现在权限还没有申请下来。

没关系，facebook 提供了一个 测试账号（需要自己添加）

左侧边栏--Roles--Test Users 

然后使用新增的用户，用刚才的代码，就可以进行发布测试了。

流水帐式记录。Done.