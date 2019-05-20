[TOC]



##### Flutter中的View

###### Widget定义

Flutter中可以将Widget当作Android,iOS中的View

Widget与View的qubie

- Widget具有不同的生命周期,是不可变的,回存在于状态被改变之前,每当Widget或其状态发生变化时,Flutter的框架就会创建一个新的Widget实例树,而Android/iOS视图植被绘制一次,并且在调用invalidate/setNeedsDisplay之前不会重绘
- Flutter的Widget很轻巧,原因在于它的不变性,因为本身不是视图,并且不是直接绘制任何东西,而是对UI及其语义的描述

###### 更新Widgets

Widget时不可变的,不会直接更新,我们可以通过Widget的状态来更新他们,这就是有状态和无状态Widget的概念来源,StatelessWidget就是一个没有状态信息的Widget

StatelessWidgets适用于当我们描述的用户界面不依赖于对象中的配置信息时

- 例如Android/iOS中使用ImageView/UIImageView显示logo,logo在运行时不会改变,因此在Flutter中使用StatelessWidget最合适
- 如果根据Http网络请求等受到数据动态更改UI,则必须使用StatefulWidget并通知Flutter框架Widget的状态一更新了,以便更新Widget
- 有状态和无状态Widget最重要的区别在于StatefulWidgets具有一个State对象,该对象存储状态数据并将其传递到树重建中,因此状态不会丢失
- 规则:如果Widget在build之外更改(例如运行时用户交互),则它是有状态的,如果Widget永远不会改变,一旦构建,它就是无状态的,但是即使Widget是有状态的,如果包含它的父窗口小部件本身不对这些更改(或其它输入)作出反应,父Widget仍然可以是无状态的

使用StatelessWidget

Text就是一个常见的StatelessWidget它是StatelessWidget的子类(仅呈现构造中传递的内容)

```
new Text('测试输出',
              style: new TextStyle(fontWeight: FontWeight.bold),)
```

如果想让构造中传入的内容变化,可以将Text包装在StatefulWidget中并更新它来实现,下面点击FLoation更改文本内容

```
class MyApp extends StatelessWidget {
  // This widget is the root of your application.
  final index = 6;

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
        title: 'Welcome',
        theme: ThemeData(
          primarySwatch: Colors.blue,
        ),
        home: SampleAppPage()
    );
  }
}

class SampleAppPage extends StatefulWidget {

  SampleAppPage({Key key}) :super(key: key);

  @override
  State<StatefulWidget> createState() => _SampleAppPageState();

}

class _SampleAppPageState extends State<SampleAppPage> {
  String textShow = "I learn Flutter";

  void _updateText() {
    setState(() {
      textShow = "I always learn Flutter";
    });
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: Text("Sample APp"),
      ),
      body: Center(child: Text(textShow),),
      floatingActionButton: FloatingActionButton(
        onPressed: _updateText,
        tooltip: "Update Text",
        child: Icon(Icons.update),
      ),
    );
  }

}
```

##### Flutter布局

在Flutter中使用widget树声明布局

```
  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: Text("Sample APp"),
      ),
      body: Center(child: Text(textShow),),
      floatingActionButton: FloatingActionButton(
        onPressed: _updateText,
        tooltip: "Update Text",
        child: Icon(Icons.update),
      ),
    );
  }
```

###### 如何在布局中添加删除组件

类似于Android中的addChild或removeChild

在Flutter中,因为Widget是不可变的,所以没有类似的方法,我们可以传入一个函数或表达式,该函数或表达式返回一个Widget给父项,并通过布尔值控制该Widget的创建

使用函数或表达式动态返回Widget

```
class _SampleAppPageState extends State<SampleAppPage> {
  String textShow = "I learn Flutter";
  bool toggle = true;

  void _updateText() {
    setState(() {
      textShow = "I always learn Flutter";
    });
  }

  void _toggle() {
    setState(() {
      toggle = !toggle;
    });
  }

  _getToggleChild() {
    if (toggle) {
      return Text("开始的");
    } else {
      return MaterialButton(onPressed: () {}, child: Text("更改后"),);
    }
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: Text("Sample APp"),
      ),
      body: Center(child: _getToggleChild(),),
      floatingActionButton: FloatingActionButton(
        onPressed: _toggle,
        tooltip: "Update Text",
        child: Icon(Icons.update),
      ),
    );
  }

}
```

###### Widget动画

使用动画库包裹widgets,使用AnimationController,可以暂停,寻找,停止,反转动画的Animation类型,需要一个Ticker当vsync发生时发送信号,并在每帧运行时创建一个介于0-1之间的线形差值(interpolation),可以创建一个或多个Animation附加给一个controller

例:用CurvedAnimation实现一个interpolated曲线,controller是动画过程的主人,CurvedAnimation计算曲线,并替代controller默认的线形模式

构建widget树时,把Animation指定给一个widget的动画属性,比如FadeTransition的opacity,并告诉控制器开始动画

```
class MyApp extends StatelessWidget {
  // This widget is the root of your application.
  final index = 6;

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      title: 'Welcome',
      theme: ThemeData(
        primarySwatch: Colors.blue,
      ),
//        home: SampleAppPage()
      home: MyFadeTest(title: 'Fade Demo'),
    );
  }
}

class MyFadeTest extends StatefulWidget {
  final String title;

  MyFadeTest({Key key, this.title}) :super(key: key);

  @override
  _MyFadeTest createState() => _MyFadeTest();
}

class _MyFadeTest extends State<MyFadeTest> with TickerProviderStateMixin {
  AnimationController controller;
  CurvedAnimation curve;

  @override
  void initState() {
    super.initState();
    controller = AnimationController(
        duration: const Duration(milliseconds: 2000), vsync: this);
    curve = CurvedAnimation(parent: controller, curve: Curves.easeIn);
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: Text(widget.title),
      ),
      body: Center(
        child: FadeTransition(opacity: curve,
          child: FlutterLogo(
            size: 100.0,
          ),),
      ),
      floatingActionButton: FloatingActionButton(
          tooltip: 'Fade',
          child: Icon(Icons.brush),
          onPressed: () {
            controller.forward();
          }),
    );
  }
}
```

###### 绘图

Flutter有俩个类可以帮助我们绘制画布,CustomPaint和CustomPainter,实现算法可以绘制到画布

例:手指绘图

```
void main() => runApp(MaterialApp(home: MyApp(),));

class MyApp extends StatelessWidget {
  // This widget is the root of your application.
  final index = 6;

  @override
  Widget build(BuildContext context) =>
      Scaffold(body: Signature());
}


class Signature extends StatefulWidget {
  SignatureState createState() => SignatureState();
}

class SignatureState extends State<Signature> {

  List<Offset> _points = <Offset>[];

  @override
  Widget build(BuildContext context) {
    return GestureDetector(
      onPanUpdate: (DragUpdateDetails details) {
        setState(() {
          RenderBox referenceBox = context.findRenderObject();
          Offset location = referenceBox.globalToLocal(details.globalPosition);
          _points = List.from(_points)
            ..add(location);
        });
      },
      onPanEnd: (DragEndDetails details) => _points.add(null),
      child: CustomPaint(
        painter: SignaturePainter(_points), size: Size.infinite,),
    );
  }
}

class SignaturePainter extends CustomPainter {
  final List<Offset> points;

  SignaturePainter(this.points);

  @override
  void paint(Canvas canvas, Size size) {
    var paint = Paint()
      ..color = Colors.black
      ..strokeCap = StrokeCap.round
      ..strokeWidth = 5.0;
    for (int i = 0; i < points.length - 1; i ++) {
      if (points[i] != null && points[i + 1] != null) {
        canvas.drawLine(points[i], points[i + 1], paint);
      }
    }
  }

  @override
  bool shouldRepaint(SignaturePainter other) => other.points != points;
}
```

###### 构建自定义Widgets

在Flutter中推荐组合多个小widgets来构建一个CustomButton,并在构造器中传入label,那就组合RaiseButton和label,而不是扩展ReisedButton

```
void main() => runApp(MaterialApp(home: MyWidgetApp(),));

class MyWidgetApp extends StatelessWidget{
  @override
  Widget build(BuildContext context) {
    // TODO: implement build
    return new Center(
      child: new CustomButton("Hello"),
    );
  }
}

class CustomButton extends StatelessWidget {
  final String label;
  CustomButton(this.label);

  @override
  Widget build(BuildContext context) {
    return new RaisedButton(onPressed: (){},child: new Text(label),);
  }
}
```

为Widget设置透明度

```
Opacity(
	opacity:0.5,
	child:Text("透明度50%")
)
```

##### 布局与列表

###### Linearlayout等价是什么

在Flutter中使用Row或Column widget来实现相同的结果

```
class LinearApp extends StatelessWidget{
  @override
  Widget build(BuildContext context) {
    // TODO: implement build
    return Row(
      mainAxisAlignment: MainAxisAlignment.center,
      children: <Widget>[
        Text("Row One"),
        Text("Row Two"),
        Text("Row Three"),
        Text("Row Four"),
      ],
    );
  }
}
class LinearOrApp extends StatelessWidget{
  @override
  Widget build(BuildContext context) {
    // TODO: implement build
    return Column(
      mainAxisAlignment: MainAxisAlignment.center,
      children: <Widget>[
        Text("Row One"),
        Text("Row Two"),
        Text("Row Three"),
        Text("Row Four"),
      ],
    );
  }
}
```

###### RelativeLayout等价是什么

通过使用Colums,Row和Stack的组合来实现RelativeLayout,可以为widget构造函数指定相对于父组件的布局规则

###### 如何使用widget定义布局属性

在Flutter中,布局主要由专门设计用于提供布局的小部件定义 ,并结合控件widget及其样式属性

例:列和行widgets控制的一个数组中的条目,并且分别垂直和水平对其,Container widget控制一个布局的样式和属性,并且Center widget负责居中它的子widget

```
        Center(
            child:Column(
              children: <Widget>[
                Container(
                  color: Colors.res,
                  width: 100.0,
                  height: 100.0,
                ),  Container(
                  color: Colors.blue,
                  width: 100.0,
                  height: 100.0,
                ),  Container(
                  color: Colors.green,
                  width: 100.0,
                  height: 100.0,
                )
              ],
            )
        )
```

Flutter在其核心widget库中提供了各种布局小部件,例如Padding,Align,和Stack

###### 如何分层布局

Flutter使用Stack widget控制子widget在一层,子widgets可以完全或者部分覆盖widgets.Stack控件将其子项相对于其框的边缘定位,如果您只想重叠多个子窗口小部件,很有用

```
Stack(
          alignment: const Alignment(0.6, 0.6),
          children: <Widget>[
            CircleAvatar(
              backgroundImage: NetworkImage(
                  "https://cdn.jsdelivr.net/gh/flutterchina/website@1.0/images/flutter-mark-square-100.png"
              ),
            ),
            Container(
              decoration: BoxDecoration(
                color: Colors.black45,
              ),
              child: Text('Flutter'),
            )
          ],
        )
        
```

###### 如何设置布局样式

Flutter使用Padding,Center,Column,Row等widget,另外组件也通常接受用于布局样式的构造参数,例如可以创建一个TextStyle类并将其应用于各个Text widget

```
var textStyle = TextStyle(
    fontSize: 32.0, color: Colors.cyan, fontWeight: FontWeight.w600);
    Text('text', style: textStyle,)
```

###### ScrollView在Flutter中等价于什么

在Flutter中,使用ListView,一个ListView既是一个ScrollView,也是一个Android ListView

###### Flutter中的列表组件

在Flutter中没有adapter的等价产物,唯一要做的就是控制这个list中要展示的数据

```
void main() => runApp(ListApp());

class ListApp extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    // TODO: implement build
    return MaterialApp(
      title: "List APP",
      theme: ThemeData(
          primarySwatch: Colors.blue
      ),
      home: ListModel(),
    );
  }
}

class ListModel extends StatefulWidget {
  ListModel({Key key}) :super(key: key);

  @override
  _ListState createState() {
    // TODO: implement createState
    return _ListState();
  }
}

class _ListState extends State<ListModel> {
  @override
  Widget build(BuildContext context) {
    // TODO: implement build
    return Scaffold(
      appBar: AppBar(title: Text("List APP"),),
      body: ListView(children: _getListData()),
    );
  }
}

_getListData() {
  List<Widget> widgets = [];
  for (int i = 0; i < 100; i++) {
    widgets.add(
        Padding(padding: EdgeInsets.all(10.0), child: Text("Row $i"),));
  }
  return widgets;
}
```

###### 动态更新ListView

在Flutter中不能使用setState()方法更新widget俩表,因为setState()被调用时,Flutter渲染引擎会去检查widget树来查看是否有什么地方被改变了,当得到ListView时,会使用==进行判断,发现俩个ListView是相通的,没有什么改变,就不会更新

更新ListView的简单方法是,在setState()中创建一个新的List,并把旧List的数据拷贝给新的list,但数据量大时不建议这样做

高效的做法是使用ListView.Builder来构建列表,这个方法在你想要构建动态列表,或者列表有大量数据时可以使用

##### Flutter状态管理

状态是在构建widget时可以同步读取的信息,或者在widget的生命周期中可能更改的信息,在Flutter中如果要管理状体啊需要用到StatefulWidget

###### 什么是StatelessWidget

Flutter中StatelessWidget是一个不需要状态更改的widget —它没有要管理的内部状态

当用户界面部分不依赖于对象本身中的配置信息以及widget的BuildContext时,无状态widget非常有用

AboutDialog,CircleAvator和Text都是StatelessWidget的子类

无状态widget的build方法通常只会在以下三种情况调用

- 将widget插入树中时
- 当widget的父级更改其配置时
- 当它依赖的InheritedWidget发生变化时

###### 什么是StatefulWidget

可变状态的widget,使用setState方法管理StatefulWidget的状态改变,调用setState告诉Flutter框架,某个状态发生了变化,Flutter会重新运行build方法,更新到最新状态

Checkbox,Radio,Slider,InWell,Form和TextFiled都是有状态的widget

声明一个StatefulWidget,需要一个createState()方法

###### 什么是StatefulWidget和StatelessWidget最佳实践

在Flutter中,widget是有状态的还是无状态的取决于是否他们依赖于状态的变化

- 如果用户交互或数据改变导致widget改变,那么它就是有状态的
- 如果一个widget是最终的或不可变的,那么它就是无状态的

确定那个对象管理widget的状态(StatefulWidget),在Flutter中,管理状态有三种主要方式

- 每个widget管理自己的状态
- 父widget管理widget的状态
- 混合搭配管理的方法

如何决定使用哪种方式

- 如果所讨论的状态是用户数据,例如复选框的已选中或为选中状态,或滑块位置,则状态最好由父widget管理
- 如果widget的状态取决于动作,例如动画,那么最好由widget自身管理状态
- 如果还不确定谁管理中状态,让父widget管理子widget的状态

##### 路由与导航

###### Flutter中Intent等价

Flutter不具有Intents的概念,FLutter可以通过Native整合来出发Intents

Flutter需要借助三方插件实现调用外部组件(相机,文件选择器)

###### Flutter中跳转

Flutter中切换屏幕,可以访问路由以绘制新的Widget,管理多个屏幕有俩个核心概念和类:Route和Navigator,Rounte是应用程序的屏幕或页面的抽象(Activity),Navigator是管理Rounte的WIdget,Navigator可以通过push和pop route以实现页面切换

类似于Android中在AndroidManifest.xml中声明Activities,在Flutter中,可以将具有指定Rounte的Map传递到顶层MaterialApp实例,但这不是必须的

```
    return MaterialApp(
      title: "List APP",
      theme: ThemeData(
          primarySwatch: Colors.blue
      ),
      home: ListModel(),
      routes: <String,WidgetBuilder>{
        '/a':(BuildContext context) => MyApp(title:'Page A'),
        '/b':(BuildContext context) => MyApp(title:'Page B'),
        '/c':(BuildContext context) => MyApp(title:'Page C'),
      },
    );
```

```
Navigator.of(context).pushNamed('/b');
```

```
Navigator.push(context,MaterialPageRoute(builder:(BuildContext context)=>UsualApp()));
```

###### 获取跳转返回结果

Navigator类不仅用来处理Flutter中的路由,还被用来获取刚push到栈中的路由返回的结果,通过await等待路由返回的结果实现

```
Map coordinates = await Navigator.of(context).pushNamed('/b');

//b页面设置返回结果
Navigator.of(context).pop({"lat":43.1234,"long",-34.2345});
```

###### 处理外部应用传入的Intents

使用MethodChannel,Activity接受到信息再传递到Flutter

###### 跳转其它App

创建一个原生平台的整合层,或者使用现有的plugin,例如url_launcher