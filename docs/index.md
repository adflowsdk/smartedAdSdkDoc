# SmartedAdSdk Android 接入文档

SmartedAdSdk 为 Android App 提供统一的广告加载与展示接口，目前支持：

- 插屏广告
- 激励广告
- AppLovin MAX、Google AdMob、Unity LevelPlay 三个聚合平台

SDK 会根据运行时配置为插屏和激励广告分别选择聚合平台。App 只需调用 SmartedAdSdk 的统一接口，无需直接调用各聚合平台 API。

Unity 和 Cocos Creator 的 Android 版本可以通过宿主工程中的轻量桥接代码调用同一套 Android SDK。桥接层只负责传递参数和事件，AdMob、MAX、LevelPlay、SmartedAdActivity、服务器通信以及广告加载与展示逻辑仍由 SmartedAdSdk 处理。

## 推荐接入顺序

1. 阅读 [SDK 集成说明](Android接入指南/Android%20SDK集成说明.md)，完成依赖和 Manifest 配置。
2. 阅读 [SDK 初始化说明](Android接入指南/Android%20SDK初始化说明.md)，在 `Application` 中初始化。
3. 按广告类型接入 [插屏广告](Android接入指南/Android插屏广告.md) 或 [激励广告](Android接入指南/Android激励广告.md)。

Unity 项目从 [Unity SDK 集成与桥接](Unity接入指南/Unity%20SDK集成与桥接.md) 开始；Cocos Creator 项目从 [Cocos Creator SDK 集成与桥接](Cocos%20Creator接入指南/Cocos%20Creator%20SDK集成与桥接.md) 开始。

## 环境要求

| 项目 | 要求 |
| --- | --- |
| Android 最低版本 | API 24（Android 7.0） |
| SDK 编译版本 | compileSdk 36 |
| 开发语言 | Java 或可调用 Java API 的 Kotlin |
| SDK 依赖版本 | `1.0.1`（暂定） |

## 线程约定

SmartedAdSdk 的公开接口可从任意线程调用。广告平台初始化、加载、展示和销毁由 SDK 调度到 Android 主线程；网络、JSON、设备信息和文件读写在工作线程执行。初始化回调及广告事件回调统一返回主线程。
