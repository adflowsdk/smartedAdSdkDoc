# Cocos Creator 插屏广告

## 加载

```javascript
SmartedAds.loadInterstitial();
```

等待 `interstitialLoaded` 事件后再展示。加载失败时会收到 `interstitialLoadFailed`，失败原因位于 `error`。

## 判断就绪并展示

```javascript
function showInterstitial() {
    if (SmartedAds.isInterstitialReady()) {
        const accepted = SmartedAds.showInterstitial();
        console.log("interstitial show accepted=", accepted);
    } else {
        SmartedAds.loadInterstitial();
    }
}
```

`accepted=true` 只表示 Android SDK 接受了展示请求。实际结果以 `interstitialShowed` 或 `interstitialShowFailed` 为准。

## 处理关闭

```javascript
case "interstitialClosed":
    console.log("interstitial ad closed");
    // 在这里恢复游戏界面或更新业务状态。
    break;
```

SDK 会根据服务端的 `reloadAfterClose` 策略决定是否自动加载下一条广告。只有项目明确关闭自动重载时，才需要在这里手动调用 `loadInterstitial()`。

展示广告时，Cocos App 必须位于前台，并且绑定的 `AppActivity` 仍然有效。
