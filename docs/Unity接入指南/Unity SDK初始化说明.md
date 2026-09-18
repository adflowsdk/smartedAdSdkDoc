# Unity SDK 初始化说明

## 1. 创建回调接收对象

场景中创建一个名为 `SmartedAdManager` 的 GameObject，并挂载下面的脚本。GameObject 名称必须与初始化时传入的名称一致。

```csharp
using UnityEngine;

public sealed class SmartedAdManager : MonoBehaviour
{
    private void Awake()
    {
        DontDestroyOnLoad(gameObject);

        SmartedAdsAndroid.SetLogEnabled(Debug.isDebugBuild);
        SmartedAdsAndroid.Initialize(
            "YOUR_APP_ID",
            "YOUR_CHANNEL_ID",
            gameObject.name);
    }

    // 方法名由 Java 桥接层固定调用。
    public void OnSmartedAdEvent(string json)
    {
        SmartedAdEvent adEvent = JsonUtility.FromJson<SmartedAdEvent>(json);
        Debug.Log("SmartedAd event=" + adEvent.event
            + ", platform=" + adEvent.platform
            + ", placementId=" + adEvent.placementId
            + ", error=" + adEvent.error);

        switch (adEvent.event)
        {
            case "initSuccess":
                SmartedAdsAndroid.LoadInterstitial();
                SmartedAdsAndroid.LoadReward();
                break;
            case "initFailure":
                Debug.LogError("SmartedAd init failed: " + adEvent.error);
                break;
        }
    }
}
```

## 2. 初始化说明

- `YOUR_APP_ID` 为 SmartedAdSdk 分配的应用 ID。
- `YOUR_CHANNEL_ID` 为 SmartedAdSdk 分配的渠道 ID。
- `initSuccess` 表示 SDK 启动完成，不代表广告已经加载完成。
- `UnitySendMessage` 按 GameObject 名称查找对象，因此接收对象切换场景时应使用 `DontDestroyOnLoad`，或确保新场景创建同名对象。
- 广告事件由 Android SDK 在主线程产生，再转发到 Unity。

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
