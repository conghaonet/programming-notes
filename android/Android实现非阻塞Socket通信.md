# Android实现非阻塞Socket通信

[参考链接](https://blog.csdn.net/lyklykkk/article/details/75968680)

前言：Android开发中可以使用Java API提供的Socket和ServerSocket类来实现Socket通信。但是，通过这两个类实现的Socket通信是阻塞式的，当程序执行输入/输出操作后，在这些操作返回之前会一直阻塞线程。当有大量任务需要处理时，这种方式会降低性能。在Java中提供了另一种NIO API，可以实现非阻塞的Socket通信，该NIO API主要提供了以下两种特殊类：Selector和SelectableChannel。

## Selector类
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
SelectableChannel代表了可以支持非阻塞IO操作的Channel对象，可以将其注册到Selector上，这种注册的关系由SelectionKey实例表示。Java中可以调用SelectableChannel实例的register()方法将其注册到指定的Selector上。当注册到Selector中的Channel当中有需要处理IO操作时，可以调用Selector的select()方法获取它们的数量，并通过selectedKeys()方法返回它们对应的SelectionKey集合。通过这个集合，可以获取所需要处理IO操作的SelectableChannel集。
* SelectableChannel支持阻塞和非阻塞两种模式，默认是阻塞的，必须通过非阻塞模式才能实现非阻塞IO操作。可以通过以下两个方法来设置和返回Channel的模式：
  * SelectionKey configureBlocking(boolean block)：设置是否采用阻塞模式
  * boolean isBlocking()：返回该Channel是否阻塞模式。
* 不同的SelectableChannel所支持的操作不一样。在SelectableChannel中提供了如下方法来返回不同SelectableSocketChannel所支持的操作：
  * int validOps()：返回一个bit mask，表示这个Channel上支持的IO操作。
    此外，SelectableChannel还提供了如下方法获取它的注册状态：
  * boolean isRegistered()：返回该Channel是否已注册在一个或多个Selector上。
  * selectionKey keyFor(Selector sel)：返回该Channel和Selector之间的注册关系，如不存在注册关系，则返回null。
* 下面介绍两种SelectableSocketChannel，分别是对应于java.net.ServerSocket的ServerSocketChannel和对应于java.net.Socket的SocketChannel。
  * **ServerSocketChannel类**    
      ServerSocketChannel代表一个ServerSocket，提供了一个TCP协议的IO接口，对应于java.net.ServerSocket，支持非阻塞模式。它只支持OP_ACCEPT操作。同时，该类也提供了accept()方法，功能相当于ServerSocket的accept()方法。
  * **SocketChannel类**  
      SocketChannel对应于java.net.Socket，同样提供一个TCP协议的IO接口，支持非阻塞模式。它支持OP_CONNECT、OP_READ和OP_WRITE操作。此外，该类还实现了ByteChannel接口、ScatteringByteChannel接口和GatheringByteChannel接口，可以直接通过SocketChannel来读写ByteBuffer对象。

## 代码示例
服务器上所有的Channel都需要向Selector注册，包括ServerSocketChannel和SocketChannel。该Selector负责监控这些Channel的IO状态，当有Channel需要IO操作时，Selector的select()方法将会返回具有IO操作的Channel的数量。并且，可通过selectedKeys()方法获取这些Channel对应的SelectionKey集合，进而获取到具有IO操作的SelectableChannel集。下面以一个实例来说明Java中NIO Socket的使用。直接上代码加注释
* Server端（在Eclipse上）
```java
public class NIOSocketServer implements ActionListener {

   private String ip = "192.168.1.132";
   private Window mWindow;
   private JButton mBtnSend;
   private JButton mBtnSendAll;
   private JTextField mTextFiled;
   private JTextArea mTextArea;
   private JButton mBtnClear;
   // 用于检测所有Channel状态的Selector
   private Selector mSelector = null;
   // 定义实现编码、解码的字符集对象
   private Charset mCharset = Charset.forName("UTF-8");
   private SocketChannel mSocketChannel = null;
   // private static ExecutorService mThreadPool;
   // private static SocketRun mSocketRun;

   public NIOSocketServer() {
      mWindow = new Window("服务端");
      mBtnSend = mWindow.getSendButton();
      mBtnSend.setName("发送");
      mBtnSendAll = mWindow.getSendAllButton();
      mBtnSendAll.setName("群发");
      mBtnClear = mWindow.getClearButton();
      mBtnClear.setName("clear");
      mTextFiled = mWindow.getTextField();
      mTextArea = mWindow.getJTextArea();
      mBtnSend.addActionListener(this);
      mBtnSendAll.addActionListener(this);
      mBtnClear.addActionListener(this);
      // mSocketRun = new SocketRun();
      // mThreadPool = Executors.newCachedThreadPool();
   }

   public static void main(String[] args) throws IOException {
      System.out.println("服务端已启动...");
      new NIOSocketServer().init();
   }

   @Override
   public void actionPerformed(ActionEvent event) {
      JButton source = (JButton) event.getSource();
      String name = source.getName();
      if (mBtnSend.getName().equals(name)) { // 向单个客户端发送消息
         String content = mTextFiled.getText().toString();
         if (mSocketChannel != null) {
            try {
               mSocketChannel.write(mCharset.encode(content));
            } catch (IOException e) {
               e.printStackTrace();
            }
         }
      } else if (mBtnSendAll.getName().equals(name)) { // 向所有客户端发送消息
         String content = mTextFiled.getText().toString();
         sendToAll(content);
      } else if (mBtnClear.getName().equals(name)) {
         mTextArea.setText("");
      }
   }

   private void sendToAll(String message) {

      for (SelectionKey sk : mSelector.keys()) {
         Channel channel = sk.channel();
         if (channel instanceof SocketChannel) {
            SocketChannel dest = (SocketChannel) channel;
            if (dest.isOpen()) {
               try {
                  dest.write(mCharset.encode(message));
               } catch (IOException e) {
                  e.printStackTrace();
               }
            }
         }
      }
   }

   public void init() throws IOException {
      mSelector = Selector.open();
      // 通过open方法打开一个未绑定的ServerSocketChannel实例
      ServerSocketChannel serverSocketChannel = ServerSocketChannel.open();
      InetSocketAddress inetSocketAddress = new InetSocketAddress(ip, 30000);
      // 将该ServerSocketChannel绑定到指定的IP地址
      serverSocketChannel.socket().bind(inetSocketAddress);
      // 设置ServerSocket以非阻塞方式工作
      serverSocketChannel.configureBlocking(false);
      // 将serverSocketChannel注册到指定Selector对象
      serverSocketChannel.register(mSelector, SelectionKey.OP_ACCEPT);
      while (mSelector.select() > 0) {
         // 依次处理selector上的每个已选择的SelectionKey
         Set<SelectionKey> selectedKeys = mSelector.selectedKeys();
         //这里必须用iterator，如果用for遍历Set程序会报错
         Iterator<SelectionKey> iterator = selectedKeys.iterator();
         while (iterator.hasNext()) {
            SelectionKey sk = iterator.next();
            // 从selector上的已选择的SelectionKey集合中删除正在处理的SelectionKey
            iterator.remove();
            // 如果sk对应的通道包含客户端的连接请求
            if (sk.isAcceptable()) {
               // 调用accept方法接受连接，产生服务端对应的SocketChannel
               SocketChannel socketChannel = serverSocketChannel.accept();
               mTextArea.append("客户端接入，IP："+socketChannel.getLocalAddress()+"\n");
               // 设置采用非阻塞模式
               socketChannel.configureBlocking(false);
               mSocketChannel = socketChannel;
               // 将该SocketChannel也注册到selector
               socketChannel.register(mSelector, SelectionKey.OP_READ);
               // 将sk对应的Channel设置成准备接受其他请求
               sk.interestOps(SelectionKey.OP_ACCEPT);

            }
            // 如果sk对应的通道有数据需要读取
            if (sk.isReadable()) {
               // 获取该SelectionKey对应的Channel，该Channel中有可读的数据
               SocketChannel socketChannel = (SocketChannel) sk.channel();
               mSocketChannel = socketChannel;
               // 定义准备执行读取数据的ByteBuffer
               ByteBuffer buffer = ByteBuffer.allocate(1024);
               String content = "";
               // 开始读取数据
               try {
                  while (socketChannel.read(buffer) > 0) {
                     buffer.flip();
                     content += mCharset.decode(buffer);
                  }
                  if ("shutdown".equals(content)) {
                     sk.cancel();
                     if (sk.channel() != null) {
                        sk.channel().close();
                     }
                     sendToAll("reconnect");
                  } else {
                     // 打印从该sk对应的Channel里读取到的数据
                     mTextArea.append("来自客户端的消息：" + content + "\n");
                     // 将sk对应的Channel设置成准备下一次读取
                     sk.interestOps(SelectionKey.OP_READ);
                  }
               }
               // 如果捕捉到该sk对应的Channel出现了异常，即表明该Channel
               // 对应的Client出现了问题，所以从Selector中取消sk的注册
               catch (IOException ex) {
                  // 从Selector中删除指定的SelectionKey
                  sk.cancel();
                  if (sk.channel() != null) {
                     sk.channel().close();
                  }
                  sendToAll("reconnect");
               }
            }
         }
      }
   }
}
```
