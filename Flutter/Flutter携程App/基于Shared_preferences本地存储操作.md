### 如何使用shared_preferences

pubspec.yaml文件中添加依赖

```
dependencies:
  flutter:
    sdk: flutter

  shared_preferences:^0.5.1+
```

导入头文件

```
import 'package:shared_preferences/shared_preferences.dart';
```

存储数据

```
    final prefs = await SharedPreferences.getInstance();
    prefs.setInt('counter', counter);
```

读取使用

```
    final prefs = await SharedPreferences.getInstance();
    final counter = prefs.getInt('counter') ?? 0;
```

移除数据

```
    final prefs = await SharedPreferences.getInstance();
    prefs.remove('counter');
```



### shared_preferences有哪些常用的API

- shared_preferences支持int,double,bool,string与stringList类型的数据存储

- 对应API为setString,setBool等,读取为getString,getBool,直接使用get获取到的是dynamic类型

- getKeys可以获取到所有的key

### 基于shared_preferences实现计数器Demo

```
import 'package:flutter/material.dart';
import 'package:shared_preferences/shared_preferences.dart';

void main() {
  runApp(MaterialApp(
    home: Scaffold(
      appBar: AppBar(title: Text("shared_preference"),),
      body:_CounterWidget(),
    ),
  ));
}

class _CounterWidget extends StatefulWidget {
  @override
  _CounterWidgetState createState() => _CounterWidgetState();
}

class _CounterWidgetState extends State<_CounterWidget> {
  String countString = '';
  String localCount = '';

  @override
  Widget build(BuildContext context) {
    return Center(
      child: Column(
        children: <Widget>[
          RaisedButton(onPressed: _incrementCounter, child: Text("增加"),),
          RaisedButton(onPressed: _getCounter, child: Text('获取'),),
          Text(
            countString,
            style: TextStyle(fontSize: 20),
          ),
          Text(
            'result' + localCount,
            style: TextStyle(fontSize: 20),
          )
        ],
      ),
    );
  }

  _incrementCounter() async {
    SharedPreferences prefs = await SharedPreferences.getInstance();
    setState(() {
      countString = countString + "1";
    });
    int counter = (prefs.getInt('counter') ?? 0) + 1;
    await prefs.setInt('counter', counter);
  }

  _getCounter() async {
    SharedPreferences prefs = await SharedPreferences.getInstance();
    setState(() {
      localCount = prefs.getInt('counter').toString();
    });
  }
}
```

![](http://ww3.sinaimg.cn/large/006tNc79ly1g396drjyoeg308v07o74r.gif)

注:使用苹果设备xcode中需要安装cocoapods

`brew install cocoapods`

