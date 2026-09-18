# Unity SDK 集成与桥接

Unity Android 版本通过宿主工程中的 Java 桥接类调用 SmartedAdSdk。桥接代码属于 Unity 项目，不需要放入或修改 SmartedAdSdk 模块。

## 1. 调用关系

```text
Unity C#
    ↓ AndroidJavaClass
UnitySmartedAdBridge.java（宿主工程）
    ↓
SmartedAdSdk Android API
    ↓
MAX / AdMob / LevelPlay
```

广告平台初始化、服务器请求、路由、加载和展示仍由 Android SDK 完成。Java 桥接只负责调用公开接口，并通过 `UnityPlayer.UnitySendMessage` 把回调传回 Unity。

## 2. 添加 Android SDK 依赖

在 Unity 启用 Custom Main Gradle Template 后，在 `mainTemplate.gradle` 的依赖区域加入：

```groovy
dependencies {
    implementation "com.tripleadflow.smartad:smartedad:1.0.1"
}
```

如果使用本地 AAR，也可以把 AAR 放入 `Assets/Plugins/Android/`，并确保 Gradle 包含：

```groovy
implementation fileTree(dir: 'libs', include: ['*.jar', '*.aar'])
```

不要在 Unity 工程中重复引入冲突版本的 MAX、AdMob、LevelPlay 或相同联盟 Adapter。

## 3. 配置 AndroidManifest

使用 AdMob 时，在 Unity 自定义 Manifest 的 `<application>` 中添加：

```xml
<meta-data
    android:name="com.google.android.gms.ads.APPLICATION_ID"
    android:value="ca-app-pub-xxxxxxxxxxxxxxxx~yyyyyyyyyy" />
```

这里填写 AdMob App ID。没有配置或值为空时，SmartedAdSdk 会停用 AdMob。

## 4. 添加 Java 桥接类

把下面文件放入 Unity Android 插件源码目录，或放入导出 Android 工程的 `unityLibrary/src/main/java/com/example/smartedbridge/`：

```text
com/example/smartedbridge/UnitySmartedAdBridge.java
```

```java
package com.example.smartedbridge;

import android.app.Activity;

import com.tripleadflow.smartad.InitializeCallback;
import com.tripleadflow.smartad.SmartedAdSdk;
import com.tripleadflow.smartad.interfacecallback.SmartedAdInterstitialEventListener;
import com.tripleadflow.smartad.interfacecallback.SmartedAdRewardEventListener;
import com.unity3d.player.UnityPlayer;

import org.json.JSONObject;

public final class UnitySmartedAdBridge {
    private static String callbackObject = "SmartedAdManager";
    private static boolean listenersRegistered;

    private UnitySmartedAdBridge() {}

    private static Activity activity() {
        return UnityPlayer.currentActivity;
    }

    public static void setLogEnabled(boolean enabled) {
        SmartedAdSdk.setLogEnabled(enabled);
    }

    public static void initialize(String appId, String channelId, String unityObjectName) {
        if (unityObjectName != null && !unityObjectName.trim().isEmpty()) {
            callbackObject = unityObjectName.trim();
        }

        Activity activity = activity();
        if (activity == null) {
            emit("initFailure", "", "", "Unity Activity unavailable");
            return;
        }

        registerListeners();
        SmartedAdSdk.initActivity(activity);
        SmartedAdSdk.initialize(activity.getApplicationContext(), appId, channelId,
                new InitializeCallback() {
                    @Override
                    public void initSuccess() {
                        emit("initSuccess", "", "", "");
                    }

                    @Override
                    public void initFailure(String reason) {
                        emit("initFailure", "", "", reason);
                    }
                });
    }

    public static void loadInterstitial() {
        Activity activity = activity();
        if (activity != null) {
            SmartedAdSdk.loadInterstitialAd(activity.getApplicationContext());
        }
    }

    public static boolean isInterstitialReady() {
        return SmartedAdSdk.AdInterstitialReady();
    }

    public static boolean showInterstitial() {
        Activity activity = activity();
        if (activity == null) return false;
        SmartedAdSdk.initActivity(activity);
        return SmartedAdSdk.showInterstitialAd(activity);
    }

    public static void loadReward() {
        Activity activity = activity();
        if (activity != null) {
            SmartedAdSdk.loadRewardAd(activity.getApplicationContext());
        }
    }

    public static boolean isRewardReady() {
        return SmartedAdSdk.AdRewardReady();
    }

    public static boolean showReward() {
        Activity activity = activity();
        if (activity == null) return false;
        SmartedAdSdk.initActivity(activity);
        return SmartedAdSdk.showRewardAd(activity);
    }

    public static void destroy() {
        SmartedAdSdk.setSmartedAdInterstitialEventListener(null);
        SmartedAdSdk.setSmartedAdRewardEventListener(null);
        listenersRegistered = false;
        SmartedAdSdk.destroy();
    }

    private static synchronized void registerListeners() {
        if (listenersRegistered) return;
        listenersRegistered = true;

        SmartedAdSdk.setSmartedAdInterstitialEventListener(
                new SmartedAdInterstitialEventListener() {
                    @Override public void onInterstitialAdLoadedSuccess() {
                        emit("interstitialLoaded", "", "", "");
                    }
                    @Override public void onInterstitialAdLoadedFailed(String errorinfo) {
                        emit("interstitialLoadFailed", "", "", errorinfo);
                    }
                    @Override public void onInterstitialAdImpression() {
                        emit("interstitialImpression", "", "", "");
                    }
                    @Override public void onInterstitialAdShowedSuccess(String platform, String placementId) {
                        emit("interstitialShowed", platform, placementId, "");
                    }
                    @Override public void onInterstitialAdShowedFailed(
                            String platform, String placementId, String errorinfo) {
                        emit("interstitialShowFailed", platform, placementId, errorinfo);
                    }
                    @Override public void onInterstitialAdClicked(String platform, String placementId) {
                        emit("interstitialClicked", platform, placementId, "");
                    }
                    @Override public void onInterstitialAdClosed(String platform, String placementId) {
                        emit("interstitialClosed", platform, placementId, "");
                    }
                });

        SmartedAdSdk.setSmartedAdRewardEventListener(
                new SmartedAdRewardEventListener() {
                    @Override public void onRewardAdLoadedSuccess() {
                        emit("rewardLoaded", "", "", "");
                    }
                    @Override public void onRewardAdLoadedFailed(String errorinfo) {
                        emit("rewardLoadFailed", "", "", errorinfo);
                    }
                    @Override public void onRewardAdImpression() {
                        emit("rewardImpression", "", "", "");
                    }
                    @Override public void onRewardAdShowedSuccess(String platform, String placementId) {
                        emit("rewardShowed", platform, placementId, "");
                    }
                    @Override public void onRewardAdShowedFailed(
                            String platform, String placementId, String errorinfo) {
                        emit("rewardShowFailed", platform, placementId, errorinfo);
                    }
                    @Override public void onRewardAdClicked(String platform, String placementId) {
                        emit("rewardClicked", platform, placementId, "");
                    }
                    @Override public void onRewardAdClosed(String platform, String placementId) {
                        emit("rewardClosed", platform, placementId, "");
                    }
                });
    }

    private static void emit(String event, String platform, String placementId, String error) {
        try {
            JSONObject json = new JSONObject();
            json.put("event", event == null ? "" : event);
            json.put("platform", platform == null ? "" : platform);
            json.put("placementId", placementId == null ? "" : placementId);
            json.put("error", error == null ? "" : error);
            UnityPlayer.UnitySendMessage(callbackObject, "OnSmartedAdEvent", json.toString());
        } catch (Throwable ignored) {
            // 桥接异常不能影响广告 SDK。
        }
    }
}
```

## 5. 添加 Unity C# 包装类

```csharp
using System;
using UnityEngine;

public static class SmartedAdsAndroid
{
    private const string BridgeClass =
        "com.example.smartedbridge.UnitySmartedAdBridge";

    private static AndroidJavaClass Bridge()
    {
        return new AndroidJavaClass(BridgeClass);
    }

    public static void SetLogEnabled(bool enabled)
    {
#if UNITY_ANDROID && !UNITY_EDITOR
        using (AndroidJavaClass bridge = Bridge())
            bridge.CallStatic("setLogEnabled", enabled);
#endif
    }

    public static void Initialize(string appId, string channelId, string callbackObject)
    {
#if UNITY_ANDROID && !UNITY_EDITOR
        using (AndroidJavaClass bridge = Bridge())
            bridge.CallStatic("initialize", appId, channelId, callbackObject);
#endif
    }

    public static void LoadInterstitial()
    {
#if UNITY_ANDROID && !UNITY_EDITOR
        using (AndroidJavaClass bridge = Bridge())
            bridge.CallStatic("loadInterstitial");
#endif
    }

    public static bool IsInterstitialReady()
    {
#if UNITY_ANDROID && !UNITY_EDITOR
        using (AndroidJavaClass bridge = Bridge())
            return bridge.CallStatic<bool>("isInterstitialReady");
#else
        return false;
#endif
    }

    public static bool ShowInterstitial()
    {
#if UNITY_ANDROID && !UNITY_EDITOR
        using (AndroidJavaClass bridge = Bridge())
            return bridge.CallStatic<bool>("showInterstitial");
#else
        return false;
#endif
    }

    public static void LoadReward()
    {
#if UNITY_ANDROID && !UNITY_EDITOR
        using (AndroidJavaClass bridge = Bridge())
            bridge.CallStatic("loadReward");
#endif
    }

    public static bool IsRewardReady()
    {
#if UNITY_ANDROID && !UNITY_EDITOR
        using (AndroidJavaClass bridge = Bridge())
            return bridge.CallStatic<bool>("isRewardReady");
#else
        return false;
#endif
    }

    public static bool ShowReward()
    {
#if UNITY_ANDROID && !UNITY_EDITOR
        using (AndroidJavaClass bridge = Bridge())
            return bridge.CallStatic<bool>("showReward");
#else
        return false;
#endif
    }
}

[Serializable]
public sealed class SmartedAdEvent
{
    public string event;
    public string platform;
    public string placementId;
    public string error;
}
```

桥接类包名可以调整，但 Java 文件中的 `package`、C# 的 `BridgeClass` 和实际目录必须一致。
