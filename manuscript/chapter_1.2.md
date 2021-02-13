# 1.2 Getting to know Flutter

## 1.2.1 Introduction to Flutter

Flutter is a mobile application development framework launched and open sourced by Google, focusing on cross-platform, high fidelity, and high performance. Developers can develop App through Dart language, same codebase can be used to develop apps for iOS and Android. Flutter provides a wealth of widgets and plugins, and developers can quickly add native extensions to Flutter. At the same time, Flutter also uses the Native engine to render the view, which will undoubtedly provide users with a good experience.

#### Cross-platform self-drawing engine

Flutter is different from most other frameworks used to build mobile applications because Flutter neither uses WebView nor the native controls of the operating system. Instead, Flutter uses its own high-performance rendering engine to draw widgets. This not only ensures the consistency of the UI on Android and iOS, but also avoids the restrictions and high maintenance costs caused by dependence on native controls.

Flutter uses Skia as its 2D rendering engine. Skia is a 2D graphics processing library of Google. It includes fonts, coordinate conversion, and bitmaps with high performance and concise performance. Skia is cross-platform and provides very Friendly API, currently both Google Chrome and Android use Skia as their drawing engine.

Currently Flutter supports three mobile platforms: iOS, Android, and Fuchsia (Google's new self-developed operating system) by default. But Flutter also supports Web development which is in beta and Desktop support which is in alpha . The examples and introductions in this book are mainly based on iOS and Android platforms, and readers of other platforms can understand by themselves.

#### high performance

The high performance of Flutter is mainly guaranteed by two points. First, the Flutter APP is developed in Dart language. In the JIT (Just-in-time compilation) mode, Dart's speed is basically the same as JavaScript. But Dart supports AOT. When running in AOT mode, JavaScript is far behind. The speed increase is very helpful for refreshing under high frame rate. Secondly, Flutter uses its own rendering engine to draw the UI, and the layout data is directly controlled by the Dart language, so there is no need to communicate between JavaScript and Native like React Native during the layout process. In scenarios like sliding and dragging,Flutter has some obvious advantages, because the sliding and dragging process often cause layout changes, so JavaScript needs to constantly synchronize layout information with Native, which is the same as the problem caused by frequently operating DOM in the browser. , Will bring considerable performance overhead.

#### Developed in Dart language

This is very interesting, but also very controversial issue. Before we understand why Flutter chose Dart instead of JavaScript, let's introduce two concepts: JIT and AOT.

Currently, there are two main ways to run programs: compiling and interpreting codes. The compiled programs are all translated into machine code before execution. This is usually called **AOT** (Ahead of time), or "compiled ahead of time". While the interpretation and execution is to translate sentence by sentence while running, usually this type of The type is called **JIT** (Just-in-time) or "just-in-time compilation". The typical representatives of AOT programs are applications developed in C/C++, which must be compiled into machine code before execution, while there are many representatives of JIT, such as JavaScript, python, etc. In fact, all scripting languages ​​support JIT mode. But it should be noted that JIT and AOT refer to the program operation mode, and are not strongly related to the programming language. Some languages ​​can be run in JIT mode or AOT mode, such as Java and Python, which can be executed for the first time When the program is executed, it can be compiled into intermediate bytecode, and then the bytecode can be directly executed when it is executed later. Some people may say that the intermediate bytecode is not machine code, and the bytecode needs to be dynamically converted to machine code during program execution. Yes, this is not wrong, but usually the standard for us to distinguish whether it is AOT is to see whether the code needs to be compiled before execution. As long as it needs to be compiled, whether the compiled product is bytecode or machine code, it belongs to AOT. Here, the reader does not need to be entangled in the concept, the concept is invented for the purpose of conveying the spirit, as long as the reader can understand the principle.

Now let's see why Flutter chooses Dart language? The author summarizes the following points based on the official explanation and my own understanding of Flutter (because other cross-platform frameworks use JavaScript as their development language, we mainly compare Dart and JavaScript):

1.  **High Development Efficiency**
    
    The Dart runtime and compiler support a combination of two key features of Flutter:
    
    **JIT-based rapid development cycle** : Flutter adopts the JIT mode in the development phase, which avoids the need to compile every change and greatly saves development time;
    
    **AOT-based release package** : Flutter can generate efficient ARM code through AOT during release to ensure application performance. JavaScript does not have this capability.
    
2.  **High Performance**
    
    Flutter aims to provide a smooth, high-fidelity UI experience. In order to achieve this, Flutter needs to be able to run a large amount of code in each animation frame. This means that a language that can provide high performance without periodic pauses that will drop frames is needed. Dart supports AOT, which can do better than JavaScript at this point.
    
3.  **Fast Memory Allocation**
    
    The Flutter framework uses functional streams, which makes it largely dependent on the underlying memory allocator. Therefore, it is very important to have a memory allocator that can effectively handle trivial tasks. Flutter will not work effectively in languages ​​lacking this feature. Of course, the JavaScript engine of Chrome V8 has also done a good job in memory allocation. In fact, many members of the Dart development team are from the Chrome team, so Dart cannot be used as an advantage over JavaScript in memory allocation, but for Flutter Say, it needs such features, and Dart just meets it.
    
4.  **Type Safety**
    
    Since Dart is a type-safe language and supports static type detection, some types of errors can be found before compilation and potential problems can be eliminated, which may be more attractive to front-end developers. In contrast, JavaScript is a weakly typed language, so many extension languages ​​and tools that add static type detection to JavaScript code have appeared in the front-end community, such as Microsoft's TypeScript and Facebook's Flow. In contrast, Dart itself supports static typing, which is an important advantage of it.
    
5.  **The Dart team is by your side**
    
    It may seem inconspicuous, but it is very important. Thanks to the active investment of the Dart team, the Flutter team can get more and more convenient support. As stated on the Flutter official website, "We are working closely with the Dart community to improve the use of Dart in Flutter. For example, when we initially adopted Dart, the language did not provide a toolchain for generating native binary files (which helps to achieve high performance), but it is now implemented because the Dart team built it specifically for Flutter. 
    

#### to sum up

This section mainly introduces the characteristics of Flutter. If you feel that some points are not well understood, don't worry, as you understand the details of Flutter in the future, I believe you will have a deeper experience when you look back.

## 1.2.2 Flutter frame structure

In this section, we first give an overall introduction to the Flutter framework, aiming to give readers an overall impression, which is very important for beginners. If you dive into Flutter all at once, you will be like a person without a map in the desert. Even if you can find an oasis, you will not know where the next oasis is. Therefore, no matter what technology we learn, we must first have a clear "map", and our learning process should be based on a clear picture, so that we will not be trapped in the details and "being incomplete". Now, let's take a look at the Flutter framework diagram officially provided by Flutter, as shown in Figure 1-1:

![Picture 1-1](https://github.com/shubhamhackz/flutter-ninja/blob/main/resources/1-1.png)

### Flutter Framework
Flutter is designed as an extensible, layered system. It exists as a series of independent libraries that each depend on the underlying layer. No layer has privileged access to the layer below, and every part of the framework level is designed to be optional and replaceable.This is a pure Dart SDK, which implements a set of basic libraries.

 From the bottom up, let's briefly introduce:

- Basic foundational classes, and building block services such as animation, painting, and gestures that offer commonly used abstractions over the underlying foundation.
- The rendering layer provides an abstraction for dealing with layout. With this layer, you can build a tree of renderable objects. You can manipulate these objects dynamically, with the tree automatically updating the layout to reflect your changes.
- The widgets layer is a composition abstraction. Each render object in the rendering layer has a corresponding class in the widgets layer. In addition, the widgets layer allows you to define combinations of classes that you can reuse. This is the layer at which the reactive programming model is introduced.
- The Material and Cupertino libraries offer comprehensive sets of controls that use the widget layer’s composition primitives to implement the Material or iOS design languages.
    

### Flutter Engine

This is a pure C++ SDK, which includes Skia engine, Dart runtime, text typesetting engine, etc. The engine is responsible for rasterizing composited scenes whenever a new frame needs to be painted. It provides the low-level implementation of Flutter’s core API, including graphics (through Skia), text layout, file and network I/O, accessibility support, plugin architecture, and a Dart runtime and compile toolchain.

### to sum up

The Flutter framework itself has a good layered design. This section aims to give readers a general impression of the overall framework of Flutter. I believe that up to now, readers have an initial impression of Flutter. Before we officially start, we still need to understand Dart.Let's take a look at Dart, the development language of Flutter.

## 1.2.3 How to learn Flutter

This section gives you some learning suggestions and shares some of the author's experience in learning Flutter, hoping to help you improve your learning efficiency and avoid unnecessary pits.

### Resources

-   **Official website** : Reading the resources of Flutter official website is the best way to get started quickly. At the same time, the official website is also a place to learn about the latest developments in Flutter. Since Flutter is still in the rapid development stage, readers are advised to go to the official website from time to time to see if there are any new trends. .
    
-   **Source code and comments** : The source code comments should be used as the first document for learning Flutter. The source code of the Flutter SDK is open source, and the comments are very detailed. There are also many examples. In fact, the official Flutter SDK documentation is generated through comments. Source code combined with comments can help you solve most problems.
    
-   **Github** : If the problem you encounter is not answered on StackOverflow, you can go to the flutter's github project to raise an issue.
-   **Gallery source code** : Gallery is the official sample app of Flutter. There are rich examples in it. Readers can download and install it online. The source code of Gallery is in the "examples" directory of the Flutter source code.

### community

-   **StackOverflow** : If you have not heard of StackOverflow, this is currently the world's largest programmer Q&A community, and it is also the most active Flutter Q&A community. In addition to Flutter users from all over the world on StackOverflow, members of the Flutter development team will often answer questions on it.
-  **Flutter Medium** : Get the latest news and insights from a diverse group of users building with Flutter.
-  **Twitter** : Follow the Flutter team in real-time with information on new features, upcoming events, and more.
-  **Blog** : With the promotion of Flutter technology, The internet in filled with informative flutter articles and blogs. Just do a google search and the world is yours.

### to sum up

With the information and community, the most important thing for us learners is to do more and practice more. In the later chapters of this book, we hope that readers can write examples by themselves. Are you ready? In the next chapter, we will officially enter the world of Flutter!
