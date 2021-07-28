# 3.2 State management

Responsive programming frameworks will have an eternal theme-"State (State) management", whether it is in React/Vue (both are web development frameworks that support reactive programming) or Flutter, they discuss issues And the idea of ​​solution is consistent. So, if you have an understanding of React/Vue state management, you can skip this section. Closer to home, we would like to ask, `StatefulWidget`who should manage the state? Widget itself? The parent Widget? City? Or another object? The answer depends on the actual situation! The following are the most common ways to manage status:

-   Widget manages its own state.
-   Widget manages the state of sub-Widgets.
-   Hybrid management (parent Widget and child Widget both manage state).

How to decide which management method to use? The following are some of the official principles that can help you make a decision:

-   If the state is user data, such as the selected state of a check box and the position of a slider, the state is best managed by the parent Widget.
-   If the state is related to the appearance of the interface, such as color and animation, the state is best managed by the Widget itself.
-   If a certain state is shared by different Widgets, it is better to be managed by their common parent Widget.

It will be better to manage the state encapsulation inside the Widget, but it will be more flexible in the parent Widget. Sometimes, if you are not sure how to manage the state, then the recommended first choice is to manage it in the parent widget (flexibility is more important).

Next, we will illustrate the different ways of managing state by creating three simple examples TapboxA, TapboxB, and TapboxC. The functions of these examples are similar-create a box, and when you click on it, the background of the box will switch between green and gray. Status `_active`determination Color: green `true`, gray as `false`shown in Figure 3-4.![a large grey box with the text, 'Inactive'](https://pcdn.flutterchina.club/imgs/3-4.png)

The following example will be used `GestureDetector`to identify the click event. `GestureDetector`We will introduce the details in the chapter "Event Handling".

### 3.2.1 Widget manages its own state

_TapboxAState class:

-   Manage the status of TapboxA.
-   Definition `_active`: A Boolean value that determines the current color of the box.
-   Define a `_handleTap()`function, which updates when the box is clicked `_active`, and calls to `setState()`update the UI.
-   Implement all interactive behaviors of the widget.

``` dart 
// TapboxA 管理自身状态.

//------------------------- TapboxA ----------------------------------

class TapboxA extends StatefulWidget {
 TapboxA({Key key}) : super(key: key);

 @override
 _TapboxAState createState() => new _TapboxAState();
}

class _TapboxAState extends State<TapboxA> {
 bool _active = false;

 void _handleTap() {
   setState(() {
     _active = !_active;
   });
 }

 Widget build(BuildContext context) {
   return new GestureDetector(
     onTap: _handleTap,
     child: new Container(
       child: new Center(
         child: new Text(
           _active ? 'Active' : 'Inactive',
           style: new TextStyle(fontSize: 32.0, color: Colors.white),
         ),
       ),
       width: 200.0,
       height: 200.0,
       decoration: new BoxDecoration(
         color: _active ? Colors.lightGreen[700] : Colors.grey[600],
       ),
     ),
   );
 }
}

```

### 3.2.2 The parent widget manages the state of the child widget

For the parent Widget, it is usually a better way to manage the state and tell its child Widget when to update. For example, it `IconButton`is an icon button, but it is a stateless Widget, because we think that the parent Widget needs to know whether the button is clicked to take corresponding processing.

In the following example, TapboxB exports its state to its parent component through a callback. The state is managed by the parent component, so its parent component is `StatefulWidget`. But because TapboxB does not manage any state, it `TapboxB`is `StatelessWidget`.

`ParentWidgetState` class:

-   Manage `_active`status for TapboxB .
-   Implementation `_handleTapboxChanged()`, the method to be called when the box is clicked.
-   When the state changes, call the `setState()`update UI.

TapboxB class:

-   Inherit the `StatelessWidget`class, because all state is handled by its parent component.
-   When it detects a click, it notifies the parent component.

``` dart 
// ParentWidget 为 TapboxB 管理状态.

//------------------------ ParentWidget --------------------------------

class ParentWidget extends StatefulWidget {
 @override
 _ParentWidgetState createState() => new _ParentWidgetState();
}

class _ParentWidgetState extends State<ParentWidget> {
 bool _active = false;

 void _handleTapboxChanged(bool newValue) {
   setState(() {
     _active = newValue;
   });
 }

 @override
 Widget build(BuildContext context) {
   return new Container(
     child: new TapboxB(
       active: _active,
       onChanged: _handleTapboxChanged,
     ),
   );
 }
}

//------------------------- TapboxB ----------------------------------

class TapboxB extends StatelessWidget {
 TapboxB({Key key, this.active: false, @required this.onChanged})
     : super(key: key);

 final bool active;
 final ValueChanged<bool> onChanged;

 void _handleTap() {
   onChanged(!active);
 }

 Widget build(BuildContext context) {
   return new GestureDetector(
     onTap: _handleTap,
     child: new Container(
       child: new Center(
         child: new Text(
           active ? 'Active' : 'Inactive',
           style: new TextStyle(fontSize: 32.0, color: Colors.white),
         ),
       ),
       width: 200.0,
       height: 200.0,
       decoration: new BoxDecoration(
         color: active ? Colors.lightGreen[700] : Colors.grey[600],
       ),
     ),
   );
 }
}

```

### 3.2.3 Mixed state management

For some components, a mixed management approach can be very useful. In this case, the component itself manages some internal state, while the parent component manages some other external state.

In the TapboxC example below, when the finger is pressed, a dark green border will appear around the box, and when it is lifted, the border disappears. After clicking Finish, the color of the box changes. TapboxC exports its `_active`state to its parent component, but manages its `_highlight`state internally . This example has two state objects `_ParentWidgetState`and `_TapboxCState`.

`_ParentWidgetStateC`class:

-   Management `_active`status.
-   Implementation `_handleTapboxChanged()`, called when the box is clicked.
-   The update UI `_active`is called when the box is clicked and the state changes `setState()`.

`_TapboxCState` Object:

-   Management `_highlight`status.
-   `GestureDetector`Listen to all tap events. When the user clicks, it adds a highlight (dark green border); when the user releases it, it removes the highlight.
-   Update the `_highlight`state when pressing, lifting, or canceling the click, and call the `setState()`update UI.
-   When clicked, the state change is passed to the parent component.

``` dart 
//---------------------------- ParentWidget ----------------------------

class ParentWidgetC extends StatefulWidget {
 @override
 _ParentWidgetCState createState() => new _ParentWidgetCState();
}

class _ParentWidgetCState extends State<ParentWidgetC> {
 bool _active = false;

 void _handleTapboxChanged(bool newValue) {
   setState(() {
     _active = newValue;
   });
 }

 @override
 Widget build(BuildContext context) {
   return new Container(
     child: new TapboxC(
       active: _active,
       onChanged: _handleTapboxChanged,
     ),
   );
 }
}

//----------------------------- TapboxC ------------------------------

class TapboxC extends StatefulWidget {
 TapboxC({Key key, this.active: false, @required this.onChanged})
     : super(key: key);

 final bool active;
 final ValueChanged<bool> onChanged;

 @override
 _TapboxCState createState() => new _TapboxCState();
}

class _TapboxCState extends State<TapboxC> {
 bool _highlight = false;

 void _handleTapDown(TapDownDetails details) {
   setState(() {
     _highlight = true;
   });
 }

 void _handleTapUp(TapUpDetails details) {
   setState(() {
     _highlight = false;
   });
 }

 void _handleTapCancel() {
   setState(() {
     _highlight = false;
   });
 }

 void _handleTap() {
   widget.onChanged(!widget.active);
 }

 @override
 Widget build(BuildContext context) {
   // 在按下时添加绿色边框，当抬起时，取消高亮  
   return new GestureDetector(
     onTapDown: _handleTapDown, // 处理按下事件
     onTapUp: _handleTapUp, // 处理抬起事件
     onTap: _handleTap,
     onTapCancel: _handleTapCancel,
     child: new Container(
       child: new Center(
         child: new Text(widget.active ? 'Active' : 'Inactive',
             style: new TextStyle(fontSize: 32.0, color: Colors.white)),
       ),
       width: 200.0,
       height: 200.0,
       decoration: new BoxDecoration(
         color: widget.active ? Colors.lightGreen[700] : Colors.grey[600],
         border: _highlight
             ? new Border.all(
                 color: Colors.teal[700],
                 width: 10.0,
               )
             : null,
       ),
     ),
   );
 }
}

```

Another implementation may export the highlighted state to the parent component, but at the same time keep the `_active`state as an internal state, but if you want to use the TapBox for other people, it may not make much sense. Developers only care about whether the box is in Active state, not how the highlighting is managed, so tapBox should handle these details internally.

### 3.2.4 Global state management

When the application requires some cross-component (including cross-route) state needs to be synchronized, the method described above is difficult to do. For example, we have a settings page in which we can set the language of the application. In order to make the settings take effect in real time, we expect that when the language status changes, the components that depend on the application language in the APP can be rebuilt, but these components that depend on the application language It is not together with the settings page, so this situation is difficult to manage with the above method. At this time, the correct approach is to use a global state manager to handle the communication between components that are far apart. There are currently two main methods:

1.  Realize a global event bus, correspond the language state change to an event, and then `initState`subscribe to the language change event in the method of the application language-dependent component in the APP . When the user switches the language on the settings page, we publish a language change event, and the component that subscribes to this event will receive a notification. After receiving the notification, call the `setState(...)`method to restart `build`itself.
2.  Using some packages dedicated to state management, such as Provider and Redux, readers can view their detailed information on the pub.

This book will introduce the implementation principle and usage of the Provider package in the chapter "Functional Components". At the same time, it will also implement a global event bus in the chapter "Event Handling and Notification". Readers can read it directly if necessary.