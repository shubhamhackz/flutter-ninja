# 2.5 Debug Flutter application
There are various tools and features to help debug Flutter applications.

# Dart analyzer
Before running the application, please run and flutter analyzetest your code. This tool is a static code inspection tool. It is dartanalyzera package of the tool. It is mainly used to analyze the code and help developers find possible errors. For example, the Dart analyzer uses a lot of type annotations in the code to help track down problems, avoid var, Untyped parameters, untyped list text, etc.

If you use IntelliJ's Flutter plug-in, then the analyzer is automatically enabled when you open the IDE. If the reader is using another IDE, it is strongly recommended that the reader enable the Dart analyzer, because most of the time, the Dart analyzer can be run in the code Found most problems before.

# Dart Observatory (statement-level single-step debugging and analyzer)
If we use the flutter runstartup application, then when it is running, we can open the Web page of the Observatory tool, for example, Observatory listens by default http://127.0.0.1:8100/ (opens new window), You can open the link directly in your browser. Connect directly to your application using the statement-level stepping debugger. If you are using IntelliJ, you can also use its built-in debugger to debug your application.

Observatory also supports analysis and inspection of the heap. For more information about Observatory, please refer to Observatory documentation (opens new window).

If you use Observatory for analysis, make sure to run the application through the --profileoption to run the flutter runcommand. Otherwise, the main problem that will appear in the configuration file will be debugging assertions to verify various invariants of the framework (see "Debug Mode Assertions" below).

# debugger() statement
When using Dart Observatory (or another Dart debugger, such as the debugger in IntelliJ IDE), you can use this debugger()statement to insert programmatic breakpoints. To use this, you must add it import 'dart:developer';to the top of the relevant file.

debugger()The statement takes an optional whenparameter, which you can specify to break only when certain conditions are true, as shown below:

``` dart 
[void someFunction(double offset) {
 debugger(when: offset > 30.0);
 // ...
}](url)
```
# print, debugPrint,flutter logs
The Dart print()function will output to the system console, which you can use flutter logsto view it (basically a wrapper adb logcat).

If you output too much at a time, Android sometimes discards some log lines. To avoid this, you can use the Flutter foundationlibrarydebugPrint() (opens new window). This is a package print, which limits the output to one level to avoid being discarded by the Android kernel.

Many classes in the Flutter framework have toStringimplementations. By convention, these outputs usually include runtimeTypesingle-line output of the object , usually in the form ClassName (more information about this instance...). Some classes used in the tree also have toStringDeepa multi-line description of the entire subtree from that point. Some toStringclasses with detailed information will implement one toStringShort, which only returns the type of object or other very brief (one or two words) description.

# Debug mode assertion
During Flutter application debugging, Dart assertstatements are enabled, and the Flutter framework uses it to perform many runtime checks to verify whether some immutable rules are violated.

When an immutable rule is violated, it is reported to the console with some contextual information to help track down the source of the problem.

To turn off debug mode and use release mode, use flutter run --releaseRun your application. This also turns off the Observatory debugger. An intermediate mode can turn off all debugging aids except Observatory, called "profile mode", just use it --profileinstead --release.

#Debug application layer
Each layer of the Flutter framework provides the debugPrintfunction of dumping its current state or event to the console (used ).

# Widget tree
To dump the state of the Widgets tree, calldebugDumpApp() (opens new window). As long as the application has been built at least once (that is, at build()any time after the call ), you can build()call this method ( runApp()after the call) at any time when the application is not in the construction phase (that is, not called within a method ).

For example, this application:

``` dart 
import 'package:flutter/material.dart';

void main() {
 runApp(
   new MaterialApp(
     home: new AppHome(),
   ),
 );
}

class AppHome extends StatelessWidget {
 @override
 Widget build(BuildContext context) {
   return new Material(
     child: new Center(
       child: new FlatButton(
         onPressed: () {
           debugDumpApp();
         },
         child: new Text('Dump App'),
       ),
     ),
   );
 }
}
```
…Will output something like this (the exact details will vary according to the version of the framework, the size of the device, etc.):
``` dart 
I/flutter ( 6559): WidgetsFlutterBinding - CHECKED MODE
I/flutter ( 6559): RenderObjectToWidgetAdapter<RenderBox>([GlobalObjectKey RenderView(497039273)]; renderObject: RenderView)
I/flutter ( 6559): └MaterialApp(state: _MaterialAppState(1009803148))
I/flutter ( 6559):  └ScrollConfiguration()
I/flutter ( 6559):   └AnimatedTheme(duration: 200ms; state: _AnimatedThemeState(543295893; ticker inactive; ThemeDataTween(ThemeData(Brightness.light Color(0xff2196f3) etc...) → null)))
I/flutter ( 6559):    └Theme(ThemeData(Brightness.light Color(0xff2196f3) etc...))
I/flutter ( 6559):     └WidgetsApp([GlobalObjectKey _MaterialAppState(1009803148)]; state: _WidgetsAppState(552902158))
I/flutter ( 6559):      └CheckedModeBanner()
I/flutter ( 6559):       └Banner()
I/flutter ( 6559):        └CustomPaint(renderObject: RenderCustomPaint)
I/flutter ( 6559):         └DefaultTextStyle(inherit: true; color: Color(0xd0ff0000); family: "monospace"; size: 48.0; weight: 900; decoration: double Color(0xffffff00) TextDecoration.underline)
I/flutter ( 6559):          └MediaQuery(MediaQueryData(size: Size(411.4, 683.4), devicePixelRatio: 2.625, textScaleFactor: 1.0, padding: EdgeInsets(0.0, 24.0, 0.0, 0.0)))
I/flutter ( 6559):           └LocaleQuery(null)
I/flutter ( 6559):            └Title(color: Color(0xff2196f3))
... 
```
This is a "flattened" tree that shows all the widgets projected through various construction functions (if you call in the root of the widget tree toStringDeepwidget, this is the tree you get). You will see a lot of widgets that do not appear in your application source code because they are build()inserted by widget functions in the framework . E.g,InkFeature (opens new window)It is an implementation detail of Material widget.

When the button changes from being pressed to being released, debugDumpApp() is called, and the FlatButton object is called at the same time setState()and marks itself as "dirty". This is why if you look at the dump, you will see that specific objects are marked as "dirty". You can also check which gesture listeners have been registered; in this case, a single GestureDetector is listed and listens to the "tap" gesture ("tap" is TapGestureDetectorthe toStringShortoutput of the function)

If you write your own widget, you can overridedebugFillProperties() (opens new window)To add information. Will DiagnosticsProperty (opens new window)The object is used as a method parameter, and the parent method is called. This function is toStringused by this method to fill in the description information of the widget.

# Render tree
If you are trying to debug a layout issue, the Widget tree may not be detailed enough. In this case, you can debugDumpRenderTree()dump the render tree by calling . Just as debugDumpApp()you can call this function at any time except for the layout or drawing phase. As a general rule, call back from the frame (opens new window)Or calling it in the event handler is the best solution.

To call debugDumpRenderTree(), you need to add import'package:flutter/rendering.dart';to your source file.

The output of the above small example is as follows:
``` dart 
I/flutter ( 6559): RenderView
I/flutter ( 6559):  │ debug mode enabled - android
I/flutter ( 6559):  │ window size: Size(1080.0, 1794.0) (in physical pixels)
I/flutter ( 6559):  │ device pixel ratio: 2.625 (physical pixels per logical pixel)
I/flutter ( 6559):  │ configuration: Size(411.4, 683.4) at 2.625x (in logical pixels)
I/flutter ( 6559):  │
I/flutter ( 6559):  └─child: RenderCustomPaint
I/flutter ( 6559):    │ creator: CustomPaint ← Banner ← CheckedModeBanner ←
I/flutter ( 6559):    │   WidgetsApp-[GlobalObjectKey _MaterialAppState(1009803148)] ←
I/flutter ( 6559):    │   Theme ← AnimatedTheme ← ScrollConfiguration ← MaterialApp ←
I/flutter ( 6559):    │   [root]
I/flutter ( 6559):    │ parentData: <none>
I/flutter ( 6559):    │ constraints: BoxConstraints(w=411.4, h=683.4)
I/flutter ( 6559):    │ size: Size(411.4, 683.4)
... 
```
This is the output RenderObjectof the toStringDeepfunction of the root object .

When debugging layout issues, the key thing to look at is the sizeand constraintsfield. Constraints are passed down the tree and dimensions are passed up.

If you write your own rendering object, you can overridedebugFillProperties() (opens new window)Add information to the dump. Will DiagnosticsProperty (opens new window)The object is used as the parameter of the method, and the parent method is called.

# Layer tree
Readers can understand that the rendering tree can be layered, and the final drawing needs to be combined with different layers, and Layer is the layer that needs to be combined when drawing, if you try to debug the composition problem, you can usedebugDumpLayerTree() (opens new window). For the above example, it will output:

``` dart 
I/flutter : TransformLayer
I/flutter :  │ creator: [root]
I/flutter :  │ offset: Offset(0.0, 0.0)
I/flutter :  │ transform:
I/flutter :  │   [0] 3.5,0.0,0.0,0.0
I/flutter :  │   [1] 0.0,3.5,0.0,0.0
I/flutter :  │   [2] 0.0,0.0,1.0,0.0
I/flutter :  │   [3] 0.0,0.0,0.0,1.0
I/flutter :  │
I/flutter :  ├─child 1: OffsetLayer
I/flutter :  │ │ creator: RepaintBoundary ← _FocusScope ← Semantics ← Focus-[GlobalObjectKey MaterialPageRoute(560156430)] ← _ModalScope-[GlobalKey 328026813] ← _OverlayEntry-[GlobalKey 388965355] ← Stack ← Overlay-[GlobalKey 625702218] ← Navigator-[GlobalObjectKey _MaterialAppState(859106034)] ← Title ← ⋯
I/flutter :  │ │ offset: Offset(0.0, 0.0)
I/flutter :  │ │
I/flutter :  │ └─child 1: PictureLayer
I/flutter :  │
I/flutter :  └─child 2: PictureLayer
```
This is Layerthe toStringDeepoutput of root .

The transformation of the root is the transformation of the applied device pixel ratio; in this case, each logical pixel represents 3.5 device pixels.

RepaintBoundaryThe widget creates one in the layer of the render tree RenderRepaintBoundary. This is used to reduce the need for redrawing.

# Semantics
You can also calldebugDumpSemanticsTree() (opens new window)Get a dump of the semantic tree (the tree presented to the system accessibility API). To use this feature, you must first enable accessibility features, such as enabling system accessibility tools or SemanticsDebugger(discussed below).

For the above example, it will output:

``` dart 
I/flutter : SemanticsNode(0; Rect.fromLTRB(0.0, 0.0, 411.4, 683.4))
I/flutter :  ├SemanticsNode(1; Rect.fromLTRB(0.0, 0.0, 411.4, 683.4))
I/flutter :  │ └SemanticsNode(2; Rect.fromLTRB(0.0, 0.0, 411.4, 683.4); canBeTapped)
I/flutter :  └SemanticsNode(3; Rect.fromLTRB(0.0, 0.0, 411.4, 683.4))
I/flutter :    └SemanticsNode(4; Rect.fromLTRB(0.0, 0.0, 82.0, 36.0); canBeTapped; "Dump App")
```
# Scheduling
To find out where the start/end event occurred relative to the frame, you can switchdebugPrintBeginFrameBanner (opens new window)anddebugPrintEndFrameBanner (opens new window)Boolean to print the beginning and end of the frame to the console.

E.g:
``` dart 
I/flutter : ▄▄▄▄▄▄▄▄ Frame 12         30s 437.086ms ▄▄▄▄▄▄▄▄
I/flutter : Debug print: Am I performing this work more than once per frame?
I/flutter : Debug print: Am I performing this work more than once per frame?
I/flutter : ▀▀▀▀▀▀▀▀▀▀▀▀▀▀▀▀▀▀▀▀▀▀▀▀▀▀▀▀▀▀▀▀▀▀▀▀▀▀▀▀▀▀▀▀▀▀▀▀▀▀▀▀
```
debugPrintScheduleFrameStacks (opens new window)It can also be used to print the call stack that caused the current frame to be scheduled.

#Visual debugging
You can also set up debugPaintSizeEnabledto truevisually debug layout problems. This is renderinga boolean value from the library. It can be enabled at any time and affects drawing when it is true. The easiest way to set it up is to set it at void main()the top of the.

When it is enabled, all boxes will get a bright dark cyan border, padding (from widgets such as Padding) is displayed in light blue, sub-widgets are surrounded by a dark blue box, alignment (from widgets such as Center and Align) Displayed as a yellow arrow. Blank (such as a Container without any child nodes) is displayed in gray.

debugPaintBaselinesEnabled (opens new window)Do a similar thing, but for objects with a baseline, the text baseline is displayed in green and the ideographic baseline is displayed in orange.

debugPaintPointersEnabled (opens new window)The logo turns on a special mode, and any objects that are being clicked will be highlighted in dark cyan. This can help you determine whether an object performs a hit test in an incorrect way (Flutter detects whether there is a widget that can respond to user actions at the clicked position), for example, if it actually exceeds the scope of its parent, first Will not consider passing the hit test.

If you are trying to debug composite layers, for example to determine if and where to add RepaintBoundarywidgets, you can usedebugPaintLayerBordersEnabled (opens new window)Logo, the logo uses orange or outline lines to mark the boundaries of each layer, or usedebugRepaintRainbowEnabled (opens new window)The logo, as long as they are redrawn, this will cause the layer to be covered by a set of rotating colors.

All these flags can only work in debug mode. Generally, debug...anything starting with " " in the Flutter framework can only work in debug mode.

# Debug animation
The easiest way to debug animations is to slow them down. To do this, pleasetimeDilation (opens new window)The variable (in the scheduler library) is set to a number greater than 1.0, such as 50.0. It is best to set it only once when the application starts. If you change it during the run, especially if you change its value to a smaller value while the animation is running, you may experience a regression in the observation, which may result in an assertion hit, and this usually interferes with our development work.

# Debug performance issues
To understand why your application is causing re-layout or re-drawing, you can set thedebugPrintMarkNeedsLayoutStacks (opens new window)and debugPrintMarkNeedsPaintStacks (opens new window)Sign. Whenever the render box is required to re-layout and re-draw, these will log the stack trace to the console. If this method is useful to you, you can use servicesthe debugPrintStack()methods in the library to print stack traces on demand.

# Statistic application startup time
To collect detailed information about the time it takes for a Flutter application to start, you can flutter runuse trace-startupand profileoptions at runtime .
``` dart 
$ flutter run --trace-startup --profile
```
The trace output is saved start_up_info.jsonin the build directory in the Flutter project directory. The output lists the time taken from the start of the application to these trace events (captured in microseconds):

When entering the Flutter engine.
When showing the first frame of the app.
When initializing the Flutter framework.
When the Flutter framework is initialized.
like :
``` dart 
{
 "engineEnterTimestampMicros": 96025565262,
 "timeToFirstFrameMicros": 2171978,
 "timeToFrameworkInitMicros": 514585,
 "timeAfterFrameworkInitMicros": 1657393
}
```
# Track Dart code performance
To perform custom performance tracking and measure the wall/CPU time of any code segment in Dart (similar to using systrace on Android (opens new window)). Use dart:developerof Timeline (opens new window)Tools to include the code block you want to test, for example:
``` dart 
Timeline.startSync('interesting function');
// iWonderHowLongThisTakes();
Timeline.finishSync();
```
Then open the Observatory timeline page of your application, select the'Dart' checkbox in "Recorded Streams", and perform the function you want to measure.

Refresh the page will be in Chrome's tracking tool (opens new window)Displays the timeline records of the application in chronological order.

Please ensure that the runtime flutter runis --profilemarked to ensure that the runtime performance characteristics are minimally different from your final product.
