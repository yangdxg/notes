### 基于ListView实现水平和垂直方向滚动列表

#### 实现垂直滚动列表

```
import 'package:flutter/material.dart';

void main() => runApp(MyApp());

const CITY_NAMES = ['北京', '上海', '广州', '深圳', '海南', '杭州', '苏州', '温州'];

class MyApp extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      title: 'Flutter Demo',
      theme: ThemeData(
        primarySwatch: Colors.blue,
      ),
      home: Scaffold(
        appBar: AppBar(title: Text('ListView'),),
        body: ListView(
          children: _buildList(),
        ),
      ),
    );
  }

  List<Widget> _buildList() {
    return CITY_NAMES.map((city) => _item(city)).toList();
  }

  Widget _item(String city) {
    return Container(
      height: 80,
      margin: EdgeInsets.only(bottom: 5),
      alignment: Alignment.center,
      decoration: BoxDecoration(color: Colors.teal),
      child: Text(
        city,
        style: TextStyle(color: Colors.white, fontSize: 20),
      ),
    );
  }

}
```

也可以使用ListView.Builder更加高效的实现列表(适用于动态列表或者数据来给你较大时)

```
      home: Scaffold(
        appBar: AppBar(title: Text('ListView'),),
        body: ListView.builder(
          itemCount: CITY_NAMES.length,
          itemBuilder: (BuildContext context, int position) {
            return _buildList()[position];
          },
        ),
      ),
```



#### 实现水平滚动列表

水平滚动只需要为ListView添加scrollDirection: Axis.horizontal参数即可

```
      home: Scaffold(
        appBar: AppBar(title: Text('ListView'),),
        body: ListView(
          scrollDirection: Axis.horizontal,
          children: _buildList(),
        ),
      ),
```

- 为ListView设置高度需要对ListView外层Container设置高度

### 基于ExpansionTitle实现可展开的列表

#### ExpansionTitle基本知识

```
const ExpansionTile({
    Key key,
    this.leading,//标题左侧要展示的widget
    @required this.title,//要展示的标题widget(是整个标题父控件)
    this.backgroundColor,//背景
    this.onExpansionChanged,//列表展开收起的回调函数
    this.children = const <Widget>[],//列表展开时显示的widget
    this.trailing,//标题右侧要展示的widget
    this.initiallyExpanded = false,//是否默认状态下展开
  }) : assert(initiallyExpanded != null),
       super(key: key);
```

- 数据要求

  ```
  const CITY_NAMES = {
    '北京': ['东城区', '西城区', '丰台区', '海淀区', '房山区'],
    '上海': ['黄埔区', '徐汇区', '虹口区'],
    '杭州': ['上城区', '下城区']
  };
  ```

- ExpansionTitle

  ```
  import 'package:flutter/material.dart';
  
  void main() => runApp(MyApp());
  
  const CITY_NAMES = {
    '北京': ['东城区', '西城区', '丰台区', '海淀区', '房山区'],
    '上海': ['黄埔区', '徐汇区', '虹口区'],
    '杭州': ['上城区', '下城区']
  };
  
  class MyApp extends StatelessWidget {
    @override
    Widget build(BuildContext context) {
      return MaterialApp(
        title: 'ExpansionTile Demo',
        theme: ThemeData(
          primarySwatch: Colors.blue,
        ),
        home: Scaffold(
          appBar: AppBar(title: Text('ListView'),),
          body: ListView(
            children: _buildList(),
          ),
        ),
      );
    }
  
    List<Widget> _buildList() {
      List<Widget> widgets = [];
      CITY_NAMES.keys.forEach((key) {
        widgets.add(_item(key, CITY_NAMES[key]));
      });
      return widgets;
    }
  
    Widget _item(String city, List<String> subCities) {
      return ExpansionTile(
        title: Text(
          city,
          style: TextStyle(color: Colors.black54, fontSize: 20),
        ),
        children: subCities.map((subCities) => _buildSub(subCities)).toList(),
      );
    }
  
    Widget _buildSub(String subCity) {
      return FractionallySizedBox(//可伸缩（如果不设置FractionallySizedBox，子列表宽度无法撑满屏幕）
        widthFactor: 1,
        child: Container(
          height: 50,
          margin: EdgeInsets.only(bottom: 5),
          decoration: BoxDecoration(color: Colors.lightBlueAccent),
          child: Text(subCity),
        ),
      );
    }
  
  }
  ```

  ![](http://ww1.sinaimg.cn/large/006tNc79ly1g39v581kisg30830c3dg2.gif)

### 基于GridView实现网格列表

GridView时flutter中用于展示网格布局风格的widget,通常使用GridView.count构造函数来创建一个GridView

与ListView用法一直,只需要使用GridView.count构建即可

```
      home: Scaffold(
        appBar: AppBar(title: Text('ListView'),),
        body: GridView.count(
          crossAxisCount: 2,//列数
          children: _buildList(),
        ),
      ),
```

![](http://ww4.sinaimg.cn/large/006tNc79ly1g39vbmrlfsg30830c3dgf.gif)

### 高级功能列表下拉刷新与上拉加载更多

#### 实现下拉刷新

使用RefreshIndicator实现列表的下拉刷新

#### 实现上拉加载

借助ScrollController,为列表设置controller参数,通过ScrollController监听列表滚动的位置,实现加载更多功能

```
import 'package:flutter/material.dart';

void main() => runApp(MyApp());

List<String> cityNames = [
  '北京', '上海', '广州', '深圳', '海南', '杭州', '苏州', '温州', '泉州', '保定', '郑州', '唐山', '东京'];

class MyApp extends StatefulWidget {

  @override
  _MyAppState createState() => _MyAppState();

}

class _MyAppState extends State<MyApp> {

  ScrollController _scrollController = ScrollController();

  @override
  void initState() {
    _scrollController.addListener(() {
      if (_scrollController.position.pixels ==
          _scrollController.position.maxScrollExtent) { //当前位置等于可滚动位置
        _loadData();
      }
    });
    super.initState();
  }

  @override
  void dispose() {
    //在声明周期结束的时候将监听移除
    _scrollController.dispose();
    super.dispose();
  }

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      title: '下拉刷新',
      theme: ThemeData(
        primarySwatch: Colors.blue,
      ),
      home: Scaffold(
          appBar: AppBar(title: Text('ListView'),),
          body: RefreshIndicator(
              child: ListView(
                controller: _scrollController,
                children: _buildList(),
              ),
              onRefresh: _handleRefresh)
      ),
    );
  }


  List<Widget> _buildList() {
    return cityNames.map((city) => _item(city)).toList();
  }

  Widget _item(String city) {
    return Container(
      height: 80,
      margin: EdgeInsets.only(bottom: 5),
      alignment: Alignment.center,
      decoration: BoxDecoration(color: Colors.teal),
      child: Text(
        city,
        style: TextStyle(color: Colors.white, fontSize: 20),
      ),
    );
  }

  Future<Null> _handleRefresh() async {
    await Future.delayed(Duration(seconds: 2));
    setState(() {
      cityNames = cityNames.reversed.toList();
    });
    return null;
  }

  _loadData() async {
    await Future.delayed(Duration(milliseconds: 200));
    setState(() {
      List<String> list = new List <String>.from(cityNames);
      list.addAll(cityNames);
      cityNames = list;
    });
  }

}
```

![](http://ww3.sinaimg.cn/large/006tNc79ly1g39xyt2y77g30830hwwm9.gif)