# 12.1 Development Package

The second chapter has already talked about how to use Package (package), we know that through the package can create shared modular code, this section will focus on how to develop and publish our own Package. A minimal Package includes:

-   A `pubspec.yaml`file: a metadata file that declares the package name, version, author, etc.
-   A `lib`folder: a (public) code package disclosed, should be at least a document`<package-name>.dart`

Flutter Packages are divided into two categories:

-   Dart packages: Some of them may contain specific features of Flutter and therefore have a dependency on the Flutter framework. Such packages are only used for Flutter, such as [`fluro`](https://pub.dartlang.org/packages/fluro)packages.
-   Plug-in package: A dedicated Dart package that contains APIs written in Dart code, and specific implementations for Android (using Java or Kotlin) and iOS (using OC or Swift) platforms, which means that the plug-in includes native code, A specific example is a [`battery`](https://pub.dartlang.org/packages/battery)plug-in package.

Note that although Flutter's Dart runtime and Dart VM runtime are not exactly the same, if these differences are not involved in the Package, then such a package can support both Flutter and Dart VM, such as Dart http network library [dio](https://github.com/flutterchina/dio) .

Below I will lead the reader to develop a Dart Package step by step.

### Step 1: Create Dart package

You can create a Package project through Android Studio: File>New>New Flutter Project, as shown in Figure 12-1:

![Figure 12-1](https://pcdn.flutterchina.club/imgs/12-1.png)

You can also use `--template=package`to perform `flutter create`to create a command:

```
flutter create --template=package hello

```

This will `hello/`create a package project with the following dedicated content under the folder:

-   `lib/hello.dart`: Dart code of Package
    
-   `test/hello_test.dart`: The unit test code of Package.
    

### Implement package

For pure Dart packages, you only need to add functions in the main file or in the files in the directory. To test the package, add [unit tests](https://flutter.io/testing/#unit-testing) in the directory . Let's take a look at how to organize the package code. Let's take shelf Package as an example. Its directory structure is shown in Figure 12-2:`lib/<package  name>.dart``lib``test`[](https://flutter.io/testing/#unit-testing)

![Figure 12-2](https://pcdn.flutterchina.club/imgs/12-2.png)

In the "shelf.dart" in the lib root directory, multiple dart files in the "lib/src" directory are exported:

```
export 'src/cascade.dart';
export 'src/handler.dart';
export 'src/handlers/logger.dart';
export 'src/hijack_exception.dart';
export 'src/middleware.dart';
export 'src/pipeline.dart';
export 'src/request.dart';
export 'src/response.dart';
export 'src/server.dart';
export 'src/server_handler.dart';

```

The source code of the main functions in the Package are in the src directory. Shelf Package also exports a mini library: shelf_io, which mainly handles HttpRequest.

### **Import package**

When we need to use this Package, we can specify the entry file of the package through the "package:" instruction:

```
import 'package:utilities/utilities.dart';

```

Source files in the same package can also be imported using relative paths.

### Generate documentation

You can use the [dartdoc](https://github.com/dart-lang/dartdoc#dartdoc) tool to generate documentation for the Package. All developers need to do is to follow the documentation comment syntax to add documentation comments to the code, and finally use dartdoc to directly generate API documentation (a static website). Documentation comments start with a triple slash "///", such as:

```
/// The event handler responsible for updating the badge in the UI.
void updateBadge() {
  ...
}

```

Please refer to [dartdoc for](https://github.com/dart-lang/dartdoc#dartdoc) detailed documentation syntax .

### Handling of package interdependence

If we are developing a `hello`package that depends on another package, we need to add the dependent package to the `pubspec.yaml`file `dependencies`section. The following code makes the `url_launcher`plug-in API `hello`available in the package:

In the `hello/pubspec.yaml`middle:

```
dependencies:
  url_launcher: ^0.4.2

```

Now in `hello`the `import 'package:url_launcher/url_launcher.dart'`then call the `launch()`method has.

This is no different from referencing a package in a Flutter application or any other Dart project.

However, if it `hello`happens to be a plug-in package and its platform-specific code needs to access the `url_launcher`public platform-specific API, then we also need to add appropriate dependency declarations for the platform-specific build files, as shown below.

**Android**

In `hello/android/build.gradle`:

```
android {
    // lines skipped
    dependencies {
        provided rootProject.findProject(":url_launcher")
    }
}

```

You can now access the class in the `hello/android/src`source code .`import io.flutter.plugins.urllauncher.UrlLauncherPlugin``UrlLauncherPlugin`

**iOS**

In `hello/ios/hello.podspec`:

```
Pod::Spec.new do |s|
  # lines skipped
  s.dependency 'url_launcher'

```

You can now `hello/ios/Classes`source code `#import "UrlLauncherPlugin.h"`and then access the `UrlLauncherPlugin`class.

### Resolve dependency conflicts

Suppose we want `hello`to use `some_package`sum in our package `other_package`, and both packages depend on it `url_launcher`, but they depend on `url_launcher`different versions. Then we have potential conflicts. The best way to avoid this is when specifying dependencies, package authors use [version ranges](https://www.dartlang.org/tools/pub/dependencies#version-constraints) instead of specific versions.

```
dependencies:
  url_launcher: ^0.4.2    # 这样会较好, 任何0.4.x(x >= 2)都可.
  image_picker: '0.1.1'   # 不是很好，只有0.1.1版本.

```

If `some_package`the above dependencies are `other_package`declared and the `url_launcher`version is declared like '0.4.5' or'^0.4.0', pub will be able to solve the problem automatically.

Even `some_package`and `other_package`declared incompatible `url_launcher`version, it still might be and `url_launcher`in a compatible way to work. You can handle conflicts by adding dependency override declarations to the `hello`package `pubspec.yaml`files to force the use of specific versions:

Forced to use `0.4.3`version `url_launcher`in `hello/pubspec.yaml`the:

```
dependencies:
  some_package:
  other_package:
dependency_overrides:
  url_launcher: '0.4.3'

```

If the conflicting dependency is not a package, but an Android-specific library, for example `guava`, the dependency rewrite statement must be added to the Gradle build logic.

Mandatory use of the `23.0`version of the `guava`library, in `hello/android/build.gradle`:

```
configurations.all {
    resolutionStrategy {
        force 'com.google.guava:guava:23.0-android'
    }
}

```

Cocoapods currently does not provide dependency coverage.

### Release Package

Once a package is implemented, we can publish it on [Pub](https://pub.dartlang.org/) so that other developers can easily use it.

Prior to publication, inspection `pubspec.yaml`, `README.md`and `CHANGELOG.md`documentation to ensure the integrity and accuracy of its contents. Then, run the dry-run command to see if everything is ready to go:

```
flutter packages pub publish --dry-run

```

After verifying that it is correct, we can run the release command:

```
flutter packages pub publish

```

> If you encounter a package release failure, first check whether it is due to well-known network reasons. If it is a network problem, you can use a VPN. It should be noted that some proxies will only proxy some APP network requests, such as browsers. It may not be able to proxy dart's network requests, so in this case, even if the proxy is turned on, you still cannot connect to Pub. Therefore, it is safer to use a global proxy or global VPN when publishing Pub packages. If there is no problem with the network, run the release command with administrator privileges (sudo) and try again.  
> In many cases, turning on the global proxy will not let the traffic in the terminal go through the proxy server. Taking socks5 as an example, you should enter the following command in the terminal:
> 
> ```
> export all_proxy=socks5://127.0.0.1:1080
> 
> ```
> 
> At this time, the http and https traffic in the terminal will go through the proxy server, and you can `curl -i https://ip.cn`check whether the proxy setting is successful or not through instructions.