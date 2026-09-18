# Android SDK 初始化说明

## 1. 在 Application 中初始化

建议先开启测试日志，再调用初始化接口：

```java
package com.example.app;

import android.app.Application;
import android.util.Log;

import com.tripleadflow.smartad.InitializeCallback;
import com.tripleadflow.smartad.SmartedAdSdk;

public final class MyApplication extends Application {
    private static final String TAG = "SmartedAdDemo";

    @Override
    public void onCreate() {
        super.onCreate();

        // 测试阶段建议开启，正式包可关闭。
        SmartedAdSdk.setLogEnabled(BuildConfig.DEBUG);

        SmartedAdSdk.initialize(
                this,
                "YOUR_APP_ID",
                "YOUR_CHANNEL_ID",
                new InitializeCallback() {
                    @Override
                    public void initSuccess() {
                        Log.d(TAG, "SmartedAdSdk 初始化成功");

                        // 可按业务需要预加载广告。
                        SmartedAdSdk.loadInterstitialAd(MyApplication.this);
                        SmartedAdSdk.loadRewardAd(MyApplication.this);
                    }

                    @Override
                    public void initFailure(String reason) {
                        Log.e(TAG, "SmartedAdSdk 初始化失败：" + reason);
                    }
                }
        );
    }
}
```

## 2. 初始化参数

| 参数 | 说明 |
| --- | --- |
| `context` | 建议传入 `Application`。SDK 内部使用 Application Context。 |
| `appId` | SmartedAdSdk 分配的应用 ID，不是 AdMob App ID。 |
| `channelId` | SmartedAdSdk 分配的渠道 ID。 |
| `callback` | 初始化结果回调，运行在 Android 主线程。 |

`appId`、`channelId` 或 `context` 无效时会回调 `initFailure()`。同一进程内应使用同一组 `appId` 和 `channelId`。

初始化成功代表 SDK 已完成启动流程，可以调用广告加载接口。它不代表某一条广告已经加载完成，广告是否可展示仍以加载回调和 `AdInterstitialReady()` / `AdRewardReady()` 为准。

## 3. 注册广告监听器

插屏和激励广告使用独立监听器，互不覆盖。推荐在 `Application` 中注册一次：

```java
SmartedAdSdk.setSmartedAdInterstitialEventListener(interstitialListener);
SmartedAdSdk.setSmartedAdRewardEventListener(rewardListener);
```

传入 `null` 可以单独取消注册：

```java
SmartedAdSdk.setSmartedAdInterstitialEventListener(null);
SmartedAdSdk.setSmartedAdRewardEventListener(null);
```

监听器不要长期持有 Activity、Fragment 或 View。需要更新页面时，可将回调转发给当前页面。

## 4. 登记当前 Activity

展示接口建议直接传入当前前台 Activity：

```java
SmartedAdSdk.showInterstitialAd(this);
SmartedAdSdk.showRewardAd(this);
```

也可以在页面恢复时登记 Activity，作为 SDK 查找展示页面的备用方式：

```java
@Override
protected void onResume() {
    super.onResume();
    SmartedAdSdk.initActivity(this);
}
```

## 5. 日志

```java
SmartedAdSdk.setLogEnabled(true);
```

测试完成后建议关闭：

```java
SmartedAdSdk.setLogEnabled(false);
```

## 6. 销毁 SDK

```java
SmartedAdSdk.destroy();
```

`destroy()` 会取消任务、销毁广告对象并清空监听器和初始化状态。不要在普通 Activity 的 `onDestroy()` 中调用。只有整个广告业务不再使用，或需要完整重置 SDK 时才调用。销毁后再次使用需要重新注册监听器并初始化。
