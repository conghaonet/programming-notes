## Flutter 学习笔记一
[TOC]
### Widgets
#### MaterialButton（带有水波纹效果的按钮）
* 带有水波纹效果的按钮
#### WillPopScope（监听返回按键）
监听返回按键
```dart
WillPopScope(
    // 点击返回键即触发该事件
    onWillPop: () async { 
        //监听到返回按键事件
        return true； //退出界面
        //不退出界面
        //return false;
    },
    child: _yourWidget,
)
```
#### GestureDetector（手势控制）
手势控制 onTap，onDoubleTap，onLongPress等事件监听
```dart
GestureDetector(
    onTap: () {    
        //处理点击事件
    },  
    child: YourWidget(),
)
```
#### InkWell（触摸时有水波纹效果，类似MaterialButton）
使控件在点击（触摸）时有水波纹效果，效果类似**MaterialButton**
```dart
InkWell(
    onTap: () {    
        //处理点击事件
    },  
    child: YourClickableWidget(),
)
```
#### SafeArea（在安全区域内显示UI）
在安全区域内显示UI，不会覆盖到上边状态栏和底部系统导航按键。
```dart 
SafeArea(
    child: YourWidget(),
)
```
#### Offstage（隐藏/显示widget）
隐藏/显示widget，类似Android中的View.Visible/View.Gone
```dart
Offstage(
    // offstage 为 true 时表示不渲染，也不占位，相当于 android 的 gone
    offstage: true,
    child: Text( "offstage=true时，该控件不渲染、不占位" ),
),

```
#### AnimatedOpacity（带动画效果的alpha值渐变）
带动画效果的alpha值渐变，类似淡入淡出效果，或者使用**Opacity**(无动画效果)用于隐藏某个widget。
**FadeTransition**也可以达到同样的效果，但是代码更复杂。
如果要淡入显示图片，建议使用**FadeInImage**。
```dart
AnimatedOpacity(  
    opacity: _homeImageVisible ? 1.0 : 0.0,  
    duration: Duration(milliseconds: 2000) ,  
    child: Text( "opacity=0.0时完全透明，可用于隐藏widget；opacity=1.0时不透明正常显示" ),
),
```
#### Placeholder（占位符控件）
占位符控件，主要在UI设计时使用，类似于原型设计中的线框图效果。
```dart
Placeholder(
    fallbackWidth: 300, //Placeholder默认填充满可用空间，但如果父级控件是横向滚动的无限宽度的控件，可以指定回调高度加以约束
    fallbackHeight: 400, //Placeholder默认填充满可用空间，但如果父级控件是ListView等无限高度的控件，可以指定回调高度加以约束
    color: Colors.red, // 线框图中间线条的颜色
    strokeWidth: 10, //线条宽度
)
```
![50074351388d282ad01cd5b90bcef7c4.png](en-resource://database/13043:1)

#### AbsorbPointer（禁止触摸的控件）
禁止触摸的小控件，类似于Android中的disable属性
```dart
AbsorbPointer(
  //absorbing 为 true 时，MyWidget 控件无法接收到触摸事件
  absorbing: true,  
  child: MyWidget(),
}
```

#### DropdownButton（下拉框）
```dart
//经验证，T的类型只能是String或int，为其他类型时报错。
DropdownButton<T>(
  value: _value, //当前选中的值，_value的类型为T
  hint: Text('——请选择——'),
  items: _items, //_items的类型为List<DropdownMenuItem<T>>
  onChanged: (_selected) { //_selected类型为T
    setState(() {    
      selected = _selected;  
    });
  },
)
```

#### IgnorePointer(忽略触摸事件，将事件向下传递)
忽略触摸事件，并向下传递。
```dart
IgnorePointer(
  ignoring: true,
  child: RaisedButton( //RaisedButton不接收点击事件，向下传递
    child: FlatButton(), //接收点击事件
  ),
)
```
默认情况下，在不使用IgnorePointer/AbsorbPointer的情况下，点击蓝色方块时，会向蓝色方块发送一个单击事件，而红色方块什么也得不到。在这种情况下，将蓝色方块用**AbsorbPointer**包裹，意味着当点击蓝色区域时，蓝色方块和红色方块都不会得到单击事件。
如果我们使用IgnorePointer，那么红色方块将在录制蓝色方块时接收单击事件。

#### Dismissible（侧滑删除控件）
侧滑删除控件， 多用于ListView中的item。
```dart
Dismissible(
  //如果Dismissible是一个列表项 它必须有一个key 用来区别其他项
  key: Key(_weather.cityInfo.citykey),
  //滑动时组件下一层显示的内容，没有设置secondaryBackground时，从右往左或者从左往右滑动都显示该内容，
  //设置了secondaryBackground后，从左往右滑动显示该内容，从右往左滑动显示secondaryBackground的内容
  background: Container(),
  //不能单独设置，只能在已经设置了background后才能设置，从右往左滑动时显示
  secondaryBackground：Container(),
  confirmDismiss: (direction) async {
    //返回Future<bool>, 为true时执行删除动作；false则恢复界面，取消删除；
  },
  onDismissed: (direction) {
    //侧滑动作的回调，直接将widget从界面删除，无返回值。
  },
  //侧滑时要从界面删除的widget
  child: ItemWidget(),
)
```

#### SimpleDialog（简单对话框）
对话框只显示title和按钮，布局相对简单，需要复杂的对话框时可使用AlertDialog。
* 该段代码片段可配合**Dismissible**的**confirmDismiss**函数使用。
```dart
  /// 弹出确认删除的对话框
  Future<bool> showConfirmDeleteDialog(SojsonWeather _weather) async {
    bool isConfirm = await showDialog(context: context, builder: (BuildContext context) {
      return SimpleDialog(
        title: Text('确认删除【${_weather.cityInfo.city}】?'),
        children: <Widget>[
          SimpleDialogOption(
            onPressed: () {
              Navigator.pop<bool>(context, false);
            },
            child: const Text('取消', style: TextStyle(color: Colors.black),),
          ),
          SimpleDialogOption(
            onPressed: () {
              Navigator.pop<bool>(context, true);
            },
            child: const Text('删除', style: TextStyle(color: Colors.black),),
          ),
        ],
      );
    });
    // isConfirm是bool类型，dart中bool可为null。isConfirm如为null，则返回false；
    // 点击对话框以外区域时，对话框会消失，此时isConfirm为null。
    return isConfirm ?? false;
  }
```

#### AlertDialog（弹出对话框）
弹出对话框
* [with the URL 
inline](https://book.flutterchina.club/chapter8/listener.html)

```dart
// 一般通过showDialog函数调用对话框；
// barrierDismissible为true时，点击对话框背景时可取消对话框，类似Android中的cancelable属性；
showDialog<void>(context: context, barrierDismissible: true, builder: (BuildContext context){
  return AlertDialog(
    //对话框标题
    title: Text("添加城市"),
    //如果MyWidget是通过FutureBuilder创建的对象，则必须要创建新的class并继承StatefulWidget才能刷新界面。
    content: MyWidget(),
    actions: <Widget>[
      FlatButton(
        child: Text('添加'),
        onPressed: () {
            //业务代码
        },
      ),
    ],
  );
}
```
#### AnimatedDefaultTextStyle（动画形式改变文字样式）
动画形式改变文字样式
**划重点**：AnimatedDefaultTextStyle中的**Text不能定义style**，否则动画不会生效（始终显示Text中定义的style）
```dart
child: AnimatedDefaultTextStyle(
  child: Text('可变字体的文字',), //Text不能定义style，否则动画无效。
  style: isShown 
      ? TextStyle(
          fontSize: 48,
          fontWeight: FontWeight.w900,
          color: Colors.green)
      : TextStyle(
          fontSize: 36,
          fontWeight: FontWeight.w200,
          color: Colors.blue,),
  duration: const Duration(seconds: 3),
),
```

#### FocusNode（用于widget焦点管理）
* [Focus and text field](https://flutter.dev/docs/cookbook/forms/focus)
```dart
class _MyCustomFormState extends State<MyCustomForm> {
  FocusNode myFocusNode;
  @override
  void initState() {
    super.initState();
    myFocusNode = FocusNode();
  }
  @override
  Widget build(BuildContext context) {
    return Column(
      children: [
        TextField(
          ... ...
          //autofocus: true,  //自动获取焦点，在一个页面中应该只有一个widget定义该属性为true。
          focusNode: myFocusNode
        ),
        FlatButton(
          ... ...
          //使上面的TextField获得焦点
          onPressed: () => FocusScope.of(context).requestFocus(myFocusNode),
        ),
      ]
    );
  }
  @override
  void dispose() {
    // 清除FocusNode
    myFocusNode.dispose();
    super.dispose();
  }l
}
```
#### Drawer（抽屉效果导航面板）
* [简书：Flutter 控件-Drawer使用](https://www.jianshu.com/p/70e0324c0204)

#### ClipOval（圆形剪裁）、ClipRRect（圆角矩形剪裁）、ClipRect（矩形剪裁）、ClipPath（路径剪裁）
* ClipOval（圆形剪裁）
```dart
// 圆形剪裁
ClipOval(
  child: SizedBox(
    width: 100.0,
    height:100.0,
    child:  Image.network("https://sfault-avatar.b0.upaiyun.com/206/120/2061206110-5afe2c9d40fa3_huge256",),
  ),
),
```

* ClipRRect（圆角矩形剪裁）
```dart
// 圆角矩形剪裁
ClipRRect(
    // borderRadius 参数用于控制圆角的位置大小
  borderRadius: BorderRadius.all(
    Radius.circular(10.0)),
    child:  new SizedBox(
      width: 100.0,
      height:100.0,
      child:  Image.network("https://sfault-avatar.b0.upaiyun.com/206/120/2061206110-5afe2c9d40fa3_huge256",),
  ),
)
```

* ClipRect（矩形剪裁）
```dart
/// 矩形剪裁
ClipRect(
  clipper: _MyClipper(),  //使用下面定义好的CustomClipper
  child: SizedBox(
    width: 100.0,
    height:100.0,
    child:  Image.network("https://sfault-avatar.b0.upaiyun.com/206/120/2061206110-5afe2c9d40fa3_huge256",),
  ) ,
),

/// ClipRect需要定义clipper参数才能使用，不然没有效果
class _MyClipper extends CustomClipper<Rect>{
  @override
  Rect getClip(Size size) {
    // 定义剪裁掉四周10像素的大小
    return Rect.fromLTRB(10.0, 10.0, size.width - 10.0,  size.height- 10.0);
  }
  @override
  bool shouldReclip(CustomClipper<Rect> oldClipper) {
    return true;
  }
}
```

* ClipPath（路径剪裁）

#### RefreshIndicator（下拉刷新组件）
```dart
RefreshIndicator({
  Key key,
  @required this.child,
  //触发下拉刷新的距离
  this.displacement: 40.0,
  //下拉回调方法,方法需要有async和await关键字，没有await，刷新图标立马消失，没有async，刷新图标不会消失
  @required this.onRefresh,
  //进度指示器前景色 默认为系统主题色
  this.color,
  //背景色
  this.backgroundColor,
  this.notificationPredicate: defaultScrollNotificationPredicate,
})
```
#### ExpansionPanelList和ExpansionPanel（可展开面板）
ExpansionPanel必须配合ExpansionPanelList一起使用。[参考链接](https://juejin.im/post/5d08a1d0f265da1bb27732ec)
```dart
ExpansionPanel({
  @required this.headerBuilder, //header
  @required this.body, //body 仅在展开时可见
  this.isExpanded = false, //是否展开
  this.canTapOnHeader = false, //header是否可以点击
}) : assert(headerBuilder != null),
assert(body != null),
assert(isExpanded != null),
assert(canTapOnHeader != null);

......

const ExpansionPanelList({
  Key key,
  this.children = const <ExpansionPanel>[],
  this.expansionCallback, //展开回调，这里会返回点击的 index
  this.animationDuration = kThemeAnimationDuration, //设置动画的时间
}) : assert(children != null),
assert(animationDuration != null),
_allowOnlyOnePanelOpen = false,
initialOpenPanelValue = null,
super(key: key);



```
