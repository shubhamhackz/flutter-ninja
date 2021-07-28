# 3.7 Input box and form

Input box components `TextField`and form components are provided in the Material component library `Form`. Let's introduce them separately below.

## 3.7.1 TextField

`TextField`Used for text input, it provides many attributes. Let's briefly introduce the function of the main attributes, and then demonstrate the usage of key attributes through a few examples.

``` dart 
const TextField({
 ...
 TextEditingController controller, 
 FocusNode focusNode,
 InputDecoration decoration = const InputDecoration(),
 TextInputType keyboardType,
 TextInputAction textInputAction,
 TextStyle style,
 TextAlign textAlign = TextAlign.start,
 bool autofocus = false,
 bool obscureText = false,
 int maxLines = 1,
 int maxLength,
 bool maxLengthEnforced = true,
 ValueChanged<String> onChanged,
 VoidCallback onEditingComplete,
 ValueChanged<String> onSubmitted,
 List<TextInputFormatter> inputFormatters,
 bool enabled,
 this.cursorWidth = 2.0,
 this.cursorRadius,
 this.cursorColor,
 ...
})

```

-   `controller`: The controller of the edit box, through which you can set/get the content of the edit box, select edit content, and monitor edit text change events. In most cases, we need to explicitly provide one `controller`to interact with the text box. If not provided `controller`, one `TextField`will be automatically created internally.
   
-   `focusNode`: Used to control `TextField`whether to occupy the input focus of the current keyboard. It is a handle for us to interact with the keyboard.
   
-   `InputDecoration`: `TextField`Appearance display for control , such as prompt text, background color, border, etc.
   
-   `keyboardType`: Used to set the default keyboard input type of the input box, the values ​​are as follows:
   

TextInputType enumeration value

meaning

text

Text input keyboard

multiline

Multi-line text, need to be used in conjunction with maxLines (set to null or greater than 1)

number

Number; number keyboard will pop up

phone

Optimized phone number input keyboard; the numeric keyboard will pop up and display "* #"

datetime

Optimized date input keyboard; ": -" will be displayed on Android

emailAddress

The optimized email address; "@ ." will be displayed

url

Optimized url input keyboard; it will display "/."

-   `textInputAction`：Keyboard action button icon (ie, enter key icon), it is an enumerated value, there are multiple optional values, readers can view the API document for all the value list, the following is when the value `TextInputAction.search`is in the native Android system The keyboard style is shown in Figure 3-24:
   
   ![Figure 3-24](https://pcdn.flutterchina.club/imgs/3-24.png)
   
-   `style`: The text style being edited.
   
-   `textAlign`: The horizontal alignment of the edited text in the input box.
   
-   `autofocus`: Whether to automatically acquire focus.
   
-   `obscureText`: Whether to hide the text being edited, such as the scene used to enter the password, etc. The text content will be replaced with "•".
   
-   `maxLines`: The maximum number of lines in the input box, the default is 1; if it is `null`, there is no line limit.
   
-   `maxLength`And `maxLengthEnforced`: `maxLength`represents the maximum length of the text in the input box. After setting, the input text count will be displayed in the lower right corner of the input box. `maxLengthEnforced`When the decision to enter text longer than `maxLength`is blocking input, to `true`prevent the input, as `false`when the input but does not prevent the input box will turn red.
   
-   `onChange`: The callback function when the content of the input box changes; Note: The content change event can also be `controller`monitored through .
   
-   `onEditingComplete`And `onSubmitted`: These two callbacks are both triggered when the input in the input box is completed, such as pressing the completion key (check icon) or search key (🔍 icon) of the keyboard. The difference is that two different signatures callback, `onSubmitted`callback is the type that receives input current content as a parameter, but does not receive parameters.`ValueChanged<String>``onEditingComplete`
   
-   `inputFormatters`: Used to specify the input format; when the user input changes, it will be verified according to the specified format.
   
-   `enable`: If it is `false`, then the input block is disabled, and the disabled state does not receive input events displayed simultaneously disabled state pattern (in which `decoration`the definition).
   
-   `cursorWidth`, `cursorRadius`And `cursorColor`: These three attributes are used to customize the cursor width, rounded corners and color of the input box.
   

#### Example: Login input box

##### layout

``` dart 
Column(
       children: <Widget>[
         TextField(
           autofocus: true,
           decoration: InputDecoration(
               labelText: "用户名",
               hintText: "用户名或邮箱",
               prefixIcon: Icon(Icons.person)
           ),
         ),
         TextField(
           decoration: InputDecoration(
               labelText: "密码",
               hintText: "您的登录密码",
               prefixIcon: Icon(Icons.lock)
           ),
           obscureText: true,
         ),
       ],
);

```

After running, the effect is shown in Figure 3-25:

![Figure 3-25](https://pcdn.flutterchina.club/imgs/3-25.png)

##### Get input

There are two ways to get input:

1.  Define two variables to save the user name and password, and then `onChange`save the input content when triggered.
2.  Through `controller`direct access.

The first method is relatively simple, not as an example. Let’s focus on the second method. Let’s take the username input box as an example:

Define one `controller`:

``` dart 
//定义一个controller
TextEditingController _unameController = TextEditingController();

```

Then set the input box controller:

``` dart 
TextField(
   autofocus: true,
   controller: _unameController, //设置controller
   ...
)

```

Get the content of the input box through the controller

``` dart 
print(_unameController.text)

```

##### Monitor text changes

There are also two ways to monitor text changes:

1.  Set `onChange`callback, such as:
   
``` dart 
   TextField(
       autofocus: true,
       onChanged: (v) {
         print("onChange: $v");
       }
   )
   
```
   
2.  Through `controller`monitoring, such as:
   
``` dart 
   @override
   void initState() {
     //监听输入改变  
     _unameController.addListener((){
       print(_unameController.text);
     });
   }
   
```
   

Compared with the two methods, they `onChanged`are specifically used to monitor text changes, but `controller`have more functions. In addition to monitoring text changes, it can also set default values ​​and select text. Let's look at an example:

Create one `controller`:

``` dart 
TextEditingController _selectionController =  TextEditingController();

```

Set the default value, and select the following characters from the third character

``` dart 
_selectionController.text="hello world!";
_selectionController.selection=TextSelection(
   baseOffset: 2,
   extentOffset: _selectionController.text.length
);

```

Set `controlle`r:

``` dart 
TextField(
 controller: _selectionController,
)

```

The running effect is shown in Figure 3-26:

![Figure 3-26](https://pcdn.flutterchina.club/imgs/3-26.png)

##### Control focus

Can focus `FocusNode`and `FocusScopeNode`to control, by default, the focus `FocusScope`management, the focus control range it represents, in this range may be by `FocusScopeNode`moving the focus between the input block, and the like to set the default focus. We can use `FocusScope.of(context)`to get the default in the Widget tree `FocusScopeNode`. Let's look at an example. In this example, create two `TextField`, the first one automatically gets the focus, and then two buttons are created:

-   Click the first button to move the focus from the first `TextField`to the second `TextField`.
-   Click the second button to close the keyboard.

The effect we want to achieve is shown in Figure 3-27:

![Figure 3-27](https://pcdn.flutterchina.club/imgs/3-27.png)

code show as below:

``` dart 
class FocusTestRoute extends StatefulWidget {
 @override
 _FocusTestRouteState createState() => new _FocusTestRouteState();
}

class _FocusTestRouteState extends State<FocusTestRoute> {
 FocusNode focusNode1 = new FocusNode();
 FocusNode focusNode2 = new FocusNode();
 FocusScopeNode focusScopeNode;

 @override
 Widget build(BuildContext context) {
   return Padding(
     padding: EdgeInsets.all(16.0),
     child: Column(
       children: <Widget>[
         TextField(
           autofocus: true, 
           focusNode: focusNode1,//关联focusNode1
           decoration: InputDecoration(
               labelText: "input1"
           ),
         ),
         TextField(
           focusNode: focusNode2,//关联focusNode2
           decoration: InputDecoration(
               labelText: "input2"
           ),
         ),
         Builder(builder: (ctx) {
           return Column(
             children: <Widget>[
               RaisedButton(
                 child: Text("移动焦点"),
                 onPressed: () {
                   //将焦点从第一个TextField移到第二个TextField
                   // 这是一种写法 FocusScope.of(context).requestFocus(focusNode2);
                   // 这是第二种写法
                   if(null == focusScopeNode){
                     focusScopeNode = FocusScope.of(context);
                   }
                   focusScopeNode.requestFocus(focusNode2);
                 },
               ),
               RaisedButton(
                 child: Text("隐藏键盘"),
                 onPressed: () {
                   // 当所有编辑框都失去焦点时键盘就会收起  
                   focusNode1.unfocus();
                   focusNode2.unfocus();
                 },
               ),
             ],
           );
         },
         ),
       ],
     ),
   );
 }

}

```

`FocusNode`And `FocusScopeNode`there are some other methods, you can check the API documentation for details.

##### Monitor focus state change events

`FocusNode`Inherited from `ChangeNotifier`, `FocusNode`can monitor focus change events, such as:

``` dart 
...
// 创建 focusNode   
FocusNode focusNode = new FocusNode();
...
// focusNode绑定输入框   
TextField(focusNode: focusNode);
...
// 监听焦点变化    
focusNode.addListener((){
  print(focusNode.hasFocus);
});

```

The `focusNode.hasFocus`value is when the focus is obtained, and the value is when the focus is `true`lost `false`.

##### Custom style

Although we can `decoration`define the style of the input box through attributes, the following is an example of customizing the underline color of the input box:

``` dart 
TextField(
 decoration: InputDecoration(
   labelText: "请输入用户名",
   prefixIcon: Icon(Icons.person),
   // 未获得焦点下划线设为灰色
   enabledBorder: UnderlineInputBorder(
     borderSide: BorderSide(color: Colors.grey),
   ),
   //获得焦点下划线设为蓝色
   focusedBorder: UnderlineInputBorder(
     borderSide: BorderSide(color: Colors.blue),
   ),
 ),
),

```

In the above code, we directly use the enabledBorder and focusedBorder of InputDecoration to set the underline color of the input box after it has not been focused and after it has been focused. In addition, we can also customize the style of the input box through the theme. Let's explore how to customize the underline color without using enabledBorder and focusedBorder.

Since `TextField`the color used when drawing the underline is the theme color `hintColor`, but the prompt text color is also used `hintColor`, if we modify `hintColor`it directly , the color of the underline and prompt text will change. The good news is `decoration`that `hintStyle`it can be set , it can be overwritten `hintColor`, and the theme can be `inputDecorationTheme`used to set the default input box `decoration`. So we can customize by theme, the code is as follows:

``` dart 
Theme(
 data: Theme.of(context).copyWith(
     hintColor: Colors.grey[200], //定义下划线颜色
     inputDecorationTheme: InputDecorationTheme(
         labelStyle: TextStyle(color: Colors.grey),//定义label字体样式
         hintStyle: TextStyle(color: Colors.grey, fontSize: 14.0)//定义提示文本样式
     )
 ),
 child: Column(
   children: <Widget>[
     TextField(
       decoration: InputDecoration(
           labelText: "用户名",
           hintText: "用户名或邮箱",
           prefixIcon: Icon(Icons.person)
       ),
     ),
     TextField(
       decoration: InputDecoration(
           prefixIcon: Icon(Icons.lock),
           labelText: "密码",
           hintText: "您的登录密码",
           hintStyle: TextStyle(color: Colors.grey, fontSize: 13.0)
       ),
       obscureText: true,
     )
   ],
 )
)

```

The running effect is shown in Figure 3-28:

![Figure 3-28](https://pcdn.flutterchina.club/imgs/3-28.png)

We have successfully customized the underline color and question text style. Careful readers may have discovered that after customization in this way, the input box `labelText`will not be highlighted when it gets the focus , just like the "Username" in the picture above. "It was supposed to be blue, but it is now gray, and we still can’t define the underline width. Another flexible way is to hide `TextField`its own underline directly , and then `Container`define the style by de-nesting, such as:

``` dart 
Container(
 child: TextField(
   keyboardType: TextInputType.emailAddress,
   decoration: InputDecoration(
       labelText: "Email",
       hintText: "电子邮件地址",
       prefixIcon: Icon(Icons.email),
       border: InputBorder.none //隐藏下划线
   )
 ),
 decoration: BoxDecoration(
     // 下滑线浅灰色，宽度1像素
     border: Border(bottom: BorderSide(color: Colors.grey[200], width: 1.0))
 ),
)

```

running result:

![image-20180904150511545](https://cdn.jsdelivr.net/gh/flutterchina/flutter-in-action@1.0/docs/imgs/image-20180904150511545.png)

Through this combination of components, you can also define background rounded corners. Generally speaking, `decoration`customize the style first, if `decoration`it can’t be achieved, then use the widget combination method.

> Question: In this example, the color of the underline is fixed, so the color is still gray after the focus is obtained. How to realize that the underline also changes color after clicking?

## 3.7.2 Form

In actual business, before the data is formally submitted to the server, the validity of each input box data will be verified, but it `TextField`will be very troublesome to verify each one separately. Also, if the user wants to clear a group `TextField`of contents, is there any better way besides clearing them one by one? To this end, Flutter provides a `Form`component that can group input boxes, and then perform some unified operations, such as input content verification, input box reset, and input content preservation.

#### Form

`Form`Inherited from the `StatefulWidget`object, its corresponding state class is `FormState`. Let's first look at `Form`the definition of the class:

``` dart 
Form({
 @required Widget child,
 bool autovalidate = false,
 WillPopCallback onWillPop,
 VoidCallback onChanged,
})

```

-   `autovalidate`: Whether to automatically check the input content; when is `true`the time, each sub FormField content will automatically check the legality of changes, and directly displays an error message. Otherwise, it needs `FormState.validate()`to be manually verified by calling .
-   `onWillPop`: Determine `Form`whether the route you are in can return directly (such as clicking the return button). The callback returns an `Future`object. If the final result of the Future is `false`, the current route will not return; if it is `true`, it will return to the previous route. This attribute is usually used to intercept the return button.
-   `onChanged`: This callback is triggered when `Form`any of the sub- `FormField`content changes.

#### FormField

`Form`The descendant elements of must be a `FormField`type, `FormField`an abstract class, define several attributes, `FormState`internally use them to complete the operation, `FormField`part of the definition is as follows:

``` dart 
const FormField({
 ...
 FormFieldSetter<T> onSaved, //保存回调
 FormFieldValidator<T>  validator, //验证回调
 T initialValue, //初始值
 bool autovalidate = false, //是否自动校验。
})

```

For ease of use, Flutter provides a `TextFormField`component that inherits from the `FormField`class and is also `TextField`a wrapper class, so in addition to the `FormField`defined properties, it also includes `TextField`properties.

#### FormState

`FormState`For `Form`the `State`class, you can pass `Form.of()`or `GlobalKey`get. We can use it to perform unified operations on `Form`our descendants `FormField`. Let's look at the three commonly used methods:

-   `FormState.validate()`: After calling this method, the `Form`descendant `FormField的validate`callback will be called , if one of the verification fails, it will return false, and all the verification failed items will return the error message returned by the user.
-   `FormState.save()`: After calling this method, the callback of `Form`descendants will be called to save the form content`FormField``save`
-   `FormState.reset()`: After calling this method, `FormField`the content of descendants will be cleared.

#### Example

Let's modify the user login example above and verify before submitting:

1.  The user name cannot be empty. If it is empty, it will prompt "User name cannot be empty".
2.  The password cannot be less than 6 digits. If it is less than 6, it will prompt "The password cannot be less than 6 digits".

Complete code:

``` dart 
class FormTestRoute extends StatefulWidget {
 @override
 _FormTestRouteState createState() => new _FormTestRouteState();
}

class _FormTestRouteState extends State<FormTestRoute> {
 TextEditingController _unameController = new TextEditingController();
 TextEditingController _pwdController = new TextEditingController();
 GlobalKey _formKey= new GlobalKey<FormState>();

 @override
 Widget build(BuildContext context) {
   return Scaffold(
     appBar: AppBar(
       title:Text("Form Test"),
     ),
     body: Padding(
       padding: const EdgeInsets.symmetric(vertical: 16.0, horizontal: 24.0),
       child: Form(
         key: _formKey, //设置globalKey，用于后面获取FormState
         autovalidate: true, //开启自动校验
         child: Column(
           children: <Widget>[
             TextFormField(
                 autofocus: true,
                 controller: _unameController,
                 decoration: InputDecoration(
                     labelText: "用户名",
                     hintText: "用户名或邮箱",
                     icon: Icon(Icons.person)
                 ),
                 // 校验用户名
                 validator: (v) {
                   return v
                       .trim()
                       .length > 0 ? null : "用户名不能为空";
                 }

             ),
             TextFormField(
                 controller: _pwdController,
                 decoration: InputDecoration(
                     labelText: "密码",
                     hintText: "您的登录密码",
                     icon: Icon(Icons.lock)
                 ),
                 obscureText: true,
                 //校验密码
                 validator: (v) {
                   return v
                       .trim()
                       .length > 5 ? null : "密码不能少于6位";
                 }
             ),
             // 登录按钮
             Padding(
               padding: const EdgeInsets.only(top: 28.0),
               child: Row(
                 children: <Widget>[
                   Expanded(
                     child: RaisedButton(
                       padding: EdgeInsets.all(15.0),
                       child: Text("登录"),
                       color: Theme
                           .of(context)
                           .primaryColor,
                       textColor: Colors.white,
                       onPressed: () {
                         //在这里不能通过此方式获取FormState，context不对
                         //print(Form.of(context));

                         // 通过_formKey.currentState 获取FormState后，
                         // 调用validate()方法校验用户名密码是否合法，校验
                         // 通过后再提交数据。 
                         if((_formKey.currentState as FormState).validate()){
                           //验证通过提交数据
                         }
                       },
                     ),
                   ),
                 ],
               ),
             )
           ],
         ),
       ),
     ),
   );
 }
}

```

The effect after running is shown in Figure 3-29:

![Figure 3-29](https://pcdn.flutterchina.club/imgs/3-29.png)

Note that the login button `onPressed`method can not `Form.of(context)`be obtained, because, here `context`is `FormTestRoute`the context, which `Form.of(context)`is based on the specified `context`to find the root, while `FormState`in `FormTestRoute`sub-tree, it will not do. The correct approach is through `Builder`to build the login button `Builder`will `widget`node `context`as a callback parameter:

``` dart 
Expanded(
// 通过Builder来获取RaisedButton所在widget树的真正context(Element) 
 child:Builder(builder: (context){
   return RaisedButton(
     ...
     onPressed: () {
       //由于本widget也是Form的子代widget，所以可以通过下面方式获取FormState  
       if(Form.of(context).validate()){
         //验证通过提交数据
       }
     },
   );
 })
)

```

In fact, `context`it is `Element`an interface corresponding to the operation of Widget . Since the corresponding Widget tree `Element`is different, they `context`are all different `context`. More content will be discussed in detail in the advanced section later. There are many "of(context)" methods in Flutter, and readers must pay attention to `context`whether they are correct when using them .