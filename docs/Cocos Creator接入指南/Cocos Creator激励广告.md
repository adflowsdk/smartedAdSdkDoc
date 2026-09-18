# Cocos Creator 激励广告

## 加载

```javascript
SmartedAds.loadReward();
```

等待 `rewardLoaded` 事件后再展示。加载失败时会收到 `rewardLoadFailed`。

## 判断就绪并展示

```javascript
function showReward() {
    if (SmartedAds.isRewardReady()) {
        const accepted = SmartedAds.showReward();
        console.log("reward show accepted=", accepted);
    } else {
        SmartedAds.loadReward();
    }
}
```

## 处理关闭

```javascript
case "rewardClosed":
    console.log("reward ad closed");
    // 在这里恢复游戏界面或更新业务状态。
    break;
```

`rewardClosed` 只表示广告已经关闭。SDK 会根据服务端的 `reloadAfterClose` 策略决定是否自动加载下一条广告，不建议在这里固定调用 `loadReward()`，否则可能产生重复加载。如果项目明确关闭了自动重载，并希望手动预加载，才在关闭回调中调用 `loadReward()`。

当前 Android `SmartedAdRewardEventListener` 没有独立的奖励获得回调。不能根据 `rewardClosed` 发放奖励；需要奖励业务时，应使用后续提供真实奖励回调的 SDK 版本。
