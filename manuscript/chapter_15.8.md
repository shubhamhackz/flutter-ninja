# 15.8 Multi-language and multi-theme

The language and theme in this example APP can be set, and both are achieved through `ChangeNotifierProvider`: we `main`use it in the function, `Consumer2`and rely on the `ThemeModel`sum `LocaleModel`, so when we change the current configuration on the language and theme setting page , `Consumer2`the `builder`re-executed, a new building `MaterialApp`, so the changes will take effect immediately. Let's take a look at the implementation of the language and theme setting page.

## 15.8.1 Language selection page

The APP language selection page provides three options: Simplified Chinese, American English, and Follow System. We highlight the language currently used by the APP, and add a "check mark" icon at the back to achieve the following:

```
class LanguageRoute extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    var color = Theme.of(context).primaryColor;
    var localeModel = Provider.of<LocaleModel>(context);
    var gm = GmLocalizations.of(context);
    //构建语言选择项
    Widget _buildLanguageItem(String lan, value) {
      return ListTile(
        title: Text(
          lan,
          // 对APP当前语言进行高亮显示
          style: TextStyle(color: localeModel.locale == value ? color : null),
        ),
        trailing:
            localeModel.locale == value ? Icon(Icons.done, color: color) : null,
        onTap: () {
          // 更新locale后MaterialApp会重新build
          localeModel.locale = value;
        },
      );
    }

    return Scaffold(
      appBar: AppBar(
        title: Text(gm.language),
      ),
      body: ListView(
        children: <Widget>[
          _buildLanguageItem("中文简体", "zh_CN"),
          _buildLanguageItem("English", "en_US"),
          _buildLanguageItem(gm.auto, null),
        ],
      ),
    );
  }
}

```

The above code logic is very simple, only caveat is that we `build(…)`define which method `_buildLanguageItem(…)`process it in `LanguageRoute`defined in the class distinction of this method is that: in the `build(…)`method of the definition can be shared by `build(...)`a method context variables, the present embodiment is shared Up `localeModel`. Of course, if `_buildLanguageItem(…)`the implementation is more complicated, it is not recommended to do so, at this time it is best to use it as a `LanguageRoute`class method. The running effect of this page is shown in Figure 15-6 and 15-7:

![15-6](https://pcdn.flutterchina.club/imgs/15-6.png)![15-7](https://pcdn.flutterchina.club/imgs/15-7.png)

It takes effect immediately after switching the language.

## 15.8.2 Theme selection page

A complete theme `Theme`includes many options, which `ThemeData`are defined in. For the sake of simplicity in this example, we only configure the theme color. We provide several default predefined theme colors for users to choose, and users will update the theme after clicking a color block. The implementation code of the theme selection page is as follows:

```
class ThemeChangeRoute extends StatelessWidget{
  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: Text(GmLocalizations.of(context).theme),
      ),
      body: ListView( //显示主题色块
        children: Global.themes.map<Widget>((e) {
          return GestureDetector(
            child: Padding(
              padding: const EdgeInsets.symmetric(vertical: 5, horizontal: 16),
              child: Container(
                color: e,
                height: 40,
              ),
            ),
            onTap: () {
              //主题更新后，MaterialApp会重新build
              Provider.of<ThemeModel>(context).theme = e;
            },
          );
        }).toList(),
      ),
    );
  }
}

```

The running effect is shown in Figure 15-8:

![15-8](https://pcdn.flutterchina.club/imgs/15-8.png)

After clicking on other theme color blocks, the APP theme color switch will take effect immediately.