# 1.1 Introduction to mobile development technology

In this section I'll explain the evolution of mobile development technologies. To understand why flutter is exciting and what makes it different from other technologies, we must understand how mobile development was done before it appeared.

## 1.1.1 Native development and Cross-platform technology

### Native development

Native development refers to mobile application development using the development tools, SDKs, and languages supported by a particular operating system such as iOS or Android. For example, a native Android app refers to an application developed in languages like Java or Kotlin using the Android SDK; while a native iOS app refers to an application developed in languages like Objective-C or Swift using the iOS SDK.

Advantages of Native Development:

- Access to all features of the platform (GPS,Camera,Microphone).
- Flawless performance.
- Complex animation and drawing can be achieved.
- Provides the best user experience.

Disadvantages of Native Development:

- Platform-specific (high development costs).
- Different platforms must maintain different codebases (increased development time).
- Delivering the same feature across all platforms at the same time can sometimes be challenging because of different codebases.

In the early days of mobile development app development for businesses was not complicated, and native development could cope with product demand iterations. However, in recent years, with the advent of the Internet of Things and the rapid advancement of the Internet, in many business scenarios traditional pure native development can no longer meet the growing needs of businesses.

Mainly manifested in:

- As the demand for dynamic content surges, when the demand changes, native applications need to update the app through version updates. This involves pushing an updated build to the play store or app store and going through the entire review and release process again. Which is difficult to deal with in this fast-changing Internet era. So the necessity of dynamic application updates (application content can be updated without publishing a version) becomes crucial.
- Rapid change of business requirements results in high development costs. Since native development requires maintaining two codebases and two development teams for Android and iOS, updating the app results in increased labor cost and development time..

To sum up, pure native development mainly faces two problems of dynamization and development cost, and for these two problems some cross-platform dynamization frameworks have been born.

### Introduction to cross-platform technology

People have been trying to find good solutions for the problems faced by native development, and today there are many cross-platform frameworks (note that the "cross-platform" in this book refers to Android and iOS), mainly divided into three categories:

- H5+ native (Cordova, Ionic, WeChat applet)
- JavaScript development + native rendering (React Native, Weex)
- Self-painted UI+Native (QT for mobile, Flutter)

In the following chapters we will look at the principles, advantages and disadvantages of these three types of frameworks one by one.

## 1.1.2 Introduction to Hybrid Technology

### H5+ native hybrid development (HTML5)

The main principle of this type of framework is to implement part of the APP that needs to be dynamically changed through H5 (HTML5), and load it through the native web page loading control WebView (Android) or WKWebView (iOS) (in the future, if there are no special instructions, we will use WebView to uniformly refer to the system component used in Android and iOS which allows native apps to display web content). In this way, the H5 part can be changed at any time without publishing, and dynamic requirements can be met. Because the H5 code only needs to be developed once, it can run on both Android and iOS platforms at the same time. Which can also reduce development costs. In other words, the more functions of the H5 part, the lower the development cost. We call this H5+ native development model **hybrid development** , and **apps** developed in hybrid models are called **hybrid apps** or **hybrid** **apps** . If most of the functions of an app are implemented by H5, we call them **Web APP** .

Typical representatives of current hybrid development frameworks are: Cordova, Ionic and WeChat applets. It is worth mentioning that WeChat applets are currently rendered in WebView, not native rendering, but native rendering may be used in the future.

### Hybrid development technology points

As mentioned earlier, native development can access all the functions of the platform. In hybrid development, H5 code runs in WebView. WebView is essentially a browser kernel and its JavaScript is still running in a sandbox with restricted permissions. Therefore, there is no access permission for most system capabilities, such as inability to access the file system, and inability to use Bluetooth. Therefore, all functions that cannot be implemented by H5 need to be done natively. The hybrid framework generally implements some APIs for accessing system capabilities in the native code in advance, and then exposes them to WebView for JavaScript to call. In this way, WebView becomes the communication bridge between JavaScript and native API, mainly responsible for JavaScript and native API. Call messages are transferred between, and the transfer of messages must comply with a standard protocol, which specifies the format and meaning of the message. We use WebView to communicate between JavaScript and native and implement a certain message transfer protocol The tool is called **WebView JavaScript Bridge** , or **JsBridge for short** , and it is also the core of the hybrid development framework.

#### Example: JavaScript calls native API to get phone model

Let's take Android as an example to implement a native API for obtaining the phone model for JavaScript calls. In this example, the process of calling native API by JavaScript will be shown. Readers can intuitively experience the process of calling. We use the wendux's open source dsBridge on Github as JsBridge for communication. dsBridge is a cross-platform JsBridge that supports synchronous calling. In this example, only the synchronous calling function is used.

1. First implement the API to get the phone model in the native `getPhoneModel`

```dart
   class JSAPI {
    @JavascriptInterface
    public Object getPhoneModel(Object msg) {
      return Build.MODEL;
    }
   }

```

2. Register native API to JsBridge through WebView

```dart
   import wendu.dsbridge.DWebView
   ...
   //DWebView inherits from WebView, provided by dsBridge
   DWebView dwebView = (DWebView) findViewById(R.id.dwebview);
   //Register native API to JsBridge
   dwebView.addJavascriptObject(new JsAPI(), null);

```

3. Call native API in JavaScript

```dart
   var dsBridge = require("dsbridge")
   //Directly call the native API `getPhoneModel
   var model = dsBridge.call("getPhoneModel");
   //print model
   console.log(model);

```

The above example demonstrates the process of JavaScript calling native API. Similarly, the excellent JsBridge also supports native calling JavaScript. dsBridge also supports it. If you are interested you can find more info on the github dsBridge project homepage.

Now, let’s look back. Hybrid applications are nothing more than pre-implementing a series of APIs for JavaScript calls, so that JavaScript has the ability to access the system. Seeing this, I believe you can also implement a hybrid development framework yourself.

### to sum up

The advantages of hybrid applications are; the dynamic content is H5, the web technology stack, the community and resources are rich. But the disadvantage is that the performance is not good. For complex user interfaces or animations, WebView is unbearable.

## 1.1.3 React Native, Weex and fast apps

This article mainly introduces the cross-platform framework principle of **JavaScript development + native rendering** .

React Native (RN for short) is a cross-platform mobile application development framework open sourced by Facebook in April 2015. It is a derivative of Facebook's earlier open source JS framework React in the native mobile application platform. It currently supports both iOS and Android platforms. RN uses the Javascript language, JSX (similar to HTML), and CSS to develop mobile applications. Therefore, technical personnel familiar with web/front-end development can enter the field of mobile application development with little learning.

Since RN and React have the same principles, and Flutter is also inspired by React, many ideas are also connected. So, it is necessary for us to learn more about React principles. React is a reactive web framework. Let's first understand two important concepts: DOM tree and reactive programming.

### DOM tree and control tree

The Document Object Model (DOM) is a standard programming interface recommended by the W3C organization for processing extensible markup languages. It is a platform and language-independent way to access and modify the content and structure of a document. In other words, this is a standard interface for representing and processing an HTML or XML document. Simply put, DOM is the document tree, which corresponds to the user interface control tree. In front-end development it usually refers to the rendering tree corresponding to HTML. However, the broad DOM can also refer to the control tree corresponding to the XML layout file in Android, and the term **DOM operation** It means to directly manipulate the rendering tree (or control tree). Therefore, you can see that the DOM tree and the control tree are equivalent concepts, but the former is often used in Web development, and the latter is often used in native development.

### Reactive programming

An important idea is put forward in React: the UI changes automatically when the state changes, and the React framework itself performs the work of rebuilding the user interface in response to user state changes. This is a typical **reactive** programming paradigm. Let’s summarize React.

React principle:

- Developers only need to pay attention to state transition (data). When the state changes, the React framework will automatically rebuild the UI based on the new state.
- After the React framework receives the user state change notification, it will calculate the changed part of the tree. To do that is uses a Diff algorithm which takes the current rendering tree, combined with the latest state change, and then updates only the changed part of the DOM. Thereby the reconstruction of the entire tree is avoided which improves performance.

It is worth noting that in the second step, the React framework will not immediately calculate and render the changed part of the DOM tree after the state changes. On the contrary, React will build an abstraction layer on the basis of the **DOM** , namely the **virtual DOM** tree. Any changes made to the data and state will be automatically and efficiently synchronized to the virtual DOM, and finally synchronized to the real DOM in batches, instead of operating the DOM every time it changes. Why shouldn't we directly manipulate the DOM tree on every state change? Because every DOM operation in the browser may cause the browser to redraw or reflow:

1. If the DOM change is related only to style appearance, such as a color change, it will cause the browser to redraw the interface.
2. If the structure of the DOM tree changes, such as size, layout, node hiding, etc., the browser needs to reflow (and re-layout).

Browser redrawing and reflow are both relatively expensive operations. If each change is performed directly on the DOM, this will cause performance problems, and batch operations will only trigger one DOM update.

> Questions: Shouldn’t the Diff operation and DOM batch update be the responsibility of the browser? Is it appropriate to do it in a third-party framework?
>
> An illustration is required here

### React Native

As mentioned above, React Native is a derivative of React in the native mobile application platform. What is the main difference between the two? In fact, the main difference is what is the object mapped by the virtual DOM? The virtual DOM in React will eventually be mapped to the browser DOM tree, and the virtual DOM in RN will be mapped to the native control tree through JavaScriptCore.

JavaScriptCore is a JavaScript interpreter, which has two main functions in React Native:

1. Provide a runtime environment for JavaScript.
2. It is a communication bridge between JavaScript and native applications. It has the same function as JsBridge. In fact, in iOS, many JsBridge implementations are based on JavaScriptCore.

The process of mapping virtual DOM to native control in RN is divided into two steps:

1. Layout message transfer; transfer virtual DOM layout information to native;
2. Natively render the control tree through the corresponding native control based on the layout information;

So far, React Native has achieved cross-platform. Since React Native has native control rendering, the performance will be much better than H5 in hybrid applications. Also, React Native uses the web development technology stack and only needs to maintain a single codebase. This makes it a cross-platform framework.

### Weex

Weex is a cross-platform mobile development framework released by Alibaba in 2016. Its ideas and principles are similar to React Native. The biggest difference is the syntax level. Weex supports Vue syntax and Rax syntax. Rax's DSL (Domain Specific Language) syntax is based on JSX syntax (used in React). Unlike React, JSX is mandatory in Rax. It does not support creating components in other way, so learning JSX is a necessary foundation for using Rax. React Native only supports JSX syntax.

### to sum up

The main advantages of JavaScript development + native rendering are as follows:

1. Using the Web development technology stack, the community is huge, quick to get started, and the development cost is relatively low.
2. Native rendering. Which means performance is much higher than H5.
3. The dynamic is better and supports hot updates.

Disadvantages:

1. When rendering, communication between JavaScript and native is required. In some scenes, such as dragging, it may cause freezing due to frequent communication.
2. JavaScript is a scripting language that requires JIT (Just In Time) to execute, and there is still a gap between execution efficiency and AOT (Ahead Of Time) code.
3. Because rendering relies on native controls, controls on different platforms need to be maintained separately, and community controls may lag when the system is updated; in addition, its control system will also be restricted by the native UI system. For example, in Android, gesture conflicts The disambiguation rules are fixed. When using nested controls written by different people, gesture conflicts will become very tricky.

## 1.1.4 QT Mobile

In this article, we look at the last cross-platform technology: self-painted UI + native. The idea of ​​this technology is to draw the UI by implementing a unified interface rendering engine on different platforms without relying on the system's native controls, so the consistency of the UI on different platforms can be achieved. Note that the self-drawing engine solves the cross-platform problem of the UI. If it involves calling other system capabilities, it still involves native development.

The advantages of this platform technology are as follows:

1. High performance; since the self-drawing engine calls the system API directly to draw the UI, the performance is close to that of native controls.

2. Flexible, easy to maintain the component library, high fidelity and consistency of UI appearance; since UI rendering does not rely on native controls, there is no need to maintain a set of component libraries separately for controls of different platforms, so the code is easy to maintain. Since the component library is the same set of code and the same rendering engine, the display appearance of the component can achieve high fidelity and high consistency on different platforms; in addition, because it does not rely on native controls, it will not be restricted by the native layout system. This layout system will be very flexible.

Disadvantages:

1. In order to ensure UI rendering performance, self-drawing UI systems generally use AOT mode to compile their release packages, so after the application is released, it cannot dynamically deliver code like Hybrid and RN frameworks that use JavaScript (JIT) as the development language .
2. Low development efficiency: QT uses C++ as its development language, and programming efficiency directly affects APP development efficiency. As a static language, C++ is not as flexible as a dynamic language such as JavaScript in terms of UI development. In addition, C++ requires developers to manually manage memory allocation, there is no garbage collection (GC) mechanism in JavaScript and Java.

You may have guessed that Flutter belongs to this type of cross-platform technology. Yes, Flutter has a self-drawing engine and its own UI layout system. However, the idea of ​​a self-drawing engine is not a new concept, and Flutter is not the first to try to do this. Before it came another typical representative, namely the famous QT.

### Introduction to QT

Qt is a cross-platform C++ graphical user interface application development framework developed by Qt Company in 1991. In 2008, Qt Company technology was acquired by Nokia, and Qt became a programming language tool of Nokia. In 2012, Qt was acquired by Digia. In April 2014, the cross-platform integrated development environment Qt Creator 3.1.0 was officially released, which realized full support for iOS, added WinRT, Beautifier and other plug-ins, abandoned GDB debugging support without Python interface, and integrated Clang-based C /C++ code module, and made adjustments to Android support. So far, it has realized full support for iOS, Android, WP, and it provides application developers with all the functions needed to build a graphical user interface. However, although QT has achieved great success on the PC side and is highly sought after by the community, it has not performed well on the mobile side. In recent years, although QT’s voice can be heard occasionally, it has always been weak and ultimately failed regardless of it's very interesting technology.

Main resons behing QT's Failure :

- QT mobile development community is too small, lack of learning materials, and bad ecology.
- The official promotion is not good and the support is not enough.
- QT was late as the market has been occupied by other dynamic frameworks (Hybrid and RN)
- In mobile development, C++ development has inherent disadvantages compared to Web development stacks, and the direct result is that QT development efficiency is too low.

Based on these four points, although QT is a pioneer in the development of cross-platform self-drawing engines on mobile, it has become a martyr.

## 1.1.5 Flutter is born

"Before coming out after a thousand calls", for so long, and now finally waiting for the protagonist of the book to appear!

Flutter is a framework released by Google for creating cross-platform, high-performance mobile applications. Like QT mobile, Flutter does not use native controls. Instead, both implement a self-drawing engine and use its own layout and drawing system. So, should we be worried, since QT mobile is facing the same problem as Flutter, will Flutter follow the footsteps of QT mobile and become another martyr? To return to this question, let's take a look at the birth process of Flutter:

- At the Google I/O conference in 2017, Google launched for the first time a new cross-platform, high-performance mobile application framework-Flutter.
- In February 2018, Flutter released the first Beta version. In May of the same year, at the 2018 Google I/O Conference, Flutter was updated to the beta 3 version.
- In June 2018, Flutter released the first preview version, which means that Flutter has entered the final stage before the official version (1.0) is released.
- On December 4, 2018, Flutter 1.0 was released at the Flutter Live event, denoting the first "stable" version of the Framework.

Observing its development, in May 2018, Flutter entered the top 100 on the GitHub stars list, with 27k stars. Today (Feb 3, 2021), there are already 112K Stars. After a period of more than 3 years, the Flutter ecosystem has grown rapidly. It can be seen that Flutter has been warmly welcomed among developers, and its future development is worth looking forward to!

Now, let's compare it with QT mobile:

1. Community: Judging from Github, the number of active users of Flutter is growing rapidly. Judging from the questions on Stackoverflow, the Flutter community is now very large. Flutter's documentation and resources are becoming more and more abundant. Many problems encountered during the development process can be answered in Stackoverflow or its github issue.
2. Technical support: Google is now vigorously promoting Flutter. Many of the authors of Flutter are from the Chromium team and are very active on github. From another perspective, from the frequent version releases of Flutter in the first half of this year, it can be seen that Google's investment in Flutter is not small, so there is no need to worry about official technical support.
3. Development efficiency: Flutter's hot reloading can help developers quickly test, build UI, add features, and fix errors faster. Millisecond-level hot reloading can be achieved on iOS and Android emulators or real devices without losing state. This is really great. Believe me, if you are a native developer, after experiencing the development flow of Flutter, you probably won't want to go back to native.
   Based on the above three points, I believe that the readers, like the author, have their own conclusions about the future of Flutter. Now that we have a comprehensive understanding of mobile development technology, we are going to enter the main topic of this book. Are you ready?

## 1.1.6 Summary

This chapter mainly introduces three cross-platform technologies in current mobile development. Now we compare them from a framework perspective, as shown in Table 1-1:
| Technology Type | UI Rendering | Performance | Efficiency | Dynamic | Frame Representative |
| ---------------------------------- | ------------------------- | ----------- | ---------- | -------------------- | -------------------- |
| H5+Native | Webview Rendering | General | High | Stand By | Cordova,Ionic |
| Javascript+Native Rendering | Native Control Rendering | Good | Stand By | In | React Native, Weex |
| Self Painted UI + Native rendering | Call system API Rendering | Good | Stand By | Flutter high, QT low | QT, Flutter |

Table 1-1: Cross-platform technology comparison

The development language in the above table mainly refers to the development language of the UI. The development efficiency refers to the efficiency of the entire development cycle, including coding time, debugging time, and troubleshooting and compatibility time. Dynamization mainly refers to whether to support dynamic code delivery and whether to support hot updates. It is worth noting that Flutter's Release package is compiled in Dart AOT mode by default, so it does not support dynamization, but Dart also has JIT or snapshot running modes, which all support dynamization.
