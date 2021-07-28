# 2.2 Route management

Route usually refers to page in mobile development, which is the same as the conceptual meaning of Route in single-page applications in web development. Route usually refers to an Activity in Android and a ViewController in iOS. The so-called routing management is to manage how to jump between pages, which can also be called navigation management. Routing management in Flutter is similar to native development. Whether it’s Android or iOS, navigation management will maintain a routing stack. The routing push operation corresponds to opening a new page, and the routing pop operation corresponds to the page closing operation. Routing management mainly refers to how to manage the routing stack.

## 2.2.1 A simple example

Based on the "Counter" example in the previous section, we make the following modifications:

1.  Create a new route and name it "NewRoute"
   
``` dart 
   class NewRoute extends StatelessWidget {
     @override
     Widget build(BuildContext context) {
       return Scaffold(
         appBar: AppBar(
           title: Text("New route"),
         ),
         body: Center(
           child: Text("This is new route"),
         ),
       );
     }
   }
   
```
   
   The new route is inherited from `StatelessWidget`, and the interface is very simple, with a sentence "This is new route" displayed in the middle of the page.
   
2.  Add a button ( ) to the child widget in the `_MyHomePageState.build`method :`Column``FlatButton`
   
``` dart 
   Column(
         mainAxisAlignment: MainAxisAlignment.center,
         children: <Widget>[
         ... //省略无关代码
         FlatButton(
            child: Text("open new route"),
            textColor: Colors.blue,
            onPressed: () {
             //导航到新路由   
             Navigator.push( context,
              MaterialPageRoute(builder: (context) {
                 return NewRoute();
              }));
             },
            ),
          ],
    )
   
```
   
   We added a button to open a new route and set the button text color to blue. After clicking the button, the new route page will be opened. The effect is shown in Figures 2-2 and 2-3.
   
   ![Figure 2-2](https://pcdn.flutterchina.club/imgs/2-2.png)  ![Figure 2-3](https://pcdn.flutterchina.club/imgs/2-3.png)
   

## 2.2.2 MaterialPageRoute

`MaterialPageRoute`Inherited from the `PageRoute`class, the `PageRoute`class is an abstract class that represents a modal routing page that occupies the entire screen space. It also defines the related interfaces and attributes of the transition animation during routing construction and switching. `MaterialPageRoute`It is a component provided by the Material Component Library. It can implement routing switching animations that are consistent with the platform page switching animation style for different platforms:

-   For Android, when a new page is opened, the new page will slide from the bottom of the screen to the top of the screen; when the page is closed, the current page will slide from the top of the screen to the bottom of the screen and then disappear, and the previous page will be displayed on the screen.
-   For iOS, when the page is opened, the new page will slide from the right edge of the screen to the left side of the screen until the new page is displayed on the screen, and the previous page will slide from the current screen to the left side of the screen and disappear; When you close the page, the opposite is true. The current page will slide out from the right side of the screen, while the previous page will slide in from the left side of the screen.

Below we introduce `MaterialPageRoute`the meaning of each parameter of the constructor:

``` dart 
 MaterialPageRoute({
   WidgetBuilder builder,
   RouteSettings settings,
   bool maintainState = true,
   bool fullscreenDialog = false,
 })

```

-   `builder`It is a callback function of WidgetBuilder type. Its function is to build the specific content of the routing page, and the return value is a widget. We usually implement this callback and return an instance of the new route.
-   `settings` Contains routing configuration information, such as the name of the route and whether it is an initial route (home page).
-   `maintainState`: By default, when a new route is stacked, the original route will still be stored in the memory. If you want to release all the resources occupied by the route when it is useless, you can set it `maintainState`to false.
-   `fullscreenDialog`Indicates whether the new routing page is a full-screen modal dialog box. In iOS, if it `fullscreenDialog`is `true`, the new page will slide in from the bottom of the screen (instead of horizontally).

> If you want to customize the routing switching animation, you can inherit PageRoute to implement it yourself. We will implement a custom routing component when we introduce the animation later.

## 2.2.3 Navigator

`Navigator`Is a routing management component, it provides methods to open and exit the routing page. `Navigator`A stack is used to manage the collection of active routes. Usually the page displayed on the current screen is the route at the top of the stack. `Navigator`A series of methods are provided to manage the routing stack. Here we only introduce the two most commonly used methods:

### Future push(BuildContext context, Route route)

Push the given route into the stack (that is, open a new page), and the return value is an `Future`object to receive the return data when the new route is out of the stack (that is, close).

### bool pop(BuildContext context, [ result ])

The top of the stack is routed out of the stack, which `result`is the data returned to the previous page when the page is closed.

`Navigator`There are many other methods, such as `Navigator.replace`, `Navigator.popUntil`etc. For details, please refer to the API documentation or SDK source code comments, so I won’t repeat them here. Below we also need to introduce another concept "named routing" related to routing.

### Instance method

The **static method** whose first parameter is context in the Navigator class corresponds to an **instance method of** Navigator. For **example** , it is `Navigator.push(BuildContext context, Route route)`equivalent to `Navigator.of(context).push(Route route)`, and the methods related to the named route below are also the same.

## 2.2.4 Routing Value

In many cases, we need to bring some parameters when routing jumps. For example, when opening the product detail page, we need to bring a product id so that the product detail page knows which product information to display; for example, we need to choose to receive the goods when filling in the order Address, after opening the address selection page and selecting the address, the address selected by the user can be returned to the order page and so on. Below we use a simple example to demonstrate how the new and old routes pass parameters.

### Example

We create a `TipRoute`route, which accepts a prompt text parameter and is responsible for displaying the text passed into it on the page. In addition `TipRoute`, we add a "return" button. After clicking it, it will return to the previous route with a return parameter. Let's take a look at the implementation code.

`TipRoute`Implementation code:

``` dart 
class TipRoute extends StatelessWidget {
 TipRoute({
   Key key,
   @required this.text,  // 接收一个text参数
 }) : super(key: key);
 final String text;

 @override
 Widget build(BuildContext context) {
   return Scaffold(
     appBar: AppBar(
       title: Text("提示"),
     ),
     body: Padding(
       padding: EdgeInsets.all(18),
       child: Center(
         child: Column(
           children: <Widget>[
             Text(text),
             RaisedButton(
               onPressed: () => Navigator.pop(context, "我是返回值"),
               child: Text("返回"),
             )
           ],
         ),
       ),
     ),
   );
 }
}

```

Here is `TipRoute`the code to open the new route :

``` dart 
class RouterTestRoute extends StatelessWidget {
 @override
 Widget build(BuildContext context) {
   return Center(
     child: RaisedButton(
       onPressed: () async {
         // 打开`TipRoute`，并等待返回结果
         var result = await Navigator.push(
           context,
           MaterialPageRoute(
             builder: (context) {
               return TipRoute(
                 // 路由参数
                 text: "我是提示xxxx",
               );
             },
           ),
         );
         //输出`TipRoute`路由返回结果
         print("路由返回值: $result");
       },
       child: Text("打开提示页"),
     ),
   );
 }
}

```

Run the above code, click `RouterTestRoute`the "Open Prompt Page" button on the `TipRoute`page, the page will be opened , and the running effect is shown in Figure 2-4:

![Figure 2-4](https://pcdn.flutterchina.club/imgs/2-4.png)

Need to explain:

1.  The prompt text "I am prompt xxxx" is passed to the new routing page through `TipRoute`the `text`parameters. We can get the return data of the new route by waiting for the `Navigator.push(…)`return `Future`.
   
2.  `TipRoute`There are two ways to return to the previous page in the page; the first way is to directly click the return arrow in the navigation bar, and the second way is to click the "back" button on the page. The difference between these two return methods is that the former will not return data to the previous route, while the latter will. The following is the output content `RouterTestRoute`of the `print`method in the page in the console after clicking the return button and the return arrow in the navigation bar respectively :
   
``` dart 
   I/flutter (27896): 路由返回值: 我是返回值
   I/flutter (27896): 路由返回值: null
   
```
   

The value passing method of the non-named route is introduced above. The value passing method of the named route will be different. We will introduce it when we introduce the named route below.

## 2.2.5 Named Route

The so-called "Named Route" is a named route. We can give the route a name first, and then directly open the new route through the route name. This brings an intuitive and simple way to route management. the way.

### Routing table

To use named routing, we must first provide and register a routing table (routing table) so that the application knows which name corresponds to which routing component. In fact, registering the routing table is to name the route. The definition of the routing table is as follows:

``` dart 
Map<String, WidgetBuilder> routes;

```

It is one `Map`, the key is the name of the route, it is a string; the value is a `builder`callback function, used to generate the corresponding route widget. When we open a new route through the route name, the application will find the corresponding `WidgetBuilder`callback function in the routing table according to the route name , and then call the callback function to generate a route widget and return.

### Register routing table

The registration method of the routing table is very simple. Let's go back to the previous "counter" example, and then find it in `MyApp`the `build`method of the class `MaterialApp`and add `routes`attributes. The code is as follows:

``` dart 
MaterialApp(
 title: 'Flutter Demo',
 theme: ThemeData(
   primarySwatch: Colors.blue,
 ),
 //注册路由表
 routes:{
  "new_page":(context) => NewRoute(),
   ... // 省略其它路由注册信息
 } ,
 home: MyHomePage(title: 'Flutter Demo Home Page'),
);

```

Now we have completed the registration of the routing table. The `home`routing in the code above does not use named routes. What if we want to `home`register as named routes? It's actually very simple, just look at the code:

``` dart 
MaterialApp(
 title: 'Flutter Demo',
 initialRoute:"/", //名为"/"的路由作为应用的home(首页)
 theme: ThemeData(
   primarySwatch: Colors.blue,
 ),
 //注册路由表
 routes:{
  "new_page":(context) => NewRoute(),
  "/":(context) => MyHomePage(title: 'Flutter Demo Home Page'), //注册首页路由
 } 
);

```

As you can see, we only need to register a `MyHomePage`route in the routing table and use its name as `MaterialApp`the `initialRoute`attribute value. This attribute determines which named route is the initial routing page of the application.

### Open the new route page by route name

To open a new route by route name, you can use `Navigator`the `pushNamed`method:

``` dart 
Future pushNamed(BuildContext context, String routeName,{Object arguments})

```

`Navigator`In addition to `pushNamed`methods, there are `pushReplacementNamed`other methods for managing named routes. Readers can check the API documentation by themselves. Next, we open a new routing page through the routing name, and modify `FlatButton`the `onPressed`callback code to:

``` dart 
onPressed: () {
 Navigator.pushNamed(context, "new_page");
 //Navigator.push(context,
 //  MaterialPageRoute(builder: (context) {
 //  return NewRoute();
 //}));  
},

```

For hot reloading applications, click the "open new route" button again to open a new route page.

### Named route parameter passing

In the original version of Flutter, named routes could not pass parameters, and parameters were later supported; the following shows how named routes can pass and get route parameters:

We first register a route:

``` dart 
routes:{
  "new_page":(context) => EchoRoute(),
 } ,

```

`RouteSetting`Obtain routing parameters through objects on the routing page :

``` dart 
class EchoRoute extends StatelessWidget {

 @override
 Widget build(BuildContext context) {
   //获取路由参数  
   var args=ModalRoute.of(context).settings.arguments;
   //...省略无关代码
 }
}

```

Pass parameters when opening the route

``` dart 
Navigator.of(context).pushNamed("new_page", arguments: "hi");

```

### adaptation

Suppose we also want to `TipRoute`register the routing page in the routing parameter example above to the routing table so that it can also be opened by the routing name. However, since it `TipRoute`accepts a `text`parameter, how can we `TipRoute`adapt to this situation without changing the source code? it's actually really easy:

``` dart 
MaterialApp(
 ... //省略无关代码
 routes: {
  "tip2": (context){
    return TipRoute(text: ModalRoute.of(context).settings.arguments);
  },
}, 
);

```

## 2.2.6 Route generation hook

Suppose we want to develop an e-commerce app. When users are not logged in, they can view information such as shops and products, but pages such as transaction records, shopping carts, and user personal information can only be viewed after logging in. In order to achieve the above functions, we need to determine the user login status before opening each routing page! If we need to judge every time before opening the router, it will be very troublesome, is there any better way? The answer is yes!

`MaterialApp`There is an `onGenerateRoute`attribute, it may be called when the named route is opened. The reason why it is possible is because when the `Navigator.pushNamed(...)`open named route is called , if the specified route name is registered in the routing table, the `builder`function in the routing table will be called . Generate routing components; if there is no registration in the routing table, it will be called `onGenerateRoute`to generate routing. `onGenerateRoute`The callback signature is as follows:

``` dart 
Route<dynamic> Function(RouteSettings settings)

```

With `onGenerateRoute`callbacks, it is very easy to implement the above function of controlling page permissions: we give up using the routing table, and instead provide a `onGenerateRoute`callback, and then perform unified permission control in the callback, such as:

``` dart 
MaterialApp(
 ... //省略无关代码
 onGenerateRoute:(RouteSettings settings){
     return MaterialPageRoute(builder: (context){
          String routeName = settings.name;
      // 如果访问的路由页需要登录，但当前未登录，则直接返回登录页路由，
      // 引导用户登录；其它情况则正常打开路由。
    }
  );
 }
);

```

> Note that it `onGenerateRoute`will only take effect for named routes.

## 2.2.7 Summary

This chapter first introduces the routing management and parameter transfer methods in Flutter, and then focuses on the content of named routing. One point needs to be explained here, because named routing is only an optional routing management method, in actual development, readers may hesitate in mind which routing management method to use. Here, based on the author’s experience, it is recommended that readers use the named route management method uniformly, which will bring the following benefits:

1.  Semanticization is clearer.
2.  The code is better to maintain; if you use anonymous routing, you must `Navigator.push`create a new routing page where you call it . This not only requires importing the dart file of the new routing page, but the code will be very scattered.
3.  You can `onGenerateRoute`do some global routing jump pre-processing logic.

In summary, the author recommends the use of named routing, of course, this is not a golden rule, readers can decide according to their own preferences or actual situation.

In addition, there are some details about the route of administration we have not introduced, such as routing MaterialApp there `navigatorObservers`and `onUnknownRoute`two callback properties, the former can listen to all the action jumps the route, which will be called when opening a route name that does not exist, Since these functions are not commonly used and are relatively simple, we no longer spend space on introducing them, and readers can check the API documentation by themselves.