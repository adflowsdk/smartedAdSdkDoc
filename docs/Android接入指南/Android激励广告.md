# Android 激励广告

## 1. 推荐流程

1. 初始化成功后调用 `loadRewardAd()` 预加载。
2. 收到 `onRewardAdLoadedSuccess()` 后，广告进入可展示状态。
3. 展示前使用 `AdRewardReady()` 再次检查。
4. 页面处于前台时调用 `showRewardAd(Activity)`。
5. 广告关闭或下一次需要展示前，再调用 `loadRewardAd()`。

## 2. 设置激励广告监听器

```java
import android.util.Log;

import com.tripleadflow.smartad.SmartedAdSdk;
import com.tripleadflow.smartad.interfacecallback.SmartedAdRewardEventListener;

SmartedAdSdk.setSmartedAdRewardEventListener(
        new SmartedAdRewardEventListener() {
            @Override
            public void onRewardAdLoadedSuccess() {
                Log.d(TAG, "激励广告加载成功");
            }

            @Override
            public void onRewardAdLoadedFailed(String errorinfo) {
                Log.e(TAG, "激励广告加载失败：" + errorinfo);
            }

            @Override
            public void onRewardAdImpression() {
                Log.d(TAG, "激励广告产生曝光");
            }

            @Override
            public void onRewardAdShowedSuccess(
                    String platform,
                    String placementId) {
                Log.d(TAG, "激励广告展示成功：" + platform + ", " + placementId);
            }

            @Override
            public void onRewardAdShowedFailed(
                    String platform,
                    String placementId,
                    String errorinfo) {
                Log.e(TAG, "激励广告展示失败：" + errorinfo);
            }

            @Override
            public void onRewardAdClicked(
                    String platform,
                    String placementId) {
                Log.d(TAG, "激励广告被点击：" + platform + ", " + placementId);
            }

            @Override
            public void onRewardAdClosed(
                    String platform,
                    String placementId) {
                Log.d(TAG, "激励广告已关闭：" + platform + ", " + placementId);

                // 为下一次展示重新加载。
                SmartedAdSdk.loadRewardAd(getApplicationContext());
            }
        }
);
```

所有回调均运行在 Android 主线程。App 的加载、展示、曝光、点击和关闭回调按真实广告事件发送，不受内部归因策略开关影响。

## 3. 加载激励广告

```java
SmartedAdSdk.loadRewardAd(getApplicationContext());
```

## 4. 判断广告是否就绪

```java
boolean ready = SmartedAdSdk.AdRewardReady();
```

## 5. 展示激励广告

```java
private void showReward() {
    if (isFinishing() || isDestroyed()) {
        return;
    }

    if (SmartedAdSdk.AdRewardReady()) {
        boolean accepted = SmartedAdSdk.showRewardAd(this);
        Log.d(TAG, "激励展示请求是否被接受：" + accepted);
    } else {
        SmartedAdSdk.loadRewardAd(getApplicationContext());
    }
}
```

`showRewardAd()` 返回 `true` 只表示展示请求已被接受或成功投递。展示结果以回调为准。

## 6. 回调说明

| 回调 | 说明 |
| --- | --- |
| `onRewardAdLoadedSuccess()` | 激励广告加载成功。 |
| `onRewardAdLoadedFailed(errorinfo)` | 激励广告加载失败。不要进行无延迟死循环重试。 |
| `onRewardAdImpression()` | 广告已经真实展示并产生曝光。 |
| `onRewardAdShowedSuccess(platform, placementId)` | 广告展示成功。当前 SDK 在曝光回调后发送此回调。 |
| `onRewardAdShowedFailed(platform, placementId, errorinfo)` | 没有可展示广告、Activity 不可用或平台展示失败。 |
| `onRewardAdClicked(platform, placementId)` | 用户点击激励广告。 |
| `onRewardAdClosed(platform, placementId)` | 激励广告被关闭。 |

## 7. 奖励发放注意事项

当前 `SmartedAdRewardEventListener` 没有独立的“奖励已获得”回调。`onRewardAdClosed()` 只代表广告关闭，不代表用户符合奖励条件，App 不应仅依据关闭回调发放奖励。

需要接入奖励发放业务时，应在取得包含真实奖励事件的 SDK 版本后，根据对应奖励回调处理。
