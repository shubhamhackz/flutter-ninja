# 5.6 Scaffold, TabBar, bottom navigation

The Material component library provides a rich variety of components. This section introduces some commonly used components. The rest of the readers can check the documentation or the examples in the Material component part of the Flutter Gallery.

> Flutter Gallery is the Flutter Demo officially provided by Flutter. The source code is located in the examples directory of the Flutter source code. The author strongly recommends users to run the Flutter Gallery example. It is a very comprehensive Flutter sample application, a very good reference Demo, and the author Learn first-hand information about Flutter.

## 5.6.1 Scaffold

A complete routing page may include navigation bar, drawer menu (Drawer) and bottom Tab navigation menu, etc. If each routing page requires developers to implement these manually, it will be a very troublesome and boring thing. Fortunately, the Flutter Material component library provides some ready-made components to reduce our development tasks. `Scaffold`It is the skeleton of a routing page, we can use it to assemble a complete page easily.

### Example

We implement a page, which contains:

1.  A navigation bar
2.  There is a share button on the right side of the navigation bar
3.  There is a drawer menu
4.  Has a bottom navigation
5.  There is a floating action button in the lower right corner

The final effect is shown in Figure 5-18 and Figure 5-19:

![Figure 5-18](https://pcdn.flutterchina.club/imgs/5-18.png)  ![Figure 5-19](https://pcdn.flutterchina.club/imgs/5-19.png)

The implementation code is as follows:

``` dart 
class ScaffoldRoute extends StatefulWidget {
 @override
 _ScaffoldRouteState createState() => _ScaffoldRouteState();
}

class _ScaffoldRouteState extends State<ScaffoldRoute> {
 int _selectedIndex = 1;

 @override
 Widget build(BuildContext context) {
   return Scaffold(
     appBar: AppBar( //导航栏
       title: Text("App Name"), 
       actions: <Widget>[ //导航栏右侧菜单
         IconButton(icon: Icon(Icons.share), onPressed: () {}),
       ],
     ),
     drawer: new MyDrawer(), //抽屉
     bottomNavigationBar: BottomNavigationBar( // 底部导航
       items: <BottomNavigationBarItem>[
         BottomNavigationBarItem(icon: Icon(Icons.home), title: Text('Home')),
         BottomNavigationBarItem(icon: Icon(Icons.business), title: Text('Business')),
         BottomNavigationBarItem(icon: Icon(Icons.school), title: Text('School')),
       ],
       currentIndex: _selectedIndex,
       fixedColor: Colors.blue,
       onTap: _onItemTapped,
     ),
     floatingActionButton: FloatingActionButton( //悬浮按钮
         child: Icon(Icons.add),
         onPressed:_onAdd
     ),
   );
 }
 void _onItemTapped(int index) {
   setState(() {
     _selectedIndex = index;
   });
 }
 void _onAdd(){
 }
}

```

In the above code we used the following components:

| Component Name       | Explanation               |
| -------------------- | ------------------------- |
| AppBar               | A navigation bar skeleton |
| MyDrawer             | Drawer menu               |
| BottomNavigationBar  | BottomNavigationBar       |
| FloatingActionButton | Floating button           |


Let's introduce them separately.

## 5.6.2 AppBar

`AppBar`It is a Material style navigation bar, through which you can set the navigation bar title, navigation bar menu, and Tab title at the bottom of the navigation bar. Let's look at the definition of AppBar:

``` dart 
AppBar({
 Key key,
 this.leading, //导航栏最左侧Widget，常见为抽屉菜单按钮或返回按钮。
 this.automaticallyImplyLeading = true, //如果leading为null，是否自动实现默认的leading按钮
 this.title,// 页面标题
 this.actions, // 导航栏右侧菜单
 this.bottom, // 导航栏底部菜单，通常为Tab按钮组
 this.elevation = 4.0, // 导航栏阴影
 this.centerTitle, //标题是否居中 
 this.backgroundColor,
 ...   //其它属性见源码注释
})

```

If to `Scaffold`add a drawer menu, by default, `Scaffold`automatically `AppBar`the `leading`set menu button (as shown in the screenshot above), you can click on it to open the drawer menu. If we want to customize the menu icon, we can set it manually `leading`, such as:

``` dart 
Scaffold(
 appBar: AppBar(
   title: Text("App Name"),
   leading: Builder(builder: (context) {
     return IconButton(
       icon: Icon(Icons.dashboard, color: Colors.white), //自定义图标
       onPressed: () {
         // 打开抽屉菜单  
         Scaffold.of(context).openDrawer(); 
       },
     );
   }),
   ...  
 )

```

The code running effect is shown in Figure 5-20:

![Figure 5-20](https://pcdn.flutterchina.club/imgs/5-20.png)

You can see that the menu on the left has been replaced successfully.

The method to open the drawer menu in the code is `ScaffoldState`in, through which `Scaffold.of(context)`you can get `Scaffold`the `State`object of the nearest component of the parent .

### TabBar

Next, we add a Tab button group at the bottom of the navigation bar through the "bottom" attribute. The effect to be achieved is shown in Figure 5-21:

![Figure 5-21](https://pcdn.flutterchina.club/imgs/5-21.png)

A component is provided in the Material component library `TabBar`, which can quickly generate a `Tab`menu. The following is the source code corresponding to the above figure:

``` dart 
class _ScaffoldRouteState extends State<ScaffoldRoute>
   with SingleTickerProviderStateMixin {

 TabController _tabController; //需要定义一个Controller
 List tabs = ["新闻", "历史", "图片"];

 @override
 void initState() {
   super.initState();
   // 创建Controller  
   _tabController = TabController(length: tabs.length, vsync: this);
 }

 @override
 Widget build(BuildContext context) {
   return Scaffold(
     appBar: AppBar(
       ... //省略无关代码
       bottom: TabBar(   //生成Tab菜单
         controller: _tabController,
         tabs: tabs.map((e) => Tab(text: e)).toList()
       ),
     ),
     ... //省略无关代码

 }

```

The above code first creates one `TabController`, which is used to control/monitor `Tab`menu switching. Next, a bottom menu bar is generated through TabBar. `TabBar`The `tabs`attribute accepts a Widget array, which represents each Tab submenu. We can customize it or use the `Tab`component directly as in the example . It is a Material style provided by the Material component library. Tab menu.

`Tab`The component has three optional parameters. In addition to specifying the text, you can also specify the Tab menu icon, or directly customize the component style. `Tab`The components are defined as follows:

``` dart 
Tab({
 Key key,
 this.text, // 菜单文本
 this.icon, // 菜单图标
 this.child, // 自定义组件样式
})

```

Developers can customize according to actual needs.

### TabBarView

Through `TabBar`we can only generate a static menu, the real Tab page has not yet been implemented. Since `Tab`the switching of the menu and the Tab page needs to be synchronized, we need `TabController`to switch the Tab page by monitoring the switching of the Tab menu, the code is as follows:

``` dart 
_tabController.addListener((){  
 switch(_tabController.index){
   case 1: ...;
   case 2: ... ;   
 }
});

```

If our Tab page can be switched by sliding, we also need to update the offset of the TabBar indicator during the sliding process! Obviously, it is very troublesome to handle these manually. For this reason, the Material library provides a `TabBarView`component that not only can easily implement the Tab page, but also can easily cooperate with the TabBar to achieve synchronization switching and sliding state synchronization. Examples are as follows:

``` dart 
Scaffold(
 appBar: AppBar(
   ... //省略无关代码
   bottom: TabBar(
     controller: _tabController,
     tabs: tabs.map((e) => Tab(text: e)).toList()),
 ),
 drawer: new MyDrawer(),
 body: TabBarView(
   controller: _tabController,
   children: tabs.map((e) { //创建3个Tab页
     return Container(
       alignment: Alignment.center,
       child: Text(e, textScaleFactor: 5),
     );
   }).toList(),
 ),
 ... // 省略无关代码  
)

```

The effect after running is shown in Figure 5-22:

![Figure 5-22](https://pcdn.flutterchina.club/imgs/5-22.png)

Now, no matter if you click on the Tab menu in the navigation bar or swipe left and right on the page, the Tab page will switch, and the status of the Tab menu and the Tab page are always synchronized! How do they achieve synchronization? Attentive readers may have noticed, the above example `TabBar`and `TabBarView`the `controller`one and the same! This is the case, `TabBar`and `TabBarView`it is through the same one `controller`to realize the menu switching and sliding state synchronization. For `TabController`the detailed information, we will not introduce too much in this book. Readers can directly check the SDK when using it.

In addition, the Material Component Library also provides a `PageView`component, which `TabBarView`is similar to the function, and readers can learn about it by themselves.

## 5.6.3 Drawer

`Scaffold`The `drawer`and `endDrawer`attributes can each accept a Widget as the left and right drawer menus of the page. If the developer provides a drawer menu, the drawer menu can be opened when the user's finger slides in from the left (or right) side of the screen. The example at the beginning of this section implements a left drawer menu `MyDrawer`. Its source code is as follows:

``` dart 
class MyDrawer extends StatelessWidget {
 const MyDrawer({
   Key key,
 }) : super(key: key);

 @override
 Widget build(BuildContext context) {
   return Drawer(
     child: MediaQuery.removePadding(
       context: context,
       //移除抽屉菜单顶部默认留白
       removeTop: true,
       child: Column(
         crossAxisAlignment: CrossAxisAlignment.start,
         children: <Widget>[
           Padding(
             padding: const EdgeInsets.only(top: 38.0),
             child: Row(
               children: <Widget>[
                 Padding(
                   padding: const EdgeInsets.symmetric(horizontal: 16.0),
                   child: ClipOval(
                     child: Image.asset(
                       "imgs/avatar.png",
                       width: 80,
                     ),
                   ),
                 ),
                 Text(
                   "Wendux",
                   style: TextStyle(fontWeight: FontWeight.bold),
                 )
               ],
             ),
           ),
           Expanded(
             child: ListView(
               children: <Widget>[
                 ListTile(
                   leading: const Icon(Icons.add),
                   title: const Text('Add account'),
                 ),
                 ListTile(
                   leading: const Icon(Icons.settings),
                   title: const Text('Manage accounts'),
                 ),
               ],
             ),
           ),
         ],
       ),
     ),
   );
 }
}

```

The drawer menu usually uses `Drawer`components as the root node. It implements a Material-style menu panel that `MediaQuery.removePadding`can remove some of the default blanks of the Drawer (for example, the default top of the Drawer will leave a blank with the same height as the mobile phone status bar). Readers can try to pass different To see the actual effect. The drawer menu page is composed of top and bottom. The top is composed of user avatars and nicknames, and the bottom is a menu list, implemented by ListView. We will introduce ListView in detail in the section "Scrollable Components" later.

## 5.6.4 FloatingActionButton

`FloatingActionButton`It is a special button in the Material design specification, which is usually suspended in a certain position on the page as a shortcut entry for some common actions, such as the "➕" button in the lower right corner of the page in the example in this section. We can set one through `Scaffold`the `floatingActionButton`attribute `FloatingActionButton`, and at the same time `floatingActionButtonLocation`specify its floating position in the page through the attribute. This is relatively simple and will not be repeated.

## 5.6.5 Tab navigation bar at the bottom

We can use `Scaffold`the `bottomNavigationBar`properties to set the bottom navigation, as shown in the example at the beginning of this section, we use the Material component library `BottomNavigationBar`and `BottomNavigationBarItem`two components to implement the Material style bottom navigation bar. You can see that the above implementation code is very simple, so I won't repeat it, but what if we want to achieve the bottom navigation bar with the effect shown in Figure 5-23?

![Figure 5-23](https://pcdn.flutterchina.club/imgs/5-23.png)

A component is provided in the Material component library `BottomAppBar`, which can `FloatingActionButton`cooperate to achieve this "hole punching" effect. The source code is as follows:

``` dart 
bottomNavigationBar: BottomAppBar(
 color: Colors.white,
 shape: CircularNotchedRectangle(), // 底部导航栏打一个圆形的洞
 child: Row(
   children: [
     IconButton(icon: Icon(Icons.home)),
     SizedBox(), //中间位置空出
     IconButton(icon: Icon(Icons.business)),
   ],
   mainAxisAlignment: MainAxisAlignment.spaceAround, //均分底部导航栏横向空间
 ),
)

```

As you can see, there is no attribute that controls the position of the hole in the above code. In fact, the position of `FloatingActionButton`the hole depends on the position. The above `FloatingActionButton`position is:

``` dart 
floatingActionButtonLocation: FloatingActionButtonLocation.centerDocked,

```

So the hole position is in the middle of the bottom navigation bar.

`BottomAppBar`The `shape`attribute of determines the shape of the hole, `CircularNotchedRectangle`achieving a circular shape, and we can also customize the shape. For example, there is an example of a "diamond" shape in the Flutter Gallery example. Readers can check it out if they are interested.