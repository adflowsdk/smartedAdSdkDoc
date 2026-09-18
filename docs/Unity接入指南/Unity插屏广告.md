# Unity 插屏广告

## 加载

```csharp
SmartedAdsAndroid.LoadInterstitial();
```

等待 `OnSmartedAdEvent` 收到 `interstitialLoaded` 后再展示。加载失败会收到 `interstitialLoadFailed`，失败原因位于 `error`。

## 判断就绪并展示

```csharp
public void ShowInterstitial()
{
    if (SmartedAdsAndroid.IsInterstitialReady())
    {
        bool accepted = SmartedAdsAndroid.ShowInterstitial();
        Debug.Log("interstitial show accepted=" + accepted);
    }
    else
    {
        SmartedAdsAndroid.LoadInterstitial();
    }
}
```

`accepted=true` 只表示展示请求被 Android SDK 接受。实际结果以 `interstitialShowed` 或 `interstitialShowFailed` 为准。

## 处理关闭

```csharp
case "interstitialClosed":
    Debug.Log("interstitial ad closed");
    // 在这里恢复游戏界面或更新业务状态。
    break;
```

SDK 会根据服务端的 `reloadAfterClose` 策略决定是否自动加载下一条广告。只有项目明确关闭自动重载时，才需要在这里手动调用 `LoadInterstitial()`。

插屏广告事件按真实平台事件回传，不受 Adjust 归因开关影响。
