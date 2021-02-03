## 3.3 Text and style

### 3.3.1 Text

`Text`Used to display simple style text, it contains some attributes that control the display style of the text. A simple example is as follows:

```
Text("Hello world",
  textAlign: TextAlign.left,
);

Text("Hello world! I'm Jack. "*4,
  maxLines: 1,
  overflow: TextOverflow.ellipsis,
);

Text("Hello world",
  textScaleFactor: 1.5,
);

```

The running effect is shown in Figure 3-5:

![image-20180829103242552](https://pcdn.flutterchina.club/imgs/3-5.png)

-   `textAlign`: The alignment of the text; you can choose left, right or center. Note that the frame of reference for alignment is the Text widget itself. Although the center alignment is specified in this example, because the width of the text content is less than one line, the width of the text is equal to the length of the text content, then it is meaningless to specify the alignment at this time, only specify this attribute when the width of the text is greater than the length of the text content It makes sense. Below we specify a longer string:
    
    ```
    Text("Hello world "*6,  //字符串重复六次
      textAlign: TextAlign.center,
    )；
    
    ```
    
    The running effect is shown in Figure 3-6:
    
    ![image-20180829104807535](https://pcdn.flutterchina.club/imgs/3-6.png)
    

​ If the string content exceeds one line, the Text width is equal to the screen width, and the second line of text will be displayed in the center.

-   `maxLines`, `overflow`: Specify the maximum number of lines for text display. By default, the text is automatically wrapped. If you specify this parameter, the text will not exceed the specified line at most. If there is extra text, you can `overflow`specify the truncation method. The default is direct truncation. The truncation method specified in this example `TextOverflow.ellipsis`will be truncated and then indicated by the ellipsis "..."; for other truncation methods of TextOverflow, please refer to SDK documentation.
-   `textScaleFactor`: Represents the zoom factor of the text relative to the current font size. Compared to setting the style `style`attribute of the text `fontSize`, it is a shortcut to adjust the font size. The default value of this attribute can be `MediaQueryData.textScaleFactor`obtained through , if not `MediaQuery`, then the default value will be 1.0.

### 3.3.2 TextStyle

`TextStyle`Used to specify the style of text display such as color, font, thickness, background, etc. Let's look at an example:

```
Text("Hello world",
  style: TextStyle(
    color: Colors.blue,
    fontSize: 18.0,
    height: 1.2,  
    fontFamily: "Courier",
    background: new Paint()..color=Colors.yellow,
    decoration:TextDecoration.underline,
    decorationStyle: TextDecorationStyle.dashed
  ),
);

```

The effect is shown in Figure 3-7:

![3-7](https://pcdn.flutterchina.club/imgs/3-7.png)

This example only shows part of the properties of TextStyle. It also has some other properties. The property names are basically self-explanatory. I won't repeat them here. Readers can refer to the SDK documentation. It is worth noting that:

-   `height`: This attribute is used to specify the row height, but it is not an absolute value, but a factor. The specific row height is equal to `fontSize`* `height`.
    
-   `fontFamily` : Because different platforms support different font sets by default, you must first test it on different platforms when manually specifying fonts.
    
-   `fontSize`: `textScaleFactor`Both this attribute and Text are used to control the font size. But there are two main differences:
    
    -   `fontSize`The font size can be specified precisely, but `textScaleFactor`can only be controlled by the zoom ratio.
    -   `textScaleFactor`It is mainly used for global adjustment of the Flutter application font when the system font size setting changes, but it `fontSize`is usually used for single text, and the font size will not follow the system font size change.

### 3.3.3 TextSpan

In the above example, all the text content of Text can only be in the same style. If we need to display different parts of a Text content in different styles, then we can use `TextSpan`it. It represents a "fragment" of text. Let's look at the definition of TextSpan:

```
const TextSpan({
  TextStyle style, 
  Sting text,
  List<TextSpan> children,
  GestureRecognizer recognizer,
});

```

Wherein `style`and `text`attribute represents the style and content of the text segments. `children`It is an `TextSpan`array, which means it `TextSpan`can include others `TextSpan`. It is `recognizer`used to recognize the gestures used on the text segment. Let's look at an effect (Figure 3-8), and then use `TextSpan`it to achieve it.

![3-8](https://pcdn.flutterchina.club/imgs/3-8.png)

Source code:

```
Text.rich(TextSpan(
    children: [
     TextSpan(
       text: "Home: "
     ),
     TextSpan(
       text: "https://flutterchina.club",
       style: TextStyle(
         color: Colors.blue
       ),  
       recognizer: _tapRecognizer
     ),
    ]
))

```

-   In the above code, we implemented a basic text fragment and a link fragment through TextSpan, and then added it to Text through the `Text.rich`method `TextSpan`. The reason why we can do this is because Text is actually a wrapper of RichText, and RichText can display a variety of Style (rich text) widget.
-   `_tapRecognizer`, It is a processor after clicking the link (code has been omitted), we will introduce more about gesture recognition later.

### 3.3.4 DefaultTextStyle

In the Widget tree, the style of the text can be inherited by default (the default style set by the parent in the Widget tree can be used when the subclass text component does not specify a specific style). Therefore, if you set it at a certain node of the Widget tree A default text style, then all text in the node's subtree will use this style by default, which `DefaultTextStyle`is used to set the default text style. Let's look at an example:

```
DefaultTextStyle(
  //1.设置文本默认样式  
  style: TextStyle(
    color:Colors.red,
    fontSize: 20.0,
  ),
  textAlign: TextAlign.start,
  child: Column(
    crossAxisAlignment: CrossAxisAlignment.start,
    children: <Widget>[
      Text("hello world"),
      Text("I am Jack"),
      Text("I am Jack",
        style: TextStyle(
          inherit: false, //2.不继承默认样式
          color: Colors.grey
        ),
      ),
    ],
  ),
);

```

In the above code, we first set a default text style, that is, the font is 20 pixels (logical pixels) and the color is red. Then it is `DefaultTextStyle`set to the Column node of the subtree, so that all descendants of Column will inherit this style by default, unless the Text display specifies that it does not inherit the style, as in Note 2 in the code. The example running effect is shown in Figure 3-9:

![3-9](https://pcdn.flutterchina.club/imgs/3-9.png)

### 3.3.5 Font

Different fonts can be used in Flutter applications. For example, we may use custom fonts created by designers, or other third-party fonts, such as fonts in [Google Fonts](https://fonts.google.com/) . This section will explain how to configure fonts for Flutter applications and use them when rendering text.

Using fonts in Flutter is done in two steps. `pubspec.yaml`Declare them in the first to ensure that they will be packaged into the application. Then [`TextStyle`](https://docs.flutter.io/flutter/painting/TextStyle-class.html)use the font through attributes.

#### Declared in asset

To pack the font file into the application, as with other resources, `pubspec.yaml`declare it in the first . Then copy the font file to `pubspec.yaml`the location specified in. Such as:

```
flutter:
  fonts:
    - family: Raleway
      fonts:
        - asset: assets/fonts/Raleway-Regular.ttf
        - asset: assets/fonts/Raleway-Medium.ttf
          weight: 500
        - asset: assets/fonts/Raleway-SemiBold.ttf
          weight: 600
    - family: AbrilFatface
      fonts:
        - asset: assets/fonts/abrilfatface/AbrilFatface-Regular.ttf

```

#### Use font

```
// 声明文本样式
const textStyle = const TextStyle(
  fontFamily: 'Raleway',
);

// 使用文本样式
var buttonText = const Text(
  "Use the font for this text",
  style: textStyle,
);

```

#### Fonts in the package

To use the font defined in the Package, **parameters** **must be provided`package`** . For example, suppose the font declaration above is in a `my_package`package. Then the process of creating TextStyle is as follows:

```
const textStyle = const TextStyle(
  fontFamily: 'Raleway',
  package: 'my_package', //指定包名
);

```

If you use its own defined font in the package, you should also specify the `package`parameters when creating the text style , as shown in the above example.

A package can also only provide font files without needing to be declared in pubspec.yaml. These files should be stored in the package `lib/`folder. Font files are not automatically bound to the application, and the application can selectively use these fonts when declaring fonts. Suppose there is a font file in a package named my_package:

```
lib/fonts/Raleway-Medium.ttf

```

Then, the application can declare a font, as shown in the following example:

```
 flutter:
   fonts:
     - family: Raleway
       fonts:
         - asset: assets/fonts/Raleway-Regular.ttf
         - asset: packages/my_package/fonts/Raleway-Medium.ttf
           weight: 500

```

`lib/`Is implicit, so it should not be included in the asset path.

In this case, since the application defines the font locally, it is not necessary to specify the `package`parameter when creating the TextStyle :

```
const textStyle = const TextStyle(
  fontFamily: 'Raleway',
);

```