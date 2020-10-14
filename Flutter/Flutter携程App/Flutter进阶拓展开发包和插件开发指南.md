### Flutter包和插件的开发流程步骤

#### package介绍

- 一个pubspec.yaml文件(定义包的生命)
- 一个lib文件夹(存放代码)

#### package类型

- Dart包
- 插件包

#### 创建包或插件

- 可视化的方式(推荐)
- 命令行的方式

##### 可视化方式

创建方式: 使用AndroidStudio =》File =》new Flutter Project…=>Flutter Plugin(插件) /Flutter Package(包)

###### 插件包含文件

- lib中包含 MethodChannel和Native进行通信
- android中包含android对应代码
- ios中包含iOS 中对应代码
- pubspec.yaml声明文件

###### 包包含文件

- lib,Dart部分代码
- pubspec.yaml声明文件
- 不包含Android.ios文件

##### 命令行方式

创建插件

```
flutter create --org org.yangdx(域名) --template=plugin(插件) demo(名字)
```

创建包

```
flutter create --template=package flutter_color_plugin(名字)
```

### Flutter包和插件开发与发布

创建一个Flutter包flutter_color_plugin将字符串颜色解析成FLutter中Color颜色

###### 代码编写

在flutter_color_plugin.dart中编写如下代码

```
library flutter_color_plugin;

import 'dart:ui';

/// A Calculator.
class ColorUtil {
  static const int BLACK = 0xFF000000;
  static const int DKGRAY = 0xFF444444;
  static const int GRAY = 0xFF888888;
  static const int LTGRAY = 0xFFCCCCCC;
  static const int WHITE = 0xFFFFFFFF;
  static const int RED = 0xFFFF0000;
  static const int GREEN = 0xFF00FF00;
  static const int YELLOW = 0xFFFFFF00;
  static const int CYAN = 0xFF00FFFF;
  static const int MAGENTA = 0xFFFF00FF;
  static const int TRANSPARENT = 0;

  /**
   * 将String颜色转换成int16禁止的颜色
   */
  static int intColor(String colorString) {
    if (colorString?.isEmpty ?? true) {
      throw ArgumentError('Unknow color');
    }
    if (colorString[0] == '#') {
      int color = int.tryParse(colorString.substring(1), radix: 16);
      if (colorString.length == 7) {
        color = 0x00000000ff000000;
      } else if (colorString.length != 9) {
        throw ArgumentError('Unknow color');
      }
      return color;
    } else {
      int color = sColorNameMap[(colorString.toLowerCase())];
      if (color != null) {
        return color;
      } else {
        return intColor('#' + colorString);
      }
    }
  }

  /**
   * 将String颜色转换成Color类型
   */
  static Color color(String colorString) {
    return Color(intColor(colorString));
  }

  static const sColorNameMap={
    'black':BLACK,
    'darkgray':DKGRAY,
    'gray':GRAY,
    'lightgray':LTGRAY,
    'white':WHITE,
    'red':RED,
    'green':GREEN,
    'yellow':YELLOW,
    'cyan':CYAN,
    'magenta':MAGENTA,
    'aqua':0xFF00FFFF,
    'fuchsia':0xFFFF00FF,
    'dartgrey':DKGRAY,
    'grey':GRAY,
    'lightgrey':LTGRAY,
    'lime':0xFF00FF00,
    'maroon':0xFF800000,
  };

}
```



###### pubspec.yaml添加项目描述

```
name: flutter_color_plugin
#描述
description: flutter_color_plugin
#版本号
version: 0.0.1
#作者
author: yangdxg
#仓库github地址
homepage:
```

- 可以在README中添加使用手册
- 在LICENSE添加开源协议
- CHANGELOG中添加版本记录

###### 发布插件

- 检查包是否准备成功

```
yangdongxingdeMacBook-Pro:flutter_color_plugin yangdongxing$ flutter packages pub publish --dry-run
Publishing flutter_color_plugin 0.0.1 to https://pub.flutter-io.cn:
|-- .gitignore
|-- .metadata
|-- CHANGELOG.md
|-- LICENSE
|-- README.md
|-- lib
|   '-- flutter_color_plugin.dart
|-- pubspec.yaml
'-- test
    '-- flutter_color_plugin_test.dart

Package has 0 warnings.
```

- 发布插件

  ```
  flutter packages pub publish
  #会提示使用Google账号登陆,授权后会自动上传
  #我这里pub一直链接不上google导致下面错误(稍后等网络情况好的时候再传)
  #Waiting for your authorization...
  #Authorization received, processing...
  #It looks like accounts.google.com is having some trouble.
  #Pub will wait for a while before trying to connect again.
  ```

  发布成功后直接在https://pub.dev/flutter中搜索即可

注:解决依赖冲突

- 如果项目同时依赖了A,B,
- A,依赖C :0.1,B依赖C:0.2
- 只需要再依赖上C:0.2即可

