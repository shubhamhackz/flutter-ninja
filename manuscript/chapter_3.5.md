## 3.5 Picture and ICON

## 3.5.1 Picture

In Flutter, we can `Image`load and display pictures through components, and `Image`the data sources can be assets, files, memory, and the network.

### ImageProvider

`ImageProvider`Is an abstract class that defines the interface main image data acquired `load()`acquiring images from different sources need to implement different data `ImageProvider`, such as `AssetImage`to achieve a load image from the Asset ImageProvider, whereas `NetworkImage`achieved ImageProvider Load picture from a network.

### Image

`Image`The widget has a required `image`parameter, which corresponds to an ImageProvider. Below we respectively demonstrate how to load images from asset and network.

#### Load image from asset

1.  Create one in the project root directory `images目录`and copy the picture avatar.png to this directory.
    
2.  In `pubspec.yaml`the `flutter`add the following parts:
    
    ```
      assets:
        - images/avatar.png
    
    ```
    
    > Note: Since the yaml file is strictly indented, it must be indented strictly in accordance with two spaces per layer. There should be two spaces in front of assets.
    
3.  Load the picture
    
    ```
    Image(
      image: AssetImage("images/avatar.png"),
      width: 100.0
    );
    
    ```
    
    Image also provides a shortcut constructor `Image.asset`for loading and displaying images from the asset:
    
    ```
    Image.asset("images/avatar.png",
      width: 100.0,
    )
    
    ```
    

#### Load pictures from the web

```
Image(
  image: NetworkImage(
      "https://avatars2.githubusercontent.com/u/20411648?s=460&v=4"),
  width: 100.0,
)

```

Image also provides a shortcut constructor `Image.network`for loading and displaying images from the network:

```
Image.network(
  "https://avatars2.githubusercontent.com/u/20411648?s=460&v=4",
  width: 100.0,
)

```

After running the above two examples, the picture is successfully loaded as shown in Figure 3-17:

![Figure 3-17](https://pcdn.flutterchina.club/imgs/3-17.png)

#### parameter

`Image`A series of parameters are defined when the picture is displayed, through which we can control the display appearance, size, and mixing effect of the picture. Let's take a look at the main parameters of Image:

```
const Image({
  ...
  this.width, //图片的宽
  this.height, //图片高度
  this.color, //图片的混合色值
  this.colorBlendMode, //混合模式
  this.fit,//缩放模式
  this.alignment = Alignment.center, //对齐方式
  this.repeat = ImageRepeat.noRepeat, //重复方式
  ...
})

```

-   `width`, : `height`For setting image width, height, width and height when not specified, according to the limitations of the current picture is the parent container, its original size as a display, provided only if `width`, `height`the one, the other attribute is by default by Scaling, but you can `fit`specify the adaptation rules through the properties described below .
    
-   `fit`: This attribute is used to specify the adaptation mode of the picture when the display space of the picture is different from the size of the picture itself. The adaptation mode is `BoxFit`defined in. It is an enumerated type with the following values:
    
    -   `fill`: It will stretch to fill the display space, the aspect ratio of the picture itself will change, and the picture will be deformed.
    -   `cover`: It will be enlarged according to the aspect ratio of the picture and then centered to fill the display space, the picture will not be deformed, and the part beyond the display space will be cropped.
    -   `contain`: This is the default adaptation rule of the picture. The picture will be scaled to fit the current display space while the aspect ratio of the picture itself remains unchanged, and the picture will not be deformed.
    -   `fitWidth`: The width of the picture will be scaled to the width of the display space, the height will be scaled proportionally, and then displayed in the center, the picture will not be deformed, and the part beyond the display space will be cropped.
    -   `fitHeight`: The height of the picture will be scaled to the height of the display space, the width will be scaled proportionally, and then displayed in the center, the picture will not be deformed, and the part beyond the display space will be cropped.
    -   `none`: The picture does not have an adaptation strategy and will be displayed in the display space. If the picture is larger than the display space, the display space will only display the middle part of the picture.
    
    A picture is worth a thousand words! We apply different `fit`values ​​to an avatar image with the same width and height , and the effect is shown in Figure 3-18:
    
    ![Figure 3-18](https://pcdn.flutterchina.club/imgs/3-18.png)
    

-   `color`和`colorBlendMode`: When drawing the picture, you can perform color mixing for each pixel, `color`specify the mixed color, and `colorBlendMode`specify the mixing mode. The following is a simple example:
    
    ```
    Image(
      image: AssetImage("images/avatar.png"),
      width: 100.0,
      color: Colors.blue,
      colorBlendMode: BlendMode.difference,
    );
    
    ```
    

The running effect is shown in Figure 3-19 (color):

![Figure 3-19](https://pcdn.flutterchina.club/imgs/3-19.png)

-   `repeat`: When the size of the picture itself is smaller than the display space, specify the repetition rule of the picture. A simple example is as follows:
    
    ```
    Image(
      image: AssetImage("images/avatar.png"),
      width: 100.0,
      height: 200.0,
      repeat: ImageRepeat.repeatY ,
    )
    
    ```
    
    The effect after running is shown in Figure 3-20:
    
    ![Figure 3-20](https://pcdn.flutterchina.club/imgs/3-20.png)
    

The complete sample code is as follows:

```
import 'package:flutter/material.dart';

class ImageAndIconRoute extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    var img=AssetImage("imgs/avatar.png");
    return SingleChildScrollView(
      child: Column(
        children: <Image>[
          Image(
            image: img,
            height: 50.0,
            width: 100.0,
            fit: BoxFit.fill,
          ),
          Image(
            image: img,
            height: 50,
            width: 50.0,
            fit: BoxFit.contain,
          ),
          Image(
            image: img,
            width: 100.0,
            height: 50.0,
            fit: BoxFit.cover,
          ),
          Image(
            image: img,
            width: 100.0,
            height: 50.0,
            fit: BoxFit.fitWidth,
          ),
          Image(
            image: img,
            width: 100.0,
            height: 50.0,
            fit: BoxFit.fitHeight,
          ),
          Image(
            image: img,
            width: 100.0,
            height: 50.0,
            fit: BoxFit.scaleDown,
          ),
          Image(
            image: img,
            height: 50.0,
            width: 100.0,
            fit: BoxFit.none,
          ),
          Image(
            image: img,
            width: 100.0,
            color: Colors.blue,
            colorBlendMode: BlendMode.difference,
            fit: BoxFit.fill,
          ),
          Image(
            image: img,
            width: 100.0,
            height: 200.0,
            repeat: ImageRepeat.repeatY ,
          )
        ].map((e){
          return Row(
            children: <Widget>[
              Padding(
                padding: EdgeInsets.all(16.0),
                child: SizedBox(
                  width: 100,
                  child: e,
                ),
              ),
              Text(e.fit.toString())
            ],
          );
        }).toList()
      ),
    );
  }
}

```

### Image cache

The Flutter framework has a cache (memory) for loaded pictures. The default maximum cache number is 1000, and the maximum cache space is 100M. We will introduce the details and principles of Image in depth later in the advanced part.

## 3.5.2 ICON

In Flutter, you can use iconfont like web development. Iconfont is "font icon". It makes the icon into a font file, and then displays different pictures by specifying different characters.

> In a font file, each character corresponds to a bit code, and each bit code corresponds to a display glyph. Different fonts refer to different glyphs, that is, the corresponding glyphs of the characters are different. In iconfont, only the font corresponding to the bit code is made into icons, so different characters will eventually be rendered into different icons.

In Flutter development, iconfont has the following advantages over pictures:

1.  Small size: the size of the installation package can be reduced.
2.  Vector: Iconfont are all vector icons, and zooming will not affect their clarity.
3.  You can apply text styles: you can change the color and size alignment of the font icons like text.
4.  It can be mixed with text through TextSpan.

##### Use Material Design font icons

Flutter includes a set of Material Design font icons by default `pubspec.yaml`. The configuration in the file is as follows

```
flutter:
  uses-material-design: true

```

All icons of Material Design can be viewed on its official website: [https://material.io/tools/icons/](https://material.io/tools/icons/)

Let's look at a simple example:

```
String icons = "";
// accessible: &#xE914; or 0xE914 or E914
icons += "\uE914";
// error: &#xE000; or 0xE000 or E000
icons += " \uE000";
// fingerprint: &#xE90D; or 0xE90D or E90D
icons += " \uE90D";

Text(icons,
  style: TextStyle(
      fontFamily: "MaterialIcons",
      fontSize: 24.0,
      color: Colors.green
  ),
);

```

The running effect is shown in Figure 3-21:

![Figure 3-21](https://pcdn.flutterchina.club/imgs/3-21.png)

From this example, we can see that using icons is like using text, but this method requires us to provide the code point of each icon, which is not friendly to developers. Therefore, Flutter encapsulates `IconData`and `Icon`specifically displays font icons. The example can also be implemented as follows:

```
Row(
  mainAxisAlignment: MainAxisAlignment.center,
  children: <Widget>[
    Icon(Icons.accessible,color: Colors.green,),
    Icon(Icons.error,color: Colors.green,),
    Icon(Icons.fingerprint,color: Colors.green,),
  ],
)

```

`Icons`The class contains all the `IconData`static variable definitions of the Material Design icon .

#### Use custom font icons

We can also use custom font icons. There are a lot of font icon materials on iconfont.cn, we can choose the icon we need to package and download, and some font files in different formats will be generated. In Flutter, we can use the ttf format.

Suppose we need to use a book icon and WeChat icon in our project, we package and download it and import it:

1.  Import the font icon file; this step is the same as importing the font file, assuming that our font icon file is saved in the project root directory, and the path is "fonts/iconfont.ttf":
    
    ```
    fonts:
      - family: myIcon  #指定一个字体名
        fonts:
          - asset: fonts/iconfont.ttf
    
    ```
    
2.  For ease of use, we define a `MyIcons`class with the `Icons`same function as the class: define all icons in the font file as static variables:
    
    ```
    class MyIcons{
      // book 图标
      static const IconData book = const IconData(
          0xe614, 
          fontFamily: 'myIcon', 
          matchTextDirection: true
      );
      // 微信图标
      static const IconData wechat = const IconData(
          0xec7d,  
          fontFamily: 'myIcon', 
          matchTextDirection: true
      );
    }
    
    ```
    
3.  use
    
    ```
    Row(
      mainAxisAlignment: MainAxisAlignment.center,
      children: <Widget>[
        Icon(MyIcons.book,color: Colors.purple,),
        Icon(MyIcons.wechat,color: Colors.green,),
      ],
    )
    
    ```
    
    The effect after running is shown in Figure 3-22:
    
    ![Figure 3-22](https://pcdn.flutterchina.club/imgs/3-22.png)