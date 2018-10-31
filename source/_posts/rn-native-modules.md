---
title: ReactNative 添加本地模块
date: 2018-10-30 10:54:04
categories: FrontEnd
tags:
- ReactNative
- iOS
- Android
---

这两天由于公司移动方向的开发资源不足，所以决定接触一下一些跨平台的方案。由于`Flutter`暂时还没有 Release ，所以主要还是先依靠 `React Native` 来做一些页面。虽然 `React Native` 已经是非常成熟的技术了，但我第一次用还是遇到了问题，在此记录一下。

## 需求

因为 `React Native` 并不是完全脱离本地代码，而且也是集成进现有的项目，所以调用本地代码的需求比较大。需要在 `iOS` 和 `Android` 两方代码中都建立通道，才能在 ` React Native` 中统一调用。

## 添加模块

https://reactnative.cn/docs/native-modules-ios/

https://reactnative.cn/docs/native-modules-android/

上面是 React Native 中文网的两个文档，iOS 端直接按照文档步骤添加完成就好，Android 则与文档之中的不同。

## Android 添加自定义 Toast 模块

### 定义模块

```Java
@ReactModule(name = "RNToast")
public class ToastModule extends ReactContextBaseJavaModule {

    public ToastModule(ReactApplicationContext reactContext) {
        super(reactContext);
    }

    @Override
    public String getName() {
        return "RNToast";
    }

    @ReactMethod
    public void toast(final String msg) {
        ToastUtils.showCenterToast(msg, getReactApplicationContext());
    }

}
```

>  与文档中不同的是增加了ReactModule的注解，主要参考了已有的 ToastAndroid 模块

### 定义 Package

```Java
@ReactModuleList(nativeModules = {
        ToastModule.class
})
public class ReactNativePackage extends LazyReactPackage {

    @Override
    public List<ModuleSpec> getNativeModules(ReactApplicationContext reactContext) {
        return Arrays.asList(
                ModuleSpec.nativeModuleSpec(
                        ToastModule.class,
                        () -> new ToastModule(reactContext) // 此处是 Provider 接口实现
                )
        );
    }

    @Override
    public ReactModuleInfoProvider getReactModuleInfoProvider() {
        return LazyReactPackage.getReactModuleInfoProviderViaReflection(this);
    }

}
```

> 同样增加了 ReactModuleList 注解，以及使用了 LazyReactPackage 构建懒加载的 Package，主要参考了 MainReactPackage

### 添加 Package 到 RN 环境

```Java
mReactInstanceManager = ReactInstanceManager.builder()
        .setApplication(getApplication())
        .setBundleAssetName("index.android.bundle")
        .setJSMainModulePath("index")
        .addPackage(new MainReactPackage())
        .addPackage(new ReactNativePackage()) // 此处是新增代码
        .setUseDeveloperSupport(BuildConfig.DEBUG)
        .setInitialLifecycleState(LifecycleState.RESUMED)
        .build();
```

> 这块是初始化 ReactRootView 时的参数设置，只需要增加自定义的 Package。预先提供的内置 Module 都是这样提供的，必要时可以通过 MainPeactPackage 进入之后查看源码

### 与文档中不同的地方

文档写了需要放在指定的包名的 MainApplication 中，这个并不需要。只需要在 `ReactInstanceManager` 构建时增加自定义的 `Package` 就好。

> 这个 package 需要在`MainApplication.java`文件的`getPackages`方法中提供。这个文件位于你的 react-native 应用文件夹的 android 目录中。具体路径是: `android/app/src/main/java/com/your-app-name/MainApplication.java`

