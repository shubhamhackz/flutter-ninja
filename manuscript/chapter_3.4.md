# 3.4 Button

## 3.4.1 Buttons in Material Component Library

Material component library offers a variety of components such as buttons `RaisedButton`, `FlatButton`, `OutlineButton`etc., which are directly or indirectly on the `RawMaterialButton`packaging component customization, so most of their attributes and `RawMaterialButton`the same. When introducing each button, we first introduce its default appearance, and most of the appearance of the button can be customized through attributes. We will introduce these attributes in a unified manner later. In addition, all buttons in the Material library have the following similarities:

1.  When you press it, there will be "water wave animation" (also known as "ripple animation", which is an animation of water waves rippling on the button when you click it).
2.  There is an `onPressed`attribute to set the click callback, which will be executed when the button is pressed. If the callback is not provided, the button will be in a disabled state, and the disabled state will not respond to user clicks.

### RaisedButton

`RaisedButton`The "floating" button, which has a shadow and a gray background by default. After pressing, the shadow will become larger, as shown in Figure 3-10:

![Figure 3-10](https://pcdn.flutterchina.club/imgs/3-10.png)

It `RaisedButton`is very simple to use , such as:

```
RaisedButton(
  child: Text("normal"),
  onPressed: () {},
);

```

### FlatButton

`FlatButton`It is a flat button with a transparent background and no shadow by default. After pressing, there will be a background color, as shown in Figure 3-11:

![Figure 3-11](https://pcdn.flutterchina.club/imgs/3-11.png)

Using FlatButton is also very simple, the code is as follows:

```
FlatButton(
  child: Text("normal"),
  onPressed: () {},
)

```

### OutlineButton

`OutlineButton`There is a border by default, without shadow and transparent background. After pressing, the border color will become brighter, and the background and shadow (weaker) will appear at the same time, as shown in Figure 3-12:

![Figure 3-12](https://pcdn.flutterchina.club/imgs/3-12.png)

The use is `OutlineButton`also very simple, the code is as follows:

```
OutlineButton(
  child: Text("normal"),
  onPressed: () {},
)

```

### IconButton

`IconButton`It is a clickable Icon, does not include text, and there is no background by default. The background will appear after clicking, as shown in Figure 3-13:

![Figure 3-13](https://pcdn.flutterchina.club/imgs/3-13.png)

code show as below:

```
IconButton(
  icon: Icon(Icons.thumb_up),
  onPressed: () {},
)

```

### Button with icon

`RaisedButton`, `FlatButton`, `OutlineButton`Has a `icon`constructor, you can easily create a button with an icon through it, shown in Figure 3-14:

![Figure 3-14](https://pcdn.flutterchina.club/imgs/3-14.png)

code show as below:

```
RaisedButton.icon(
  icon: Icon(Icons.send),
  label: Text("发送"),
  onPressed: _onPressed,
),
OutlineButton.icon(
  icon: Icon(Icons.add),
  label: Text("添加"),
  onPressed: _onPressed,
),
FlatButton.icon(
  icon: Icon(Icons.info),
  label: Text("详情"),
  onPressed: _onPressed,
),

```

## 3.4.2 Custom button appearance

The appearance of the button can be defined by its properties. Different button properties are similar. Let's take FlatButton as an example to introduce the common button properties. For detailed information, please refer to the API documentation.

```
const FlatButton({
  ...  
  @required this.onPressed, //按钮点击回调
  this.textColor, //按钮文字颜色
  this.disabledTextColor, //按钮禁用时的文字颜色
  this.color, //按钮背景颜色
  this.disabledColor,//按钮禁用时的背景颜色
  this.highlightColor, //按钮按下时的背景颜色
  this.splashColor, //点击时，水波动画中水波的颜色
  this.colorBrightness,//按钮主题，默认是浅色主题 
  this.padding, //按钮的填充
  this.shape, //外形
  @required this.child, //按钮的内容
})

```

Most of the attribute names are self-explanatory, so we won't repeat them. Let's take an example to see how to customize buttons.

#### Example

Define a button with a blue background and rounded corners on both sides. The effect is shown in Figure 3-15:

![Figure 3-15](https://pcdn.flutterchina.club/imgs/3-15.png)

code show as below:

```
FlatButton(
  color: Colors.blue,
  highlightColor: Colors.blue[700],
  colorBrightness: Brightness.dark,
  splashColor: Colors.grey,
  child: Text("Submit"),
  shape:RoundedRectangleBorder(borderRadius: BorderRadius.circular(20.0)),
  onPressed: () {},
)

```

Very simple, in the above code, we mainly `shape`specify its shape as a rounded rectangle. Because the button background is blue (dark), we need to specify the button topic `colorBrightness`is `Brightness.dark`, which is to ensure the button text color is light.

Flutter does not provide a setting to remove the background. If we need to remove the background, we can achieve it by setting the background color to fully transparent. Corresponding to the above code, it is to be `color: Colors.blue`replaced `color: Color(0x000000)`.

Observant readers may find that this button has no shadow (and after clicking it), it will appear to be untextured. In fact, this is also easy to top `FlatButton`be replaced `RaisedButton`on the line, do not change the other code (here color do not change), after the effect of the change shown in Figure 3-16:

![Figure 3-16](https://pcdn.flutterchina.club/imgs/3-16.png)

Is it textured? The reason for this is that `RaisedButton`there is a shadow configuration by default:

```
const RaisedButton({
  ...
  this.elevation = 2.0, //正常状态下的阴影
  this.highlightElevation = 8.0,//按下时的阴影
  this.disabledElevation = 0.0,// 禁用时的阴影
  ...
}

```

It is worth noting that in the Material component library, we will see elevation-related attributes in many components. They are used to control shadows. This is because shadows are an important form of expression in the Material design style. , I won’t repeat them when introducing other components in the future.

If we want to implement a rounded button with a gradient background, do the buttons have corresponding attributes? The answer is no, but we can do it in other ways, we will implement it in the "Custom Components" chapter later.