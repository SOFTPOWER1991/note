本文主要复习如下内容：

1. 实际开发中启动模式使用不当造成的问题
2. Android中的启动模式分类以及他们各自的特点
3. 启动模式的应用场景

# 1. 实际开发中启动模式使用不当造成的问题

情景一：打开微信，定位到tab为我的栏目，在：钱包、收藏、相册、卡包、表情、设置这些栏目中快速连续点击几次，发现会启动两个或者3个界面，这就是启动模式使用的一个问题：**启动模式使用不当造成页面打开好几个，如果是一个支付页面，当用户支付完成后点击关闭，发现又有一个支付页面这就很尴尬了！**

情境二：我们App推送的通知，有时候会启动主页面，但此时如果已经有一个主页面存在的情况下，点击通知栏通知又启动一个主页面，也会非常尴尬。这也是启动模式使用的一个问题：**启动多个重复页面**

而这些问题的解决方案，就是使用正确的启动模式：LaunchMode。下面看看具体的LaunchMode分类：

# 2. Android中的启动模式分类以及他们各自的特点

Android系统为我们提供了四种启动模式：standard、SingleTop、singleTask , singleInstance。

具体如下：

* standard，标准模式，系统默认的启动模式。 
> 特点：每次启动一个Activity都会重新创建一个新的实例，不管这个实例是否已经存在。

* singleTop，栈顶复用模式。
> 特点：如果新Activity已经位于栈顶，那么此Activity不会被重新创建，它的onNewIntent会被调用。如果新的Activity实例已存在但不在栈顶，那么新activity会重新创建。

* singleTask，栈内复用模式。
> 只要Activity在一个栈中存在，那么多次启动此Activity就不会重新创建实例，系统会回调他的onNewIntent ，并附带 clearTop的效果。具体的来讲：当一个具有SingleTask模式Activity请求启动后，系统首先会寻找是否有Activity A想要的任务栈，如果不存在，就重新创建一个任务栈，把A入栈。如果存在A所需的任务栈，看栈中是否有A实例，如果有，就把A调到栈顶并调它的onNewIntent方法，如果实例不存在，就创建A的实例并把A压入栈中。

* singleInstance，单实例模式。
> 一个Activity在一个单独的任务栈中。

上文中提到了Activity多需的任务栈，那什么是Activity所需的任务栈呢？
> 这就要说到：TaskAffinity了。TaskAffinity标识了一个Activity所需的任务栈的名字。默认，所有Activity的任务栈的名字都是应用包名。我们可以给每个Activity单独指定TaskAffinity来指定另外的任务栈，名字不能和包名相同。
> 
> TaskAffinity属性主要和SingleTask 和 allowTaskReparenting属性配对使用。

任务栈分为：前台任务栈和后台任务栈，后台任务栈中的Activity处于暂停状态，用户通过切换将后台任务栈再次调到前台。

当TaskAffinity和singleTask启动模式配对使用的时候，它是具有该模式的Activity的目前任务栈的名字，待启动的Activity会运行在名字和TaskAffinity相同的任务栈中。


注意: 

1.	设置了"singleTask"启动模式的Activity，它在启动的时候，会先在系统中查找属性值affinity等于它的属性值taskAffinity的Task存在； 如果存在这样的Task，它就会在这个Task中启动，否则就会在新的任务栈中启动。因此， 如果我们想要设置了"singleTask"启动模式的Activity在新的任务中启动，就要为它设置一个独立的taskAffinity属性值。
2.	如果设置了"singleTask"启动模式的Activity不是在新的任务中启动时，它会在已有的任务中查看是否已经存在相应的Activity实例， 如果存在，就会把位于这个Activity实例上面的Activity全部结束掉，即最终这个Activity 实例会位于任务的Stack顶端中。
3.	在一个任务栈中只有一个”singleTask”启动模式的Activity存在。他的上面可以有其他的Activity。这点与singleInstance是有区别的。

# 3. 启动模式的使用场景

* singleTop适合接收通知启动的内容显示页面。 
> 例如，某个新闻客户端的新闻内容页面，如果收到10个新闻推送，每次都打开一个新闻内容页面是很烦人的。 

* singleTask适合作为程序入口点。 
> 例如浏览器的主界面。不管从多少个应用启动浏览器，只会启动主界面一次，其余情况都会走onNewIntent，并且会清空主界面上面的其他页面。 

* singleInstance应用场景： 
> 闹铃的响铃界面。 你以前设置了一个闹铃：上午6点。在上午5点58分，你启动了闹铃设置界面，并按 Home 键回桌面；在上午5点59分时，你在微信和朋友聊天；在6点时，闹铃响了，并且弹出了一个对话框形式的 Activity(名为 AlarmAlertActivity) 提示你到6点了(这个 Activity 就是以 SingleInstance 加载模式打开的)，你按返回键，回到的是微信的聊天界面，这是因为 AlarmAlertActivity 所在的 Task 的栈只有他一个元素， 因此退出之后这个 Task 的栈空了。如果是以 SingleTask 打开 AlarmAlertActivity，那么当闹铃响了的时候，按返回键应该进入闹铃设置界面。 

以上便是LaunchMode的相关内容。

