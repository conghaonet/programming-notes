# Flutter区别于其他技术的关键

* Flutter采用自带的Native渲染引擎渲染视图，它是自己完成了组件渲染的闭环；而RN、Weex之类的框架，只是通过JavaScript虚拟机扩展调用系统组件，最后是由Android或者iOS系统来完成组件的渲染。
* Flutter的UI线程使用Dart来构建视图结构数据，这些数据会在GPU线程进行图层合成，随后交给Skia引擎加工成GPU数据，而这些数据会通过OpenGL最终提供给GPU渲染。
* Skia是Flutter的底层图像渲染引擎。  
  * Skia已然是Android官方的图像渲染引擎了，因此Flutter Android SDK无需内嵌Skia引擎就可以获得天然的Skia支持；而对于iOS平台来说，由于Skia是跨平台的，因此它作为Flutter 的iOS渲染引擎被嵌入到了Flutter iOS SDK中，代替了iOS闭源的Core Graphics/Core Animation/Core Text，这也正是Flutter iOS SDK打包的APP包体积比Android要大一些的原因。  
  * 底层渲染能力统一了，上层开发接口和功能体验也就随即统一了，开发者再也不用担心平台相关的渲染特性了。也就是说，Skia保证了同一台代码调用在Android和iOS平台上的渲染效果是完全一致的。
* 为什么是Dart？  
Dart因为同时支持JIT(JUST-IN-TIME)和AOT(Ahead-Of-Time)，所以既开发效率高，又运行速度好、执行性能高。在开发期选择JIT，开发调试异常方便（热重载）；在发布期使用AOT，本地代码的执行性能更加高效。

## Flutter原理
* Flutter的架构图：
<image src="https://ask.qcloudimg.com/http-save/yehe-4984806/3hj8g8xrsj.jpeg?imageView2/2/w/1620">

* Flutter 架构采用分层设计，从下到上分为三层，依次为：Embedder、Engine和Framework
  * Embedder是操作系统适配层，实现了渲染Surface设置，线程设置，以及平台插件等平台相关特性的适配。从这里我们可以看到Flutter平台相关特性并不多，这就使得从框架层面保持跨端一致性的成本相对较低。
  * Engine层主要包含Skia、Dart和Text，实现了Flutter的渲染引擎、文字排版、事件处理和Dart运行时等功能。Skia和Text为上层接口提供了调用底层渲染和排版的能力，Dart则为Flutter提供了运行时调用Dart和渲染引擎的能力。而Engine层的作用，则是将他们组合起来，从他们生成的数据中实现视图渲染。
  * Framework层则是一个用Dart实现的UI SDK，包含了动画、图形绘制和手势识别等功能。为了在绘制控件等固定样式的图形时提供更直观更方便的接口，Flutter还基于这些基本能力，根据Material和Cupertino两种视觉设计风格封装了一套UI组件库。我们在开发Flutter的时候，可以直接使用这些组件库。
  
## 布局
Flutter采用深度优先机制遍历渲染对象树，决定渲染对象树中各渲染对象在屏幕上的位置和尺寸。在布局过程中，渲染对象树中的每个渲染对象都会接收父对象的布局约束参数，决定自己的大小；然后父对象按照控件逻辑决定各个子对象的位置，完成布局过程。如下图所示
 
<image src="https://ask.qcloudimg.com/http-save/yehe-4984806/fzbm29uz9l.jpeg?imageView2/2/w/1620">
为了防止因子节点发生变化而导致整个控件树重新布局，Flutter加入了一个新的机制——布局边界（Relayout Boundary），可以在某些节点自动或手动地设置布局边界，当边界内的任何对象发生重新布局时，不会影响边界外的对象，反之亦然。如下图：
<image src="https://ask.qcloudimg.com/http-save/yehe-4984806/w4ovjmpuqh.jpeg?imageView2/2/w/1620">
  
## 绘制
布局完成以后，渲染对象树中的每个节点都有了明确的尺寸和位置。Flutter会把所有的渲染对象，绘制到不同的图层上。与布局过程一样，绘制过程也是深度优先遍历，而且总是先绘制自身，再绘制子节点。
以下图为例，节点1在绘制完自身后，会再绘制节点2，然后绘制子节点3、4和5，最后绘制节点6。
<image src="https://ask.qcloudimg.com/http-save/yehe-4984806/wgaxa6i6tr.jpeg?imageView2/2/w/1620">
可以看到，由于一些其他原因（比如，视图手动合并）导致2的子节点5与它的兄弟节点6处于了同一层，这样会导致当节点2需要重绘的时候，与它无关的节点6也会被重绘，带来性能损耗。

为了解决这一问题，Flutter提出了与布局边界对应的机制——重绘边界（Repaint Boundary）。在重绘边界内，Flutter会强制切换新的图层，这样就可以避免边界内外的互相影响，避免无关内容置于同一图层引起不必要的重绘。
<image src="https://ask.qcloudimg.com/http-save/yehe-4984806/2p3dnw909o.jpeg?imageView2/2/w/1620">
重绘边界的一个典型场景是ScrollView。ScrollView滚动的时候需要刷新视图内容，从而触发内容重绘。而当滚动内容重绘时，一般情况下其他内容是不需要重绘的，这时候重绘边界就派上用场了。

## 小结
Skia和Dart是构建Flutter底层的关键技术，也是Flutter区别于其他跨平台方案的核心所在。

跨平台方案的局限就是真正的多端一致性很难完全保证。RN这种就不用说了，很多组件的表现行为两端都不一样。就连Flutter也只能做到渲染层以上的多端一致性，还有一些原生的东西（比如Push、地图、定位、蓝牙、WebView）绕不开，需要通过在原生上写插件来搞定。不过话说回来，如果真的绕开了，那Flutter就变成操作系统了，打出来的包没个几百兆估计是搞不定的。

Flutter指知识体系导图：    
<image src="https://ask.qcloudimg.com/http-save/yehe-4984806/iyi15cv1ue.jpeg?imageView2/2/w/1620">

FLutter常用对象的继承关系：  
<image src="https://upload-images.jianshu.io/upload_images/4185621-fe1e969c84d57481.png">

Widget的生命周期：  
<image src="https://upload-images.jianshu.io/upload_images/4185621-77330034d352258b.png?imageMogr2/auto-orient/strip|imageView2/2/w/1200/format/webp">

