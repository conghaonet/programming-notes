# Android实现非阻塞Socket通信

[参考链接](https://blog.csdn.net/lyklykkk/article/details/75968680)

前言：Android开发中可以使用Java API提供的Socket和ServerSocket类来实现Socket通信。但是，通过这两个类实现的Socket通信是阻塞式的，当程序执行输入/输出操作后，在这些操作返回之前会一直阻塞线程。当有大量任务需要处理时，这种方式会降低性能。在Java中提供了另一种NIO API，可以实现非阻塞的Socket通信，该NIO API主要提供了以下两种特殊类：Selector和SelectableChannel。

# Selector类
* Selector是SelectableChannel对象的多路复用器，可以同时监控多个SelectableChannel的IO状况，是非阻塞IO的核心。所有希望采用非阻塞方式进行通信的Channel都应该注册到Selector对象。可通过调用Selector类的静态open()方法来创建Selector实例。一个Selector实例包含3个SelectionKey集合：
  * 所有SelectionKey集合：通过keys()方法返回，表示注册在该Selector上的所有Channel。
  * 被选择的SelectionKey集合：通过selectedKeys()返回，表示所有可通过select()方法监测到、需要进行IO处理的Channel。
  * 被取消的SelectionKey集合：表示所有被取消注册关系的Channel，在下一次执行select()方法时，这些Channel对应的SelectionKey会被彻底删除。程序通常无需直接访问该集合。
* 除了3个SelectionKey集合，Selector还提供了和select()相关的方法：
  * int select()：监控所有注册的Channel，当有Channel需要处理IO操作时，该方法返回，并将对应的SelectionKey加入被选择的SelectionKey集合中。该方法返回这些Channel的数量。
  * int select(long timeout)：可以设置超时时长的select()操作。
  * int selectNow()：执行一个立即返回的select()操作，相对于无参数的select()方法，该方法不会阻塞线程。
  * Selector wakeup()：使一个还未返回的select()方法立即返回。
    
## SelectableChannel类
* SelectableChannel代表了可以支持非阻塞IO操作的Channel对象，可以将其注册到Selector上，这种注册的关系由SelectionKey实例表示。Java中可以调用SelectableChannel实例的register()方法将其注册到指定的Selector上。当注册到Selector中的Channel当中有需要处理IO操作时，可以调用Selector的select()方法获取它们的数量，并通过selectedKeys()方法返回它们对应的SelectionKey集合。通过这个集合，可以获取所需要处理IO操作的SelectableChannel集。
* SelectableChannel支持阻塞和非阻塞两种模式，默认是阻塞的，必须通过非阻塞模式才能实现非阻塞IO操作。可以通过以下两个方法来设置和返回Channel的模式：
  * SelectionKey configureBlocking(boolean block)：设置是否采用阻塞模式
  * boolean isBlocking()：返回该Channel是否阻塞模式。
* 不同的SelectableChannel所支持的操作不一样。在SelectableChannel中提供了如下方法来返回不同SelectableSocketChannel所支持的操作：
  * int validOps()：返回一个bit mask，表示这个Channel上支持的IO操作。
    此外，SelectableChannel还提供了如下方法获取它的注册状态：
  * boolean isRegistered()：返回该Channel是否已注册在一个或多个Selector上。
  * selectionKey keyFor(Selector sel)：返回该Channel和Selector之间的注册关系，如不存在注册关系，则返回null。
* 下面介绍两种SelectableSocketChannel，分别是对应于java.net.ServerSocket的ServerSocketChannel和对应于java.net.Socket的SocketChannel。
