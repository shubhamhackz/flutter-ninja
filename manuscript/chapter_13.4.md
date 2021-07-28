# 13.4 Frequently Asked Questions about Internationalization

This section mainly answers common questions in internationalization.

### The default locale is wrong

Some Android and iOS devices purchased from some non-mainland licensed channels will have the default Locale not Chinese simplified. This is a normal phenomenon, but in order to prevent the Locale obtained by the device from being inconsistent with the actual region, all apps that support multiple languages ​​must provide an entrance to manually select the language.

### How to internationalize app titles

`MaterialApp`There is an `title`attribute to specify the title of the APP. In the Android system, the title of the APP will appear in the task manager. So it also needs to `title`be internationalized. But the problem is that many international configuration is in `MaterialApp`the set, we can not build `MaterialApp`through `Localizations.of`to get localized resources, such as:

``` dart 
MaterialApp(
 title: DemoLocalizations.of(context).title, //不能正常工作！
 localizationsDelegates: [
   // 本地化的代理类
   GlobalMaterialLocalizations.delegate,
   GlobalWidgetsLocalizations.delegate,
   DemoLocalizationsDelegate() // 设置Delegate
 ],
);

```

After running the above code, `DemoLocalizations.of(context).title`it is being given, because `Localizations.of`looks along widget from the current context to the top of the tree `DemoLocalizations`, but we are `MaterialApp`in complete set `DemoLocalizationsDelegate`after, in fact, `DemoLocalizations`in the current context of the sub-tree, it `DemoLocalizations.of(context)`will return null, Report an error. So how do we deal with this situation? It's actually very simple, we only need to set a `onGenerateTitle`callback:

``` dart 
MaterialApp(
 onGenerateTitle: (context){
   // 此时context在Localizations的子树中
   return DemoLocalizations.of(context).title;
 },
 localizationsDelegates: [
   DemoLocalizationsDelegate(),
   ...
 ],
);

```

### How to specify the same locale for English-speaking countries

There are many English-speaking countries, such as the United States, Britain, Australia, etc. Although these English-speaking countries speak English, there are some differences. If our APP only wants to provide one kind of English (such as American English) for all English-speaking countries to use, we can `localeListResolutionCallback`do compatibility in the previous introduction :

``` dart 
localeListResolutionCallback:
   (List<Locale> locales, Iterable<Locale> supportedLocales) {
 // 判断当前locale是否为英语系国家，如果是直接返回Locale('en', 'US')     
}
```
