# 3.6 Radio switches and check boxes

The Material component library provides Material style radio switches `Switch`and check boxes `Checkbox`. Although they are inherited from `StatefulWidget`, they do not save the current selected state. The selected state is managed by the parent component. When `Switch`or `Checkbox`when clicked, will trigger their `onChanged`callback, we can handle selected to change the logic in this callback. Let's look at a simple example:

``` dart 
class SwitchAndCheckBoxTestRoute extends StatefulWidget {
 @override
 _SwitchAndCheckBoxTestRouteState createState() => new _SwitchAndCheckBoxTestRouteState();
}

class _SwitchAndCheckBoxTestRouteState extends State<SwitchAndCheckBoxTestRoute> {
 bool _switchSelected=true; //维护单选开关状态
 bool _checkboxSelected=true;//维护复选框状态
 @override
 Widget build(BuildContext context) {
   return Column(
     children: <Widget>[
       Switch(
         value: _switchSelected,//当前状态
         onChanged:(value){
           //重新构建页面  
           setState(() {
             _switchSelected=value;
           });
         },
       ),
       Checkbox(
         value: _checkboxSelected,
         activeColor: Colors.red, //选中时的颜色
         onChanged:(value){
           setState(() {
             _checkboxSelected=value;
           });
         } ,
       )
     ],
   );
 }
}

```

In the above code, due to the need to maintain the selected state of the `Switch`sum `Checkbox`, it is `SwitchAndCheckBoxTestRoute`inherited from `StatefulWidget`. In its `build`method, a `Switch`sum is constructed respectively `Checkbox`, and the initial state is the selected state. When the user clicks, the state is reversed, and then the call is called back to `setState()`notify the Flutter framework to rebuild the UI.

![Figure 3-23](https://pcdn.flutterchina.club/imgs/3-23.png)

### Properties and appearance

`Switch`And `Checkbox`attributes are relatively simple, readers can view the API documentation, they all have an `activeColor`attribute to set the color of the active state. As for the size, so far, `Checkbox`the size is fixed and cannot be customized, but `Switch`only the width can be defined, and the height is also fixed. It is worth mentioning that `Checkbox`there is an attribute `tristate`that indicates whether it is tri-state or not. Its default value is `false`that the Checkbox has two states, namely "selected" and "unselected", and the corresponding value value is `true`sum `false`. If the `tristate`value `true`is too long, the value of value will increase by a state `null`, and readers can understand by themselves.

### to sum up

Through `Switch`and `Checkbox`we can see that although they are associated with the state (checked or not), they do not maintain the state by themselves, but need the parent component to manage the state, and then when the user clicks, the parent is notified through an event components, this is reasonable, because `Switch`and `Checkbox`if you select this and related user data, user data, and these can not be their private status. When we customize components, we should also think about which state management method is most reasonable.