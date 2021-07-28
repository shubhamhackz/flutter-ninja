# 13.2 Implement Localizations

I talked about how the Material component library supports internationalization. In this section, we will introduce how to support multiple languages ​​in our own UI. According to the previous section, we need to implement two classes: one `Delegate`class and one `Localizations`class. Below we will explain through an example.

### Implement the Localizations class

We already know `Localizations`that the main implementation in the class provides localized values, such as text:

``` dart 
//Locale资源类
class DemoLocalizations {
 DemoLocalizations(this.isZh);
 //是否为中文
 bool isZh = false;
 //为了使用方便，我们定义一个静态方法
 static DemoLocalizations of(BuildContext context) {
   return Localizations.of<DemoLocalizations>(context, DemoLocalizations);
 }
 //Locale相关值，title为应用标题
 String get title {
   return isZh ? "Flutter应用" : "Flutter APP";
 }
 //... 其它的值  
}

```

`DemoLocalizations`The middle will return different texts according to the current language. For example `title`, we can define all texts that need to support multiple languages ​​in this class. `DemoLocalizations`An instance of will be created in a `load`method of the Delegate class .

### Implement the Delegate class

The responsibility of the Delegate class is to load new Locale resources when the Locale changes, so it has a `load`method. The Delegate class needs to inherit from the `LocalizationsDelegate`class and implement the corresponding interface. Examples are as follows:

``` dart 
//Locale代理类
class DemoLocalizationsDelegate extends LocalizationsDelegate<DemoLocalizations> {
 const DemoLocalizationsDelegate();

 //是否支持某个Local
 @override
 bool isSupported(Locale locale) => ['en', 'zh'].contains(locale.languageCode);

 // Flutter会调用此类加载相应的Locale资源类
 @override
 Future<DemoLocalizations> load(Locale locale) {
   print("$locale");
   return SynchronousFuture<DemoLocalizations>(
       DemoLocalizations(locale.languageCode == "zh")
   );
 }

 @override
 bool shouldReload(DemoLocalizationsDelegate old) => false;
}

```

`shouldReload`The return value of determines whether to call the `load`method to reload Locale resources when the Localizations component is rebuilt . In general, Locale resources should only be loaded once when Locale is switched, and do not need `Localizations`to `false`be loaded every time when rebuild, so just return . Some people may worry `false`that if you return , the `load`method will not be called when the user changes the system language after the APP is started , so the Locale resource will not be loaded. In fact, whenever Locale change Flutter will then call the `load`method to load the new Locale, regardless of `shouldReload`return `true`or `false`.

### The last step: add multilingual support

As in the previous section, we now need to register the `DemoLocalizationsDelegate`class first , and then use it `DemoLocalizations.of(context)`to dynamically get the current Locale text.

Just `localizationsDelegates`add our Delegate instance to the list of MaterialApp or WidgetsApp to complete the registration:

``` dart 
localizationsDelegates: [
// 本地化的代理类
GlobalMaterialLocalizations.delegate,
GlobalWidgetsLocalizations.delegate,
// 注册我们的Delegate
DemoLocalizationsDelegate()
],

```

Next we can use the Locale value in the Widget:

``` dart 
return Scaffold(
 appBar: AppBar(
   //使用Locale title  
   title: Text(DemoLocalizations.of(context).title),
 ),
 ... //省略无关代码
）

```

In this way, when the system language is switched between US English and Simplified Chinese, the title of the APP will be "Flutter APP" and "Flutter APP" respectively.

### to sum up

In this section, we use a simple example to illustrate the basic process and principles of Flutter application internationalization. But the above example also has a serious shortcoming that we need `title`to manually judge the current language Locale when getting it in the DemoLocalizations class , and then return the appropriate text. Imagine that when the languages ​​we want to support are not two, but 8 or even more than 20, it would be very difficult to determine which locale is for each text attribute to obtain the text in the corresponding language. Complicated things. Also, under normal circumstances, the translator is not a developer. Can the translation be saved as an arb file like the i18n or l10n standard and handed over to the translator for translation. After the translation is completed, the developer can use the tool to convert the arb file to Code. The answer is yes! We will introduce in the next section how to achieve this through the Dart intl package.