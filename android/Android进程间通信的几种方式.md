# Android进程间通信的几种方式
## 概念
由于应用程序之间不能共享内存。在不同应用程序之间交互数据（跨进程通讯），在Android SDK中提供了4种用于跨进程通讯的方式。

这4种方式正好对应于android系统中4种应用程序组件：Activity、Content Provider、Broadcast和Service:
1. **Activity**，可以跨进程调用其他应用程序的Activity；
2. **ContentProvider**，可以跨进程访问其他应用程序中的数据（以Cursor对象形式返回）， 当然，也可以对其他应用程序的数据进行增、删、改操作；
3. **Broadcast**，可以向android系统中所有应用程序发送广播， 而需要跨进程通讯的应用程序可以监听这些广播；
4. **Service**，和Content Provider类似，也可以访问其他应用程序中的数据， 但不同的是，ContentProvider返回的是Cursor对象， 而Service返回的是Java对象，这种可以跨进程通讯的服务叫AIDL服务。

## 定义多进程
Android应用中使用多进程只有一个办法（用NDK的fork来做除外），就是在AndroidManifest.xml中声明组件时，用android:process属性来指定。

不知定process属性，则默认运行在主进程中，主进程名字为包名。

android:process = package:remote，将运行在package:remote进程中，属于全局进程，其他具有相同shareUID与签名的APP可以跑在这个进程中。

android:process = :remote ，将运行在默认包名:remote进程中，而且是APP的私有进程，不允许其他APP的组件来访问。

## 多进程引发的问题
静态成员和单例失效：每个进程保持各自的静态成员和单例，相互独立。

线程同步机制失效：每个进程有自己的线程锁。

SharedPreferences可靠性下降：不支持并发写，会出现脏数据。

Application多次创建：不同进程跑在不同虚拟机，每个虚拟机启动会创建自己的Application，自定义Application时生命周期会混乱。

综上，不同进程拥有各自独立的虚拟机，Application，内存空间，由此引发一系列问题。

## 进程间通信的四种方式
* ### Activity
Activity的跨进程访问与进程内访问略有不同。虽然它们都需要Intent对象，但跨进程访问并不需要指定Context对象和Activity的 Class对象，而需要指定的是要访问的Activity所对应的Action（一个字符串）。有些Activity还需要指定一个Uri（通过 Intent构造方法的第2个参数指定）。 
在android系统中有很多应用程序提供了可以跨进程访问的Activity，例如，下面的代码可以直接调用拨打电话的Activity。
``` java
Intent callIntent = new Intent(Intent.ACTION_CALL, Uri.parse("tel:12345678"); 
 startActivity(callIntent);
```

* ### Content Provider
Android应用程序可以使用文件或SqlLite数据库来存储数据。Content Provider提供了一种在多个应用程序之间数据共享的方式（跨进程共享数据）。应用程序可以利用Content Provider完成下面的工作 
1. 查询数据 
2. 修改数据 
3. 添加数据 
4. 删除数据 
虽然Content Provider也可以在同一个应用程序中被访问，但这么做并没有什么意义。Content Provider存在的目的向其他应用程序共享数据和允许其他应用程序对数据进行增、删、改操作。 
Android系统本身提供了很多Content Provider，例如，音频、视频、联系人信息等等。我们可以通过这些Content Provider获得相关信息的列表。这些列表数据将以Cursor对象返回。因此，从Content Provider返回的数据是二维表的形式。

* ### 广播（Broadcast）
广播是一种被动跨进程通讯的方式。当某个程序向系统发送广播时，其他的应用程序只能被动地接收广播数据。这就象电台进行广播一样，听众只能被动地收听，而不能主动与电台进行沟通。 
在应用程序中发送广播比较简单。只需要调用sendBroadcast方法即可。该方法需要一个Intent对象。通过Intent对象可以发送需要广播的数据。

* ### Service
**利用AIDL Service实现跨进程通信**

这是我个人比较推崇的方式，因为它相比Broadcast而言，虽然实现上稍微麻烦了一点，但是它的优势就是不会像广播那样在手机中的广播较多时会有明显的时延，甚至有广播发送不成功的情况出现。 
注意普通的Service并不能实现跨进程操作，实际上普通的Service和它所在的应用处于同一个进程中，而且它也不会专门开一条新的线程，因此如果在普通的Service中实现在耗时的任务，需要新开线程。 
要实现跨进程通信，需要借助AIDL(Android Interface Definition Language)。Android中的跨进程服务其实是采用C/S的架构，因而AIDL的目的就是实现通信接口。

客户端每调用一次Context.startService(Intent)，Service就会重新执行一次onStartCommand—->onStart；但是使用AIDL的话，绑定服务之后，不会重复执行onStart，显然后者的代价更小。 
Service：前台Service，像我们经常用的天气、音乐其实都利用了前台Service来实现

