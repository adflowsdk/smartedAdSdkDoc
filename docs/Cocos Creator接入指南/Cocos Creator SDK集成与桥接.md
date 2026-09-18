# Cocos Creator SDK 集成与桥接

Cocos Creator 发布 Android 原生版本时，可以通过 JSB 反射调用宿主 Android 工程中的 Java 桥接类。桥接类由接入方放在自己的 Cocos Android 工程中，不需要修改 SmartedAdSdk。

## 1. 调用关系

```text
Cocos Creator JavaScript / TypeScript
    ↓ JSB reflection
CocosSmartedAdBridge.java（宿主工程）
    ↓
SmartedAdSdk Android API
    ↓
MAX / AdMob / LevelPlay
```

广告平台、服务器通信、SmartedAdActivity、路由、加载与展示逻辑继续由 Android SDK 处理。

## 2. 添加 Android SDK 依赖

在 Cocos Creator 构建出的 Android App 模块 `build.gradle` 中添加：

```groovy
dependencies {
    implementation "com.tripleadflow.smartad:smartedad:1.0.1"
}
```

使用本地 AAR 时，将文件放入 App 模块的 `libs/`，并添加：

```groovy
dependencies {
    implementation fileTree(dir: 'libs', include: ['*.jar', '*.aar'])
}
```

不要重复引入冲突版本的 MAX、AdMob、LevelPlay 或相同联盟 Adapter。

## 3. 配置 AndroidManifest

使用 AdMob 时，在 `<application>` 中配置：

```xml
<meta-data
    android:name="com.google.android.gms.ads.APPLICATION_ID"
    android:value="ca-app-pub-xxxxxxxxxxxxxxxx~yyyyyyyyyy" />
```

## 4. 添加 Java 桥接类

把下面文件放入 Cocos Android App 模块：

```text
app/src/main/java/com/example/smartedbridge/CocosSmartedAdBridge.java
```

```java
package com.example.smartedbridge;

import android.app.Activity;

import com.tripleadflow.smartad.InitializeCallback;
import com.tripleadflow.smartad.SmartedAdSdk;
import com.tripleadflow.smartad.interfacecallback.SmartedAdInterstitialEventListener;
import com.tripleadflow.smartad.interfacecallback.SmartedAdRewardEventListener;

import org.cocos2dx.lib.Cocos2dxActivity;
import org.cocos2dx.lib.Cocos2dxJavascriptJavaBridge;
import org.json.JSONObject;

import java.lang.ref.WeakReference;

public final class CocosSmartedAdBridge {
    private static WeakReference<Activity> activityRef = new WeakReference<>(null);
    private static boolean listenersRegistered;

    private CocosSmartedAdBridge() {}

    /** 在宿主 AppActivity.onCreate() 中调用一次。 */
    public static void bind(Activity activity) {
        activityRef = new WeakReference<>(activity);
        if (activity != null) {
            SmartedAdSdk.initActivity(activity);
        }
        registerListeners();
    }

    private static Activity activity() {
        return activityRef.get();
    }

    public static void setLogEnabled(boolean enabled) {
        SmartedAdSdk.setLogEnabled(enabled);
    }

    public static void initialize(String appId, String channelId) {
        Activity activity = activity();
        if (activity == null) {
            emit("initFailure", "", "", "Cocos Activity unavailable");
            return;
        }

        registerListeners();
        SmartedAdSdk.initialize(activity.getApplicationContext(), appId, channelId,
                new InitializeCallback() {
                    @Override public void initSuccess() {
                        emit("initSuccess", "", "", "");
                    }

                    @Override public void initFailure(String reason) {
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
        activityRef.clear();
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
        Activity activity = activity();
        if (!(activity instanceof Cocos2dxActivity)) return;

        try {
            JSONObject json = new JSONObject();
            json.put("event", event == null ? "" : event);
            json.put("platform", platform == null ? "" : platform);
            json.put("placementId", placementId == null ? "" : placementId);
            json.put("error", error == null ? "" : error);

            // 同时兼容 Creator 2.x 的 window 与 Creator 3.x 的 globalThis。
            String script = "(function(){"
                    + "var root=(typeof globalThis!=='undefined')?globalThis:"
                    + "((typeof window!=='undefined')?window:this);"
                    + "if(root.SmartedAdBridge&&root.SmartedAdBridge.onNativeEvent){"
                    + "root.SmartedAdBridge.onNativeEvent("
                    + JSONObject.quote(json.toString()) + ");}})();";

            ((Cocos2dxActivity) activity).runOnGLThread(
                    () -> Cocos2dxJavascriptJavaBridge.evalString(script));
        } catch (Throwable ignored) {
            // 桥接异常不能影响广告 SDK。
        }
    }
}
```

## 5. 在 AppActivity 中绑定

找到 Cocos 项目的 `AppActivity`，在 `onCreate()` 中绑定当前 Activity：

```java
import android.os.Bundle;
import com.example.smartedbridge.CocosSmartedAdBridge;

@Override
protected void onCreate(Bundle savedInstanceState) {
    super.onCreate(savedInstanceState);
    CocosSmartedAdBridge.bind(this);
}
```

## 6. 添加 JavaScript/TypeScript 包装代码

下面的写法同时兼容常见的 Creator 2.x `jsb.reflection` 和 Creator 3.x `native.reflection`：

```javascript
const SMARTED_BRIDGE = "com/example/smartedbridge/CocosSmartedAdBridge";

function androidReflection() {
    if (typeof native !== "undefined" && native.reflection) {
        return native.reflection;
    }
    if (typeof jsb !== "undefined" && jsb.reflection) {
        return jsb.reflection;
    }
    return null;
}

function callAndroid(method, signature, ...args) {
    const reflection = androidReflection();
    if (!reflection) {
        console.warn("SmartedAd Android bridge is unavailable");
        return null;
    }
    return reflection.callStaticMethod(
        SMARTED_BRIDGE,
        method,
        signature,
        ...args
    );
}

export const SmartedAds = {
    setLogEnabled(enabled) {
        callAndroid("setLogEnabled", "(Z)V", enabled);
    },

    initialize(appId, channelId) {
        callAndroid(
            "initialize",
            "(Ljava/lang/String;Ljava/lang/String;)V",
            appId,
            channelId
        );
    },

    loadInterstitial() {
        callAndroid("loadInterstitial", "()V");
    },

    isInterstitialReady() {
        return callAndroid("isInterstitialReady", "()Z") === true;
    },

    showInterstitial() {
        return callAndroid("showInterstitial", "()Z") === true;
    },

    loadReward() {
        callAndroid("loadReward", "()V");
    },

    isRewardReady() {
        return callAndroid("isRewardReady", "()Z") === true;
    },

    showReward() {
        return callAndroid("showReward", "()Z") === true;
    },

    destroy() {
        callAndroid("destroy", "()V");
    }
};
```

如果修改 Java 包名，必须同时修改 Java 的 `package`、文件目录和脚本中的 `SMARTED_BRIDGE`。
