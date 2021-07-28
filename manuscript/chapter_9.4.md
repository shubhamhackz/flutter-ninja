# 9.4 Hero animation

Hero refers to widgets that can "fly" between routes (pages). Simply put, Hero animation means that when routes are switched, there is a shared widget that can switch between old and new routes. Since the position and appearance of the shared widget on the new and old routing pages may be different, when the routing is switched, it will gradually transition from the old road to the specified location in the new routing, which will generate a Hero animation.

You may have seen hero animation many times. For example, a route displays a list of thumbnails of products for sale, and selecting an item will redirect it to a new route, which contains detailed information of the product and a "buy" button. Flutter picture in the "fly" from one to another route called route **hero animation** , although the same action is also sometimes referred to as **shared elements convert** . Let's use an example to experience hero animation.

> Why is this kind of flying shared component called a hero? There is a saying that the superman in American culture can fly. That is the great hero in American hearts, and the superhero in Marvel is basically All of them can fly, so the Flutter developers gave this "flying widget" a romantic name hero. Of course, this statement is not an official explanation, but it is very interesting.

#### Example

Suppose there are two routes A and B, and their content interaction is as follows:

A: Contains a user avatar, circle, click to jump to route B, you can view the big picture.

B: Display the original picture of the user's avatar, rectangular;

When jumping between the two routes of AB, the user avatar will gradually transition to the avatar of the target routing page. Next, let's take a look at the code first, and then analyze:

``` dart 
// 路由A
class HeroAnimationRoute extends StatelessWidget {
 @override
 Widget build(BuildContext context) {
   return Container(
     alignment: Alignment.topCenter,
     child: InkWell(
       child: Hero(
         tag: "avatar", //唯一标记，前后两个路由页Hero的tag必须相同
         child: ClipOval(
           child: Image.asset("images/avatar.png",
             width: 50.0,
           ),
         ),
       ),
       onTap: () {
         //打开B路由  
         Navigator.push(context, PageRouteBuilder(
             pageBuilder: (BuildContext context, Animation animation,
                 Animation secondaryAnimation) {
               return new FadeTransition(
                 opacity: animation,
                 child: Scaffold(
                   appBar: AppBar(
                     title: Text("原图"),
                   ),
                   body: HeroAnimationRouteB(),
                 ),
               );
             })
         );
       },
     ),
   );
 }
}

```

Route B:

``` dart 
class HeroAnimationRouteB extends StatelessWidget {
 @override
 Widget build(BuildContext context) {
   return Center(
     child: Hero(
         tag: "avatar", //唯一标记，前后两个路由页Hero的tag必须相同
         child: Image.asset("images/avatar.png"),
     ),
   );
 }
}

```

We can see that to implement Hero animation, you only need to `Hero`wrap the widget to be shared with components and provide an identical tag. The intermediate transition frames are automatically completed by Flutter Framework. It must be noted that the shared `Hero`tags of the front and back routing pages must be the same, and the corresponding relationship between the old and new routing page widgets is determined by the tag inside the Flutter Framework.

The principle of Hero animation is relatively simple. Flutter Framework knows the position and size of the shared elements in the new and old routing pages, so based on these two endpoints, the interpolation (intermediate state) during the transition can be calculated during the execution of the animation. Yes, we don't need to do these things ourselves, Flutter has already done it for us!