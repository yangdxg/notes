### 混合开发Flutter集成步骤

- 创建Flutter module
- 添加Flutter module依赖
- 在Java/Object-c中调用Flutter module
- 编写Dart代码
- 运行项目
- 热重启/重新加载
- 调试Dart代码
- 发布应用

#### 创建Flutter module

项目目录xxx/flutter/Navive项目:

```
cd xxx/flutter/
flutter create -t module flutter_module
```

上面的代码会切换到Android/iOS项目的上一级目录,并创建flutter模块

flutter中文件用途

- .android - flutter_module的Android宿主工程;
- .ios - flutter_module的iOS宿主工程
- lib - flutter_module的Dart部分代码
- pubspec.yaml - flutter_module的项目依赖配置文件 

![](http://ww3.sinaimg.cn/large/006tNc79ly1g3aabkufp9j307105cdfv.jpg)

Flutter_module是上面创建的flutter module,Android Hybrid是Android工程

### Flutter Android混合开发

#### 添加flutter依赖

在Android工程中的settings.gradle文件中进行如下配置

```
include ':app'
setBinding(new Binding([gradle:this]))
evaluate(new File(
        settingsDir.parentFile,
        'flutter_module/.android/include_flutter.groovy'
))
```

同步后会在工程目录中显示我们的flutter module

![](http://ww3.sinaimg.cn/large/006tNc79ly1g3aadqa4hoj306b0400ss.jpg)

在app的build.gradle中添加对flutter的依赖

```
dependencies {
    implementation fileTree(dir: 'libs', include: ['*.jar'])
		...
    implementation project(':flutter')
}
```

使用flutter要求minSDK最小应该>=16

另外使用flutter也需要设置支持Java8特性

```
android {
  ...
  compileOptions {
    sourceCompatibility JavaVersion.VERSION_1_8
    targetCompatibility JavaVersion.VERSION_1_8
  }
}
```



#### 在Java中调用Flutter module

Java中调用Flutter模块有俩种方式

- 使用Flutter.createView API方式
- 使用FlutterFragment的方式

##### 使用Flutter.createView API的方式

```
View flutterView = Flutter.createView(
	MainActivity.this,
	getLifecycle(),
	"route1"
);
```

##### 使用FlutterFragment的方式

```
FragmentTransaction tx = getSupportFragmentManager().beginTransaction();
tx.replace(R.id.someContainer,Flutter.createFragment('route1'));
tx.commit();
```

字符串"route1"用来告诉Dart代码在Flutter视图中显示哪个小部件,Flutter模块项目的lib/main.dart文件需要通过window.defaultRouteName来获取Native指定要显示的路由名,以确定要创建哪个窗口小部件并传递给runAPP:

##### 调用Flutter module时传递数据

无论通过Flutter.createView的方式,还是通过Flutter.createFragment的方式,都允许我们加载Flutter module时传递一个String类型的initialRoute参数,也可以穿Json字符串.

Native代码

```
        mBtnCreate.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View view) {
                FragmentTransaction ft = getSupportFragmentManager().beginTransaction();
                //为dart模块初始化所设置的参数
                ft.replace(R.id.container, Flutter.createFragment("{name:'yangdxg',dataList:['aa','bb','cc']}"));
                ft.commit();
            }
        });
```

```
    <Button
        android:id="@+id/btn_create"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="Test" />

    <FrameLayout
        android:id="@+id/container"
        android:layout_width="match_parent"
        android:layout_height="wrap_content">

    </FrameLayout>
```



Dart代码

```
import 'package:flutter/material.dart';
import 'dart:ui';

//使用window.defaultRouteName获取Native传递过来的参数
void main() => runApp(MyApp(initParams: window.defaultRouteName,));

class MyApp extends StatelessWidget {

  final String initParams;

  const MyApp({Key key, this.initParams}) :super(key: key);

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      title: 'Flutter 混合开发',
      theme: ThemeData(
        primarySwatch: Colors.blue,
      ),
      home: MyHomePage(title: 'Flutter 混合开发', initParams: initParams),
    );
  }
}

class MyHomePage extends StatefulWidget {
  MyHomePage({Key key, this.title, this.initParams}) : super(key: key);

  final String title;
  final String initParams;

  @override
  _MyHomePageState createState() => _MyHomePageState();
}

class _MyHomePageState extends State<MyHomePage> {
  int _counter = 0;

  void _incrementCounter() {
    setState(() {
      _counter++;
    });
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: Text(widget.title),
      ),
      body: Center(
        child: Column(
          mainAxisAlignment: MainAxisAlignment.center,
          children: <Widget>[
            Text(
              'initParams:${widget.initParams}',
            ),
            Text(
              '$_counter',
              style: Theme
                  .of(context)
                  .textTheme
                  .display1,
            ),
          ],
        ),
      ),
      floatingActionButton: FloatingActionButton(
        onPressed: _incrementCounter,
        tooltip: 'Increment',
        child: Icon(Icons.add),
      ), // This trailing comma makes auto-formatting nicer for build methods.
    );
  }
}

```

![Android调用Flutter](http://ww2.sinaimg.cn/large/006tNc79ly1g3az6xs369g308q0emdg0.gif)

#### 热重启/重新加载

混合开发后热重启/重新加载会失效,我们可以通过把flutter_moudle和Android工程进行关联恢复热重启/重新加载

- 打开模拟器或将设备连接到电脑上

- 关闭App,运行flutter attach

  ```
  cd flutter_hybrid/flutter_module
  flutter attach
  ```

  注:如果同时有多个模拟器或连接的设备,运行flutter attach时会提示选择设备
  
  ```
  flutter attach -d 'emulator-5554'
  ```
  
  出现等待连接后运训Android程序`Waiting for a connection from Flutter on Android SDK built for x86...`
  
  flutter部分运行后会出现如下提示
  
  ```
  Done.
  Syncing files to device Android SDK built for x86...                    
   2,243ms (!)                                       
  
  🔥  To hot reload changes while running, press "r". To hot restart (and rebuild state), press "R".
  An Observatory debugger and profiler on Android SDK built for x86 is available at: http://127.0.0.1:55352/QMwyyeCLcSc=/
  For a more detailed help message, press "h". To detach, press "d"; to quit, press "q".
  ```
  
  - 热加载按r
  - 热重启按R
  - 请求帮助按h
  - 断开链接按d
  - 推出按q
  
  更改代码后按下r设备就会更新显示

#### 调试Dart代码

- 关闭APP
- 点击AndroidStudio的Flutter Attach按钮(必须安装好Flutter与Dart插件)
- 启动APP

#### 发布应用

##### 打包

###### 生成Android签名证书

为Android项目生成签名证书(就是原生操作)

###### 配置gradle

- 将签名证书copy到android/app目录下

- 编辑~/.gradle/gradle.properties或../android/gradle.properties(一个全局的,一个项目中的),加入如下代码

  ```
  MYAPP_RELEASE_STORE_FILE = your keystore filename
  MYAPP_RELEASE_KEY_ALIAS = your keystore alias
  MYAPP_RELEASE_STORE_PASSWORD = ******
  MYAPP_RELEASE_KEY_PASSWORD = ******
  ```

- 编辑 android/app/build.gradle文件

  ```
  android{
  	defaultConfig{...}
  	signingConfigs{
  		release{
  			storeFile file(MYAPP_RELEASE_STORE_FILE)
  			storePassword MYAPP_RELEASE_STORE_PASSWORD
  			keyAlias MYAPP_RELEASE_KEY_ALIAS
  			keyPassword MYAPP_RELEASE_KEY_PASSWORD
  		}
  	}
  	buildTypes{
  		release{
  			...
  			signingConfig signingConfigs.release
  		}
  	}
  }
  ```

###### 签名打包APK

terminal进入项目下的Android目录,运行如下代码

```
./gradlew assembleRelease
```

打包完成apk在app/build/outputs/apk

### Flutter iOS混合开发

~~**稍后补充**~~

### Flutter与Native通信

Flutter与Native的通信是通过Channel来完成的

消息使用Channel(平台通道)再客户端(UI)和主机(平台之间传递)

Flutter中的消息传递完全是异步的

Channel所支持的数据类型

| Dart                       | Android             | iOS                                            |
| -------------------------- | ------------------- | ---------------------------------------------- |
| null                       | null                | nil (NSNull when nested)                       |
| bool                       | java.lang.Boolean   | NSNumber numberWithBool:                       |
| int                        | java.lang.Integer   | NSNumber numberWithInt:                        |
| int, if 32 bits not enough | java.lang.Long      | NSNumber numberWithLong:                       |
| double                     | java.lang.Double    | NSNumber numberWithDouble:                     |
| String                     | java.lang.String    | NSString                                       |
| Uint8List                  | byte[]              | FlutterStandardTypedData typedDataWithBytes:   |
| Int32List                  | int[]               | FlutterStandardTypedData typedDataWithInt32:   |
| Int64List                  | long[]              | FlutterStandardTypedData typedDataWithInt64:   |
| Float64List                | double[]            | FlutterStandardTypedData typedDataWithFloat64: |
| List                       | java.util.ArrayList | NSArray                                        |
| Map                        | java.util.HashMap   | NSDictionary                                   |

Flutter定义了三种不同类型的Channel:

- BasicMessageChannel:用于传递字符串和半结构化的信息,持续通信,接收到消息后可以回复此消息,
- MethodChannel:用于传递方法调用,一次性通信,如Flutter调用Native拍照
- EventChannel:用于数据流的通信,持续通信,收到消息后无法回复此消息,通常用于Native向Dart的通信,如网络变化,传感仪检测

#### BasicMessageChannel用法

##### Dart端

```
const BasicMessageChannel(this.name,this.codec);
```

- String name — Channel 的名字,要和Native端保持一致;
- MessageCodec<T> codec — 消息的编解码器,要和Native端保持一致,有四种实现(见Native端讲解)

**sendMessageHandler方法**

```
void	sendMessageHandle(Future<T> handler(T message))
```

- Future<T> handler(T message) — 消息处理器,配置BinaryMessage完成消息的处理

在创建好BasicMessageChannel后,如果要让其接收Native发来的消息,则需要调用它的setMessageHandler方法为其设置一个消息处理器

```
Future<T> send(T message)
```

- T message — 要传递给Native的具体消息
- Future<T> — 消息发出去后,收到Native回复的回调函数

#### MessageChannel方法

##### Dart端

```
const MethodChannel(this.name[this.codec=const StandardMethodCodec])
```

- name — Channel的名字,和Native端保持一致
- MethodCodec codec — 消息的编解码器,默认时StandardMethodCodec,和Native端保持一致

invokeMethod方法

```
Future<T> invokeMethod<T>(String method,[dynamic arguments])
```

- method:要调用Native的方法名

- [dynamic arguments]:调用Native方法传递的参数,可不传;

#### EvnetChannel

##### Dart端

```
const EventChannel(this.name,[this.codec = const StandardMethodCodec()])
```

- name — Channel的名字,要和Native端保持一致
- MethodCodec codec — 消息的编解码器,默认是StandardMethodCodec,要和Native端保持一致

receiveBroadcastStream

```
Stream<dynamic> receivedBroadcastStream([dynamic arguments])
```

- dynamic arguments — 监听事件时向Native传递的数据;

### Flutter与Android通信

#### BasicMessageChannel用法

##### Native端

```
BasicMessageChannel(BinaryMessenger messenger, String name, MessageCodec<T> codec) {
```

- BinaryMessenger — 消息信使,是消息的发送与接收的工具
- String name — Channel的名字,也是其唯一标识符
- MessageCodec<T> — 消息的编解码器,它有几种不同类型的实现
  - BinaryCodec — 最为简单的一种Codec,其返回值类型和入参的类型相同,均为二进制格式(Android 中为ByteBuffer,iOS中为NSData),实际上,BinarCodec在编解码过程中什么都没做,只是原封不动将二进制数据消息返回而已,BinaryCodec并非没有意义,在某些情况下非常有用,比如使用BinaryCodec可以传递内存数据块时在编解码阶段等于内存拷贝
  - StringCodec — 用于字符串与二进制数据之间的编解码,其编码格式为UTF-8
  - JSONMessageCodec — 用于基础树与二进制数据之间的编解码,其支持基础数据类型以及列表,字典,其在iOS端使用NSJSONSerialization作为序列化的工具,而在Android端则使用了其自定义的JSONUTIL与StringCodec作为序列化工具;
  - StandardMessageCode — 是BasicMessageChannel的默认编解码器,支持基础数据类型,二进制数据,列表,字典,

setMessageHandle接收消息

```
void setMessageHandler(BasicMessageChannel.MessageHandler<T> handler)
```

- Handler — 消息处理器,配合BinaryMessenger完成消息的处理

创建好BasicMessageChannel后,如果要让其接收Dart发来的消息,则需要调用它的setMessageHandler方法为其设置一个消息处理器

```
public interface MessageHandler<H>{
	void onMessage(T var1,BasicMessageChannel.Repl<T> var2);
}
```

- onMessage(T var,BasicMessageChannel Reply<T> var2) — 用于接收消息,var1是消息内容,var2是回复此消息的回调函数

send方法,向Dart端发送消息

```
void send(T message);
void send(T message,BasicMessageChannel.Reply<T> callback)
```

- T message — 要传递给Dart的具体信息
- BasicMessageChannel.Reply<T> callback — 消息发出后,收到Dart 回复的回调函数

#### MethodChannel用法

##### Native端

```
//会构造一个StandardMethodCodec.INSTANCE类型的MethodCodec
MethodChannel(BinaryMessage messager,String name)
//或
MethodChannel(BinaryMessage message,String name,MethodCodec codec)
```

- messager — 消息信使
- name — Channel名字
- codec — 编解码器

```
setMethodCallhandler(@Nullable MethodChannel.MethodCallHandler handler)
```

- Handler — 消息处理器,配合BinaryMessenger完成消息的处理;

```
public interface MethodCallHandler{
	void onMethodCall(MethodCall var1,MethodChannel.Result var2);
}
```

- onMessageCall — 用于接收消息,call是消息内容,call.method表示调用的方法名,Object类型的call.arguments表示调用方法所传递的入餐;result是回复此消息的回调函数提供了success,error,noImplemented方法调用

#### EventChannel用法

##### Native端

```
EventChannel(BinaryMessenger messenger, String name) 
EventChannel(BinaryMessenger messenger, String name, MethodCodec codec)
```

- 参数意义同上

接收消息

```
setStreamHandler(EventChannel.StreamHandler handler)
```

```
public interface StreamHandler{
	void onListen(Object args,EventChannel.EventSink eventSink);
	void onCancle(Object o);
}
```

- eventSink回调函数
- onCancle取消监听检测

### Flutter与Android通信实战

### Flutter与iOS通信