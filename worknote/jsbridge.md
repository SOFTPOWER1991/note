**本文主要记录易项优选Android端原生和H5的交互实现方式，主要聚焦于如何去做。**


原生和 H5交互的场景，大致可以分为两类：

1. 从原生页面跳转到H5页面，展示一个网页；
2. 从H5传递给原生一些数据，通知原生进行相应操作。

上述这两种场景，在易项优选Android 端的两处具体体现：

1. 首页广告栏点击，打开一个网页；
2. 首页专区，点击数据排行榜跳转到网页；点击网页的item后，跳转到原生页面。

这里仅针对第二种具体场景进行分析，因为它包含了原生和H5交互的两种方式。

### 场景一: 从原生跳转到 H5（点击首页**数据排行榜**专区，跳转到数据排行版的页面）
 
 代码位于ColumnHolder.java中，代码为：
 ```
 UIHelper.startActivity(
        Constants.BundleKeys.BUNDLE_KEY_WEB_URL,
        columnGridInfos.get(position).getData().getLinkUrl(),
        Constants.RouterPath.MAIN_WEBVIEW_FOR_JS,
        -1,
        mContext);
 ```
 
其中：

* 第二个参数columnGridInfos.get(position).getData().getLinkUrl()代表要跳转到的网页地址
* 第三个参数代表要打开的页面。承载网页展示功能的Activity为：WebViewForJsActivity.java中。

从Bundle中拿到url后，通过
```
mWebView.loadUrl(url)
```
就可以打开一个网页。


### 场景二：从H5 跳转到 原生页面（点击网页的item后，跳转到原生页面）

在开始之前，不妨猜测一下，H5要让原生进行相应的操作，必须要告诉原生如下信息：

1. 做什么操作；
2. 完成这些操作需要的数据；

这些数据在CommonWVJBClient.java中，H5会通过JsBridge告诉原生。

具体实现为：

``` java
registerHandler("startNativePage", new WVJBWebViewClientBase.WVJBHandler() {

    @Override
    public void request(Object data, WVJBResponseCallback callback) {
        boolean isSuccess = UIHelper.startActivityFromJs(mContext, data);
        if (isSuccess) {
            callback.callback("jump success");
        }
    }
});
```
断点跟踪了下发现H5传过来的data数据为：

```
{
	"PageName":"ProjectTopListPage",
	"Params":
		{
			"Type":"meeting_rate"
		}
}
```

哈，这是一个典型的Json数据啊，这个data传递给了`UIHelper.startActivityFromJs(mContext, data)
`，跟踪进去，一探究竟。

代码如下（为了说明问题，精简了代码）
```
public static boolean startActivityFromJs(Context context, Object data) {
        boolean isJump = false;
        if (data != null && data instanceof JSONObject) {
            try {
            		// 要跳转到的页面
                String activityName = ((JSONObject) data).optString("PageName");
                // 解析需要的参数
                JSONObject params = ((JSONObject) data).optJSONObject("Params");

                if (params != null) {
                    String type = params.optString("Type");
                    String subType = params.optString("Subtype");
                    if (!TextUtils.isEmpty(type) && !TextUtils.isEmpty(subType)) {
                        DetectorHelper.getInstance(context).buildDetectorInfo(type, subType);
                    } else if (!TextUtils.isEmpty(type)) {
                        DetectorHelper.getInstance(context).buildDetectorInfo(type);
                    }
                }

                if (!TextUtils.isEmpty(activityName)) {
                    switch (activityName) {
                        // 找到要跳转的页面后，携带具体数据，前往目的地。
                        case Constants.NativeActivityName.PROJECT_TOP:
                            if (params != null) {
                                String type = params.optString("Type");
                                Bundle mbundle = new Bundle();
                                mbundle.putString("TYPE",type);
                                startCertificationActivity(mbundle, Constants.RouterPath.MAIN_TOP_PROJECT_LIST, -1, context);
                            }
                            break;
                      
                    }
                }
            } catch (Exception e) {
                Log.i("jump native", e.toString());
            }
        }
        return isJump;
    }
```
在这个方法中解析Json拿到如下数据：

>1. 要跳转的页面：String activityName = ((JSONObject) data).optString("PageName");
>2. 解析传递的参数:JSONObject params = ((JSONObject) data).optJSONObject("Params");

然后把这些参数传递给要跳转的页面：
```
 String type = params.optString("Type");
 Bundle mbundle = new Bundle();
 mbundle.putString("TYPE",type);
 startCertificationActivity(mbundle, Constants.RouterPath.MAIN_TOP_PROJECT_LIST, -1, context);
```

至此，就从H5页面跳转到了原生页面。

# 小结

1. 从原生跳转的H5该如何做？

	> 传递给WebViewForJsActivity.java 要展示的H5页面的url

	代码示例：

	```
	UIHelper.startActivity(Constants.BundleKeys.BUNDLE_KEY_WEB_URL,columnGridInfos.get(position).getData().getLinkUrl(),
        Constants.RouterPath.MAIN_WEBVIEW_FOR_JS,
        -1,mContext);

	```

2. H5通知原生做相应的操作该如何做？

	> 在CommonWVJBClient中的构造方法中注册相应的Handler，并根据与H5的协定完成的相应操作。

	代码示例：
	
	```java
	registerHandler("startNativePage", new WVJBWebViewClientBase.WVJBHandler() {

    @Override
    public void request(Object data, WVJBResponseCallback callback) {
        boolean isSuccess = UIHelper.startActivityFromJs(mContext, data);
        if (isSuccess) {
            callback.callback("jump success");
        }
    }
});

	```

至此，完成了H5和原生交互逻辑的梳理。至于其中的JsBridege可以单独写一篇了，下回见。

