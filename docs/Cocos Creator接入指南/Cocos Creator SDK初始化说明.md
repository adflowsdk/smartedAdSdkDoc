# Cocos Creator SDK 初始化说明

## 1. 注册原生事件接收器

在调用初始化前，为 Java 桥接层创建全局事件入口：

```javascript
import { SmartedAds } from "./SmartedAds";

globalThis.SmartedAdBridge = {
    onNativeEvent(json) {
        let adEvent;
        try {
            adEvent = JSON.parse(json);
        } catch (error) {
            console.error("Invalid SmartedAd event", json, error);
            return;
        }

        console.log(
            "SmartedAd event=", adEvent.event,
            "platform=", adEvent.platform,
            "placementId=", adEvent.placementId,
            "error=", adEvent.error
        );

        switch (adEvent.event) {
            case "initSuccess":
                SmartedAds.loadInterstitial();
                SmartedAds.loadReward();
                break;
            case "initFailure":
                console.error("SmartedAd init failed:", adEvent.error);
                break;
        }
    }
};

SmartedAds.setLogEnabled(true);
SmartedAds.initialize("YOUR_APP_ID", "YOUR_CHANNEL_ID");
```

在 Creator 2.x 项目中，如果运行环境没有 `globalThis`，可以改为：

```javascript
window.SmartedAdBridge = {
    onNativeEvent(json) {
        // 与上面的处理逻辑相同。
    }
};
```

## 2. 初始化时机

- 必须先在 `AppActivity.onCreate()` 中调用 `CocosSmartedAdBridge.bind(this)`。
- JavaScript 引擎启动并注册 `SmartedAdBridge.onNativeEvent` 后，再调用 `SmartedAds.initialize()`。
- `initSuccess` 代表 Android SDK 启动完成，不代表广告已经加载完成。
- App 返回前台后，展示接口会再次把当前 Activity 交给 SmartedAdSdk。

## 3. 事件名称

| event | 含义 |
| --- | --- |
| `initSuccess` | SDK 初始化成功 |
| `initFailure` | SDK 初始化失败 |
| `interstitialLoaded` | 插屏加载成功 |
| `interstitialLoadFailed` | 插屏加载失败 |
| `interstitialImpression` | 插屏曝光 |
| `interstitialShowed` | 插屏展示成功 |
| `interstitialShowFailed` | 插屏展示失败 |
| `interstitialClicked` | 插屏被点击 |
| `interstitialClosed` | 插屏关闭 |
| `rewardLoaded` | 激励广告加载成功 |
| `rewardLoadFailed` | 激励广告加载失败 |
| `rewardImpression` | 激励广告曝光 |
| `rewardShowed` | 激励广告展示成功 |
| `rewardShowFailed` | 激励广告展示失败 |
| `rewardClicked` | 激励广告被点击 |
| `rewardClosed` | 激励广告关闭 |

这些事件是 Android SDK 的真实 App 回调，不受 Adjust 归因策略开关影响。
