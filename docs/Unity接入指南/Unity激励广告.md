# Unity 激励广告

## 加载

```csharp
SmartedAdsAndroid.LoadReward();
```

等待 `rewardLoaded` 事件后再展示。加载失败会收到 `rewardLoadFailed`。

## 判断就绪并展示

```csharp
public void ShowReward()
{
    if (SmartedAdsAndroid.IsRewardReady())
    {
        bool accepted = SmartedAdsAndroid.ShowReward();
        Debug.Log("reward show accepted=" + accepted);
    }
    else
    {
        SmartedAdsAndroid.LoadReward();
    }
}
```

## 处理关闭

```csharp
case "rewardClosed":
    Debug.Log("reward ad closed");
    // 在这里恢复游戏界面或更新业务状态。
    break;
```

`rewardClosed` 只表示广告已经关闭。SDK 会根据服务端的 `reloadAfterClose` 策略决定是否自动加载下一条广告，不建议在这里固定调用 `LoadReward()`，否则可能产生重复加载。如果项目明确关闭了自动重载，并希望手动预加载，才在关闭回调中调用 `LoadReward()`。

当前 Android `SmartedAdRewardEventListener` 没有独立的奖励获得回调。不能根据 `rewardClosed` 发放奖励；需要奖励业务时，应使用后续提供真实奖励回调的 SDK 版本。
