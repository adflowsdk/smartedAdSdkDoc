# Android SDK 集成说明

## 1. 添加 SDK 依赖

在 App 模块的 `build.gradle` 中添加：

```groovy
dependencies {
    implementation "com.tripleadflow.smartad:smartedad:1.0.1"
}
```

`1.0.1` 为当前暂定版本号，发布后请使用实际提供的版本。

SmartedAdSdk 发布包已经包含所需广告平台及联盟 SDK。宿主 App 不要重复引入不同版本的 MAX、AdMob、LevelPlay 或相同联盟 Adapter，避免重复类和版本冲突。

## 2. AndroidManifest 配置

SDK AAR 会合并以下权限：

```xml
<uses-permission android:name="android.permission.INTERNET" />
<uses-permission android:name="android.permission.ACCESS_NETWORK_STATE" />
<uses-permission android:name="com.google.android.gms.permission.AD_ID" />
```

使用 AdMob 时，必须在宿主 App 的 `<application>` 节点内配置 AdMob App ID：

```xml
<application
    android:name=".MyApplication"
    ...>

    <meta-data
        android:name="com.google.android.gms.ads.APPLICATION_ID"
        android:value="ca-app-pub-xxxxxxxxxxxxxxxx~yyyyyyyyyy" />

</application>
```

这里填写带 `~` 的 AdMob App ID，不能填写插屏或激励广告位 ID。没有配置或值为空时，SDK 会停用 AdMob，并从其他已启用平台中选择可用平台。

## 3. 添加 Application

如果工程已有自定义 `Application`，直接在它的 `onCreate()` 中初始化 SDK。如果没有，请新建：

```java
package com.example.app;

import android.app.Application;

public final class MyApplication extends Application {
    @Override
    public void onCreate() {
        super.onCreate();
        // SmartedAdSdk 初始化代码见“SDK 初始化说明”。
    }
}
```

并在 Manifest 中登记：

```xml
<application
    android:name=".MyApplication"
    ... />
```

## 4. 混淆说明

SmartedAdSdk 对外入口和监听器接口已保留稳定名称。正常情况下，宿主 App 无需为公开 API 添加额外 keep 规则。宿主开启 R8 后，仍应保留其业务层中通过反射或第三方框架使用的类。

## 5. 接入检查

完成集成后，请确认：

- App 的 `minSdk` 不低于 24。
- 已使用分配的 SmartedAdSdk `appId` 和 `channelId`。
- 使用 AdMob 时，Manifest 中配置了有效的 AdMob App ID。
- App 没有重复引入冲突版本的广告 SDK。
- 测试阶段已调用 `SmartedAdSdk.setLogEnabled(true)`。
