### Android Native SDK集成流程

- 下载 [百度语音识别Android SDK](http://ai.baidu.com/sdk#asr)

- 使用Android Studio打开Android项目,OpenFile - Android项目

- 创建语音Library(File-new Module-Library(asr_plugin)

- 添加sdk(解压下载的SDK/core/libs/***.jar)拷贝到asr_plugin/libs下

- 配置so库文件(解压下载的SDK/core/src/main/jniLibs)将jniLibs整个文件夹拷贝到项目同等位置,删除下面的armeabi,armeabi-v7a(因为Flutter没有armeabi,armeabi-v7a架构的so),删除使用不到的库文件,保留图中文件

  ![](http://ww1.sinaimg.cn/large/006tNc79ly1g3y517jyyjj308v04xmxa.jpg)

  

- 在AndroidManifest中声明权限

  ```
  <manifest xmlns:android="http://schemas.android.com/apk/res/android"
      package="com.yangdxg.asr_plugin">
  
      <uses-permission android:name="android.permission.RECORD_AUDIO" />
      <uses-permission android:name="android.permission.ACCESS_NETWORK_STATE" />
      <uses-permission android:name="android.permission.INTERNET" />
      <uses-permission android:name="android.permission.READ_PHONE_STATE" />
      <uses-permission android:name="android.permission.WRITE_EXTERNAL_STORAGE" />
  </manifest>
  ```

- 在控制台创建应用获取AppId并配置到AndroidManifest

  ```
   <application>
      <meta-data android:name="com.baidu.speech.APP_ID"
          android:value="9788136" />
      <meta-data
          android:name="com.baidu.speech.API_KEY"
          android:value="0GjQNO5H4pGPf9HyA3AmZEbz" />
      <meta-data
          android:name="com.baidu.speech.SECRET_KEY"
          android:value="db981ef3ec647ba8a09b599ad7447a24" />
    </application>
  ```

- 设置service,同样在AndroidManifest中配置

  ```
  <service android:name="com.baidu.speech.VoiceRecognitionService" android:exported="false" />
  ```

- 在app build.gradle中dependencies中添加依赖

  ```
  implementation project(':asr_plugin')
  ```

  注:报错Error:Program type already present: android.support.v4.os.ResultReceiver

  解决:在gradle.properties中添加

  ```
  android.useAndroidX=true
  android.enableJetifier=true
  ```

## iOS Native SDK集成流程

- [下载iOS SDK](https://ai.baidu.com/sdk)

- 打包iOS工程,Mac下双击工程目录下Runner.xcworkspace文件即可

- Runner上右键new Group命名为plugin

- 将下载的SDK中BDSClientLib文件夹拖到plugin下,勾选Copy items if needed和Create groups,删除BDSClientLib下的.gitignore

- 拖动BDSClientResource/ASR/BDSClientResources到plugin下,勾选Copy items if needed和Create folder references

- 拖动BDSClientResource/ASR/BDSClientEASRResources到plugin下,勾选Copy items if needed和Create folder references

- 添加framework,点击如图'+'号,添加以下需要的framework

  ![](http://ww3.sinaimg.cn/large/006tNc79ly1g3wwr770dxj30ik08vt9s.jpg)

  ​		CoreTelephony.framework

  ​		libsqlite3.0.tbd

  ​		libiconv.2.4.0.tbd

  ​		libc++.tbd

  ​		libs.1.2.5.tbd

- 添加语音权限Privacy - Microphone Usage Description

  ![](http://ww3.sinaimg.cn/large/006tNc79ly1g3wx1y4zkaj30e504rt9a.jpg)

## Flutter Plugin开发指南 - Dart端实现

在lib下创建plugin/asr_manager.dart编写Dart端代码,用于传递信息到Native端实现相应功能

```
import 'package:flutter/services.dart';

class AsrManager {
  static const MethodChannel _channel = const MethodChannel('asr_plugin');

  /**
   * 开始录音
   */
  static Future<String> start({Map params}) async {
    return await _channel.invokeMethod('start', params ?? {});
  }

  /**
   * 停止录音
   */
  static Future<String> stop() async {
    return await _channel.invokeMethod('stop');
  }

  /**
   * 取消录音
   */
  static Future<String> cancel() async {
    return await _channel.invokeMethod('cancel');
  }
}
```

## Flutter Plugin开发指南 - Android端实现

- 为Android Model添加Flutter依赖,在model中build.gradle添加如下代码

  ```
  apply plugin: 'com.android.library'
  
  def localProperties = new Properties()
  def localPropertiesFile = rootProject.file('local.properties')
  if (localPropertiesFile.exists()) {
      localPropertiesFile.withReader('UTF-8') { reader ->
          localProperties.load(reader)
      }
  }
  
  def flutterRoot = localProperties.getProperty('flutter.sdk')
  apply from: "$flutterRoot/packages/flutter_tools/gradle/flutter.gradle"
  
  flutter {
      source '../..'
  }
  ```

- 解决model和app都依赖Flutter冲突问题

  1. app/build.gradle中defaultConfig下添加ndk选择代码

     ```
             ndk{
                 abiFilters "arm64-v7a","arm64-v8a","x86_64","x86"
             }
     ```

  2. app/build.gradle中android下添加下面代码

     ```
         packagingOptions {
             // 确保app与asr_plugin都依赖的libflutter.so merge时不冲突@https://github.com/card-io/card.io-Android-SDK/issues/186#issuecomment-427552552
             pickFirst 'lib/x86_64/libflutter.so'
             pickFirst 'lib/x86/libflutter.so'
             pickFirst 'lib/arm64-v8a/libflutter.so'
         }
     ```

- 实现Android端语音识别功能(略)

- Android端创建AsrPlugin实现接收Dart的消息

  ```
      @Override
      public void onMethodCall(MethodCall methodCall, MethodChannel.Result result) {
          initPermission();
          switch (methodCall.method) {
              case "start":
                  resultStateful = ResultStateful.of(result);
                  start(methodCall, resultStateful);
                  break;
              case "stop":
                  stop(methodCall,result);
                  break;
              case "cancel":
                  cancel(methodCall,result);
                  break;
              default:
                  result.notImplemented();
          }
      }
  ```

- 注册插件到MainActivity

  ```
  AsrPlugin.registerWith(registrarFor("com.yangdxg.asr_plugin.asr.AsrPlugin"));
  ```

  

## Flutter Plugin开发指南 - iOS端实现

- 在plugin下创建文件夹ASRPlugin,拖到工程plugin下,将导入的SDK等资源文件拖到ASRPlugin下,删除SDK等三个资源文件(选择Remove References)只删除路径再从新导入

  ![](http://ww1.sinaimg.cn/large/006tNc79ly1g3y1r0o4a7j307w07n0sz.jpg)

- 在plugin下右键创建Object-C文件取名AsrManager,FileType选择Category,class选择NSObject,创建成功后更改.h和.m文件名为AsrManager

- iOS端实现语音识别略(参考百度API文档)

- iOS接收Dart消息

  ```
  + (void)registerWithRegistrar:(NSObject <FlutterPluginRegistrar> *)registrar {
      FlutterMethodChannel *channel = [FlutterMethodChannel methodChannelWithName:@"asr_plugin" binaryMessenger:[registrar messenger] ];
      AsrPlugin* instance =[AsrPlugin new];
      [registrar addMethodCallDelegate:instance channel:channel];
  }
  - (void)handleMethodCall:(FlutterMethodCall *)call result:(FlutterResult)result {
      if ([@"start" isEqualToString:call.method]) {
          self.result = result;
          [[self _asrManager] start];
      }else if ([@"stop" isEqualToString:call.method]) {
           [[self _asrManager] stop];
      }else if ([@"cancel" isEqualToString:call.method]) {
          [[self _asrManager] cancel];
      } else{
          result(FlutterMethodNotImplemented);
      }
  }
  - (AsrManager*)_asrManager{
      if (!self.asrManager) {
          self.asrManager = [AsrManager initWith:^(NSString *message) {
              if (self.result) {
                  self.result(message);
                  self.result = nil;
              }
          } failure:^(NSString *message) {
              if (self.result) {
                  self.result([FlutterError errorWithCode:@"ASR fail" message:message details:nil]);
                  self.result = nil;
              }
          }];
      }
      return self.asrManager;
  }
  ```

- 注册Dart消息接收

  ```
  - (BOOL)application:(UIApplication *)application
      didFinishLaunchingWithOptions:(NSDictionary *)launchOptions {
    [GeneratedPluginRegistrant registerWithRegistry:self];
      //注册自有的插件
      [AsrPlugin registerWithRegistrar:[self registrarForPlugin:@"AsrPlugain"]];
    // Override point for customization after application launch.
    return [super application:application didFinishLaunchingWithOptions:launchOptions];
  }
  ```

## Flutter AI智能语音功能实现

- 界面开发略

- 实现点击方法调用AsrManager对应的调用Native的方法

  ```
    _speakStart() {
      controller.forward();
      setState(() {
        speakTips = '-- 识别中 --';
      });
  
      AsrManager.start().then((text) {
        print(text);
        if (text != null && text.length > 0) {
          setState(() {
            speakResult = text;
          });
          //选关闭再跳转
          Navigator.pop(context);
          Navigator.push(context, MaterialPageRoute(
              builder: (context) => SearchPage(keyword: speakResult,)));
        }
      }).catchError((e) {
        print('识别出错---' + e);
      });
    }
  
    _speakStop() {
      setState(() {
        speakTips = '长按说话';
      });
      controller.reset();
      controller.stop();
      AsrManager.stop();
    }
  
    _speakCancle() {
      AsrManager.cancel();
    }
  
  ```

  

[Demo下载](https://github.com/yangdxg/flutter_trip)

