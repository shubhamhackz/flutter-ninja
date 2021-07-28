# 2.4 Resource Management

The Flutter APP installation package will contain two parts: code and assets. Assets are packaged in the program installation package and can be accessed at runtime. Common types of assets include static data (such as JSON files), configuration files, icons and pictures (JPEG, WebP, GIF, animated WebP/GIF, PNG, BMP and WBMP), etc.

## Specify assets

Like package management, Flutter also uses [`pubspec.yaml`](https://www.dartlang.org/tools/pub/pubspec)files to manage the resources required by the application, for example:

``` dart 
flutter:
 assets:
   - assets/my_icon.png
   - assets/background.png

```

`assets`Specify the files that should be included in the application. Each asset `pubspec.yaml`identifies its path relative to the file system path where the file is located. The order of asset declaration is irrelevant. The actual directory of the asset can be any folder (in this example, the asset folder).

During the build, Flutter places assets in special archives called _asset bundles_ , and the application can read them (but not modify them) at runtime.

## Asset variant (variant)

The build process supports the concept of "asset variants": different versions of assets may be displayed in different contexts. When `pubspec.yaml`the asset path is specified in the assets section, during the build process, any file with the same name will be searched in adjacent subdirectories. These files will then be included in the asset bundle along with the specified asset.

For example, if the following files are in the application directory:

-   … / Pubspec.yaml
-   …/graphics/my_icon.png
-   …/graphics/background.png
-   …/graphics/dark/background.png
-   …etc.

Then the `pubspec.yaml`file only needs to include:

``` dart 
flutter:
 assets:
   - graphics/background.png

```

Then these two `graphics/background.png`and `graphics/dark/background.png`will be included in your asset bundle. The former is considered a _main asset_ , and the latter is considered a variant.

When selecting images that match the current device resolution, Flutter will use asset variants (see below). In the future, Flutter may extend this mechanism to localization, reading prompts, etc.

## Load assets

Your app can [`AssetBundle`](https://docs.flutter.io/flutter/services/AssetBundle-class.html)access its assets through objects. There are two main ways to allow loading of strings or image (binary) files from the Asset bundle.

### Load text assets

-   [`rootBundle`](https://docs.flutter.io/flutter/services/rootBundle.html)Load via object: Every Flutter application has an [`rootBundle`](https://docs.flutter.io/flutter/services/rootBundle.html)object, through which you can easily access the main resource package, and directly use `package:flutter/services.dart`the global static `rootBundle`object to load the asset.
-   By [`DefaultAssetBundle`](https://docs.flutter.io/flutter/widgets/DefaultAssetBundle-class.html)loading: It is recommended [`DefaultAssetBundle`](https://docs.flutter.io/flutter/widgets/DefaultAssetBundle-class.html)to get the current BuildContext of AssetBundle. This method is not to use the default asset bundle built by the application, but to make the parent widget dynamically replace a different AssetBundle at runtime, which is useful for localization or testing scenarios.

Generally, you can `DefaultAssetBundle.of()`load assets (such as JSON files) indirectly while the application is running, and `AssetBundle`you can `rootBundle`load these assets directly outside of the widget context or when other handles are not available , for example:

``` dart 
import 'dart:async' show Future;
import 'package:flutter/services.dart' show rootBundle;

Future<String> loadAsset() async {
 return await rootBundle.loadString('assets/config.json');
}

```

### Load picture

Similar to native development, Flutter can also load images suitable for the current device's resolution.

#### Declare resolution-related image assets

[`AssetImage`](https://docs.flutter.io/flutter/painting/AssetImage-class.html)The asset request logic can be mapped to the asset closest to the current device pixel ratio (dpi). In order for this mapping to work, assets must be saved according to a specific directory structure:

-   …/image.png
-   …/**M**x/image.png
-   …/**N**x/image.png
-   …etc.

Where M and N are numeric identifiers, corresponding to the resolution of the image contained therein, that is, they specify pictures of different device pixel ratios.

The main resource corresponds to 1.0 times the resolution picture by default. Look at an example:

-   …/my_icon.png
-   …/2.0x/my_icon.png
-   …/3.0x/my_icon.png

It `.../2.0x/my_icon.png`will be selected on a device with a device pixel ratio of 1.8 . For a device pixel ratio of 2.7, it `.../3.0x/my_icon.png`will be selected.

If `Image`the width and height of the rendered image are not specified on the widget, the `Image`widget will occupy the same screen space as the main resource. In other words, if it `.../my_icon.png`is 72px by 72px, then it `.../3.0x/my_icon.png`should be 216px by 216px; but if the width and height are not specified, they will all be rendered as 72 pixels × 72 pixels (in logical pixels).

`pubspec.yaml`Each item in the asset section should correspond to the actual file, except for the main resource item. When the main resource lacks a certain resource, it will be selected in the order of resolution from low to high, that is to say, if it is not in 1x, it will be found in 2x, and if it is not in 2x, it will be found in 3x.

#### Load picture

To load pictures, you can use [`AssetImage`](https://docs.flutter.io/flutter/painting/AssetImage-class.html)the class. For example, we can load the background image from the asset declaration above:

``` dart 
Widget build(BuildContext context) {
 return new DecoratedBox(
   decoration: new BoxDecoration(
     image: new DecorationImage(
       image: new AssetImage('graphics/background.png'),
     ),
   ),
 );
}

```

Note that it `AssetImage`is not a widget, it is actually one `ImageProvider`. Sometimes you may expect to get a widget that displays pictures directly, then you can use `Image.asset()`methods such as:

``` dart 
Widget build(BuildContext context) {
 return Image.asset('graphics/background.png');
}

```

When using the default asset bundle to load resources, the resolution is automatically processed internally, which is imperceptible to developers. (If you use some of the lower-level classes, such as [`ImageStream`](https://docs.flutter.io/flutter/painting/ImageStream-class.html)or [`ImageCache`](https://docs.flutter.io/flutter/painting/ImageCache-class.html)you will notice that there are associated with the scaling parameter)

#### Depends on the resource image in the package

To load the image in the dependent package, you must `AssetImage`provide `package`parameters.

For example, suppose your application depends on a package named "my_icons", which has the following directory structure:

-   … / Pubspec.yaml
-   …/icons/heart.png
-   …/icons/1.5x/heart.png
-   …/icons/2.0x/heart.png
-   …etc.

Then load the image, use:

``` dart 
new AssetImage('icons/heart.png', package: 'my_icons')

```

or

``` dart 
new Image.asset('icons/heart.png', package: 'my_icons')

```

**Note: The package should also be obtained by adding `package`parameters when using its own resources** .

##### Assets in the package

If `pubspec.yaml`the desired resource is declared in the file, it will be packaged into the corresponding package. In particular, the resources used by the package itself must be `pubspec.yaml`specified in.

Packages can also choose `lib/`to include `pubspec.yaml`resources in their folders that are not declared in their files. In this case, for the pictures to be packaged, the application must `pubspec.yaml`specify which images to include. For example, a package named "fancy_backgrounds" may contain the following files:

-   …/lib/backgrounds/background1.png
-   …/lib/backgrounds/background2.png
-   …/lib/backgrounds/background3.png

To include the first image, it must be `pubspec.yaml`declared in the assets section:

``` dart 
flutter:
 assets:
   - packages/fancy_backgrounds/backgrounds/background1.png

```

`lib/`Is implicit, so it should not be included in the asset path.

### Specific platform assets

The above resources are all in the flutter application. These resources can only be used after the Flutter framework is running. If we want to set an APP icon or add a launch image to our application, then we must use the assets of a specific platform.

#### Set app icon

The way to update the startup icon of the Flutter application is the same as that of updating the startup icon in the native Android or iOS application.

-   Android
   
   In the root directory of the Flutter project, navigate to the `.../android/app/src/main/res`directory, which contains various resource folders (for example, the `mipmap-hdpi`placeholder image "ic_launcher.png" is already included, see Figure 2-8). Just follow the instructions in the [Android Developer Guide](https://developer.android.com/guide/practices/ui_guidelines/icon_design_launcher.html#size) , replace it with the required resources, and follow the recommended icon size standards for each screen density (dpi).
   
   ![Figure 2-8](https://pcdn.flutterchina.club/imgs/2-8.png)
   
   > **Note:** If you rename a .png file, you must also at your `AndroidManifest.xml`'s label update name attribute.`<application>``android:icon`
   
-   iOS
   
   In the root directory of the Flutter project, navigate to `.../ios/Runner`. The directory `Assets.xcassets/AppIcon.appiconset`already contains placeholder pictures (see Figure 2-9), just replace them with pictures of appropriate size and keep the original file name.
   
   ![Figure 2-9](https://pcdn.flutterchina.club/imgs/2-9.png)
   

#### Update start page

![Figure 2-10](https://pcdn.flutterchina.club/imgs/2-10.png)

When the Flutter framework is loaded, Flutter will use the local platform mechanism to draw the startup page. This launch page will last until the first frame of Flutter rendering the application.

> **Note:** This means that if you do not `main()`call the [runApp](https://docs.flutter.io/flutter/widgets/runApp.html) function in the application method (or more specifically, if you do not call [`window.render`](https://docs.flutter.io/flutter/dart-ui/Window/render.html)to respond [`window.onDrawFrame`](https://docs.flutter.io/flutter/dart-ui/Window/onDrawFrame.html)), the splash screen will always be displayed.

##### Android

To add a splash screen to your Flutter application, navigate to `.../android/app/src/main`. Now `res/drawable/launch_background.xml`, through the custom drawable to achieve a custom start interface (you can also directly change a picture).

##### iOS

To add a picture to the center of the splash screen, navigate to `.../ios/Runner`. In `Assets.xcassets/LaunchImage.imageset`, dragged into the picture and name `LaunchImage.png`, `LaunchImage@2x.png`, `LaunchImage@3x.png`. If you use a different file name, you must also update the `Contents.json`file in the same directory. The specific size of the picture can be found in Apple's official standard.

You can also fully customize the storyboard by opening Xcode. Navigate to the Project Navigator `Runner/Runner`and `Assets.xcassets`drag in the picture by opening it , or customize it by using Interface Builder in LaunchScreen.storyboard, as shown in Figure 2-11.

![Figure 2-11](https://pcdn.flutterchina.club/imgs/2-11.png)