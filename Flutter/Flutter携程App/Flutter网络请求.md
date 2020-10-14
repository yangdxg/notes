### 基于Http实现网络操作 

#### GET请求

- 引入http插件`http: ^0.12.0+1`

#### POST请求

#### response转换成Dart object

#### 将结果展示到UI上

```
import 'package:flutter/material.dart';
import 'dart:convert';
import 'package:http/http.dart' as http;

void main() => runApp(new MyApp());

class MyApp extends StatefulWidget {
  @override
  _MyAppState createState() => _MyAppState();
}

class _MyAppState extends State<MyApp> {
  String showResult = '';

  Future<CommonModel> fetchPost() async {
    final response = await http.get(
        'http://www.devio.org/io/flutter_app/json/test_common_model.json');
    final result = json.decode(response.body);
    return CommonModel.fromJson(result);
  }

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      home: Scaffold(
        appBar: AppBar(title: Text('http'),),
        body: Column(
          children: <Widget>[
            InkWell(
              onTap: () {
                fetchPost().then((CommonModel value) {
                  setState(() {
                    showResult = '请求结果${value.hideAppBar}';
                  });
                });
              },
              child: Text('点击', style: TextStyle(fontSize: 28),),
            ),
            Text(showResult)
          ],
        ),
      ),
    );
  }

}

class CommonModel {
  final String icon;
  final String title;
  final String url;
  final String statusBarColor;
  final bool hideAppBar;

  CommonModel({this.icon, this.title, this.url, this.statusBarColor,
    this.hideAppBar});

  factory CommonModel.fromJson(Map<String, dynamic> json){
    return CommonModel(
        icon: json['icon'],
        title: json['title'],
        url: json['url'],
        statusBarColor: json['statusBarColor'],
        hideAppBar: json['hideAppBar']);
  }

}
```



### 异步:Fluture和FutureBuilder实用技巧

#### Future

接下来的某个时间的值或错误,借助Future可以实现Flutter异步操作

Future有俩种状态

- pending-执行中
- completed— 执行结束(分成功失败俩种情况)

#### Future用法

- future.then获取future的值与捕获future的异常

  ```
  Future<String> testFuture(){
  	return Future.value('success');
  	return Future.error('error');
  }
  
  main(){
  	testFuture().then((s){
  		print(s);
  	},onError(e){
  		print(e)
  	}).catchError((e){
  		print(e)
  	})
  }
  ```

  如果catchError与onError同时存在,则只会调用onError

  Future是异步编程

- 结合async,await

  可以获取到异步编程的返回值

- future.whenComplete

  就是trycatch中的finally

- future.timeout

  设置超时时间

#### FutureBuilder

FutureBuilder是一个将异步操作和异步UI更新结合在一起的类,通过它我们可以将网络请求,数据库读取等的结果更新在页面上

#### FutureBuilder用法

```
class FutureBuilder<T> extends StatefulWidget {
  const FutureBuilder({
    Key key,
    this.future,//future对象表示此构造起当前链接的异步计算
    this.initialData,//表示一个非空的Future完成前的初始化数据
    @required this.builder,//AsyncWidgetBuilder类型的回调函数,是一个基于异步交互构建widget的函数;
  }) : assert(builder != null),
       super(key: key);
```

builder函数接收俩个参数BuildContext context 与AsyncSnapshot<T> snapshot,返回一个widget,

FutureBuilder作为一个Widget使用

```
import 'package:flutter/material.dart';

import 'dart:convert';

import 'package:http/http.dart' as http;

class SearchPage extends StatefulWidget {
  @override
  _SearchPageState createState() => _SearchPageState();

}

class _SearchPageState extends State<SearchPage> {

  String showResult = '';

  Future<CommonModel> fetchPost() async {
    final response = await http.get('http://www.devio.org/io/flutter_app/json/test_common_model.json');
    Utf8Decoder utf8decoder = Utf8Decoder(); //中文乱码
    var result = json.decode(utf8decoder.convert(response.bodyBytes));
    return CommonModel.fromJson(result);
  }

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      home: Scaffold(
        appBar: AppBar(title: Text('http')),
        body: FutureBuilder<CommonModel>(
          future: fetchPost(),
          builder: (BuildContext context,
              AsyncSnapshot<CommonModel> snapshot) {
            switch (snapshot.connectionState) { //当前链接状态
              case ConnectionState.none:
                return new Text('Input a URL to Start');
              case ConnectionState.waiting: //显示进度条
                return new Center(child: new CircularProgressIndicator(),);
                break;
              case ConnectionState.active:
                return new Text('');
                break;
              case ConnectionState.done: //完成了
                if (snapshot.hasError) {
                  return new Text(
                    '${snapshot.error}', style: TextStyle(color: Colors.red),);
                } else {
                  return new Column(children: <Widget>[
                    Text('icon:${snapshot.data.icon}'),
                    Text('statusBarColor:${snapshot.data.statusBarColor}'),
                    Text('title:${snapshot.data.title}'),
                    Text('url:${snapshot.data.url}'),
                  ],);
                }
                break;
            }
          },
        ),
      ),
    );
  }
}


class CommonModel {
  final String icon;
  final String title;
  final String url;
  final String statusBarColor;
  final bool hideAppBar;

  CommonModel({this.icon, this.title, this.url, this.statusBarColor,
    this.hideAppBar});

  factory CommonModel.fromJson(Map<String, dynamic> json){
    return CommonModel(
        icon: json['icon'],
        title: json['title'],
        url: json['url'],
        statusBarColor: json['statusBarColor'],
        hideAppBar: json['hideAppBar']);
  }

}
```



### Json解析和复杂模型转换实用技巧

应该使用哪种JSON序列化方式

- 小型项目:手动序列化;
- 大型项目借助插件生成json_serializable和built_valut;

### 基于shared_preferences本地存储操作