# Android 插屏广告

## 1. 推荐流程

1. 初始化成功后调用 `loadInterstitialAd()` 预加载。
2. 收到 `onInterstitialAdLoadedSuccess()` 后，广告进入可展示状态。
3. 展示前使用 `AdInterstitialReady()` 再次检查。
4. 页面处于前台时调用 `showInterstitialAd(Activity)`。
5. 广告关闭或下一次需要展示前，再调用 `loadInterstitialAd()`。

## 2. 设置插屏监听器

```java
import android.util.Log;

import com.tripleadflow.smartad.SmartedAdSdk;
import com.tripleadflow.smartad.interfacecallback.SmartedAdInterstitialEventListener;

SmartedAdSdk.setSmartedAdInterstitialEventListener(
        new SmartedAdInterstitialEventListener() {
            @Override
            public void onInterstitialAdLoadedSuccess() {
                Log.d(TAG, "插屏广告加载成功");
            }

            @Override
            public void onInterstitialAdLoadedFailed(String errorinfo) {
                Log.e(TAG, "插屏广告加载失败：" + errorinfo);
            }

            @Override
            public void onInterstitialAdImpression() {
                Log.d(TAG, "插屏广告产生曝光");
            }

            @Override
            public void onInterstitialAdShowedSuccess(
                    String platform,
                    String placementId) {
                Log.d(TAG, "插屏广告展示成功：" + platform + ", " + placementId);
            }

            @Override
            public void onInterstitialAdShowedFailed(
                    String platform,
                    String placementId,
                    String errorinfo) {
                Log.e(TAG, "插屏广告展示失败：" + errorinfo);
            }

            @Override
            public void onInterstitialAdClicked(
                    String platform,
                    String placementId) {
                Log.d(TAG, "插屏广告被点击：" + platform + ", " + placementId);
            }

            @Override
            public void onInterstitialAdClosed(
                    String platform,
                    String placementId) {
                Log.d(TAG, "插屏广告已关闭：" + platform + ", " + placementId);

                // 为下一次展示重新加载。
                SmartedAdSdk.loadInterstitialAd(getApplicationContext());
            }
        }
);
```

所有回调均运行在 Android 主线程。App 的加载、展示、曝光、点击和关闭回调按真实广告事件发送，不受内部归因策略开关影响。

## 3. 加载插屏广告

```java
SmartedAdSdk.loadInterstitialAd(getApplicationContext());
```

加载可以从任意线程发起。SDK 会把广告平台 API 调度到主线程，网络和数据处理继续在工作线程执行。

## 4. 判断广告是否就绪

```java
boolean ready = SmartedAdSdk.AdInterstitialReady();
```

## 5. 展示插屏广告

```java
private void showInterstitial() {
    if (isFinishing() || isDestroyed()) {
        return;
    }

    if (SmartedAdSdk.AdInterstitialReady()) {
        boolean accepted = SmartedAdSdk.showInterstitialAd(this);
        Log.d(TAG, "插屏展示请求是否被接受：" + accepted);
    } else {
        SmartedAdSdk.loadInterstitialAd(getApplicationContext());
    }
}
```

`showInterstitialAd()` 返回 `true` 只表示展示请求已被接受或成功投递，最终结果以展示成功或展示失败回调为准。展示时应传入当前可见且未销毁的 Activity。

## 6. 回调说明

| 回调 | 说明 |
| --- | --- |
| `onInterstitialAdLoadedSuccess()` | 广告加载成功，可以检查就绪状态并展示。 |
| `onInterstitialAdLoadedFailed(errorinfo)` | 广告加载失败。不要在回调中进行无延迟死循环重试。 |
| `onInterstitialAdImpression()` | 广告已经真实展示并产生曝光。 |
| `onInterstitialAdShowedSuccess(platform, placementId)` | 广告展示成功。当前 SDK 在曝光回调后发送此回调。 |
| `onInterstitialAdShowedFailed(platform, placementId, errorinfo)` | 没有可展示广告、Activity 不可用或平台展示失败。 |
| `onInterstitialAdClicked(platform, placementId)` | 用户点击广告。 |
| `onInterstitialAdClosed(platform, placementId)` | 广告被关闭。关闭后如需下一条广告，应重新加载。 |

`platform` 可能为 `max`、`admob` 或 `levelplay`；`placementId` 为本次实际使用的广告位 ID。
