+++
author = "兔楠"
title = "基于TCP协议的简易通讯程序"
date = "2021-06-02"
description = "testing..."
tags = [
    "Java","网络编程","TCP"
]
categories = [
    "小玩意"
]

+++

Java课网络编程部分的一个小练习，用Socket类和ServerSocket类进行TCP编程，做一个简易的带图形界面的聊天工具。

<!--more-->

Java课教的比较简单，风格是那种什么都讲一点，又什么都讲的很浅的那种。不过老师写了这本教材还是很不容易的，入门的初初学者可以用来当字典型的书了。

书上给了个Server、Client对话程序的例子，不过是很简单的在命令行显示，然后用终端输入来卡循环，所以只能一人发一句。

<img src="../FakeQQPic/1.png" alt="1" style="zoom:75%;" />

<img src="../FakeQQPic/2.png" alt="2" style="zoom:75%;" />



## 课本源码

```java
// 客户端程序
package sample;
import java.io.*;
import java.net.*;
public class TalkClient {
    public static void main(String args[]) {
      try{
        Socket socket=new Socket("127.0.0.1",4700);
        //向本机的4700端口发出客户请求
        BufferedReader sin=new BufferedReader(new InputStreamReader(System.in));
        //由系统标准输入设备构造BufferedReader对象
        PrintWriter os=new PrintWriter(socket.getOutputStream());
        //由Socket对象得到输出流，并构造PrintWriter对象
        BufferedReader is=new BufferedReader(
new InputStreamReader(socket.getInputStream()));
        //由Socket对象得到输入流，并构造相应的BufferedReader对象        String readline;
        readline=sin.readLine(); //从系统标准输入读入一字符串
        while(!readline.equals("bye")){
        //若从标准输入读入的字符串为 "bye"则停止循环
          os.println(readline);
          //将从系统标准输入读入的字符串输出到Server
          os.flush();
          //刷新输出流，使Server马上收到该字符串
          System.out.println("Client:"+readline);
          //在系统标准输出上打印读入的字符串
          System.out.println("Server:"+is.readLine());
          //从Server读入一字符串，并打印到标准输出上
          readline=sin.readLine(); //从系统标准输入读入一字符串
        }
        os.close(); //关闭Socket输出流
        is.close(); //关闭Socket输入流
        socket.close(); //关闭Socket
      }catch(Exception e) {
        System.out.println("Error"+e); //出错，则打印出错信息
      }
  }
}
// 服务器端程序
package sample;
import java.io.*;
import java.net.*;
import java.applet.Applet;
public class TalkServer{
    public static void main(String args[]) {
      try{
        ServerSocket server=null;
        try{
          //新建4700端口的服务端
          server=new ServerSocket(4700);
}catch(Exception e) {
          System.out.println("can not listen to:"+e);
        //出现异常则返回信息
        }
        Socket socket=null;
        try{
          socket=server.accept();   //连接客户端
        }catch(Exception e) {
          System.out.println("Error."+e);  //出现异常返回信息
                 }
        String line;
        BufferedReader is=new BufferedReader(
new InputStreamReader(socket.getInputStream()));
         //由系统标准输入设备构造BufferedReader对象
        PrintWriter os=new PrintWriter(socket.getOutputStream());
         //由Socket对象得到输出流，并构造PrintWriter对象
        BufferedReader sin=new BufferedReader(new InputStreamReader(System.in));
         //由Socket对象得到输入流，并构造相应的BufferedReader对象
        System.out.println("Client:"+is.readLine());
        //从Client读入一字符串，并打印到标准输出上
        line=sin.readLine();
        //从系统标准输入读入字符串
        while(!line.equals("bye")){
        //若从标准输入读入的字符串为 "bye"则停止循环
          os.println(line);
          //将从系统标准输入读入的字符串输出到Client
          os.flush();
          //刷新输出流，使Server马上收到该字符串
          System.out.println("Server:"+line);
          //在系统标准输出上打印读入的字符串
          System.out.println("Client:"+is.readLine());
          //从Server读入一字符串，并打印到标准输出上
          line=sin.readLine();
          //从系统标准输入读入一字符串
        }
        os.close(); //关闭Socket输出流
        is.close(); //关闭Socket输入流
        socket.close(); //关闭Socket
        server.close(); //关闭ServerSocket
      }catch(Exception e){
        System.out.println("Error:"+e);
        //捕获异常，输出错误信息
      }
    }
  }
```



## 优化

我就简单改了下加了个线程，让它可以连续发送和接收。发送的线程50ms刷新一次，等发送信号（不设延时的话循环会跑的飞快，更改发送信号的线程没反应...）。接收消息的函数是会等待消息收到才会继续运行，就不用手动卡时间了。

图形界面是网上找的，删删改改了一下。

<img src="../FakeQQPic/3.png" alt="3" style="zoom:67%;" />



注释非常详尽，直接看源码吧~

![16000b282dd7588d1ffd73ff3240922](../FakeQQPic/16000b282dd7588d1ffd73ff3240922.png)

精分现场简直笑死，再也不说编程没意思了hhhhh



## 代码

```java
import java.io.*;
import java.net.*;
import java.awt.*;
import java.awt.event.ActionEvent;
import java.awt.event.ActionListener;
import java.text.SimpleDateFormat;
import java.util.Date;
import javax.swing.*;
import java.util.*;

//图形界面
class Chat extends JFrame implements ActionListener {
  static boolean status = true;
  static boolean send = false;
  private JLabel label1, label2;
  private JTextField jTextField2; // 文本框
  private JButton button2, button3; // 按钮
  private JTextArea textArea; // 文本域
  private JPanel southPanel; // 面板
  // 日期格式化 ，后面接收消息方法receive会用到
  private static SimpleDateFormat sdf2 = new SimpleDateFormat("a HH:mm:ss");

  public Chat() {

    // 定义窗口宽高常量
    final int width = 500;
    final int height = 600;
    // 获取屏幕尺寸
    Dimension screenSize = Toolkit.getDefaultToolkit().getScreenSize();
    JFrame myJFrame = new JFrame("Fake QQ");
    // 设置窗口大小
    myJFrame.setSize(width, height);
    // 设置窗口居中显示
    myJFrame.setLocation(screenSize.width / 2 - width / 2, screenSize.height / 2 - height / 2);

    // 网格布局
    this.setLayout(new BorderLayout());
    label1 = new JLabel("SERVER", SwingConstants.RIGHT);	//窗口顶部名称
    JPanel innerPanelCenter = new JPanel();
    JPanel innerPanel = new JPanel();
    innerPanel.add(label1);
    innerPanelCenter.add(innerPanel);

    label2 = new JLabel("快开始聊天吧!");
    label2.setForeground(Color.red);
    label2.setBorder(BorderFactory.createTitledBorder("提示"));
    JPanel northPanel = new JPanel(new BorderLayout());
    northPanel.add(innerPanelCenter, BorderLayout.CENTER);
    northPanel.add(label2, BorderLayout.SOUTH);
    myJFrame.add(northPanel, BorderLayout.NORTH);

    textArea = new JTextArea();
    textArea.setLineWrap(true);
    textArea.setWrapStyleWord(true);
    textArea.setFont(new Font("幼圆", Font.PLAIN, 16));
    myJFrame.add(new JScrollPane(textArea), BorderLayout.CENTER);
    southPanel = new JPanel();
    southPanel.add(new JLabel());
    jTextField2 = new JTextField(30);
    southPanel.add(jTextField2);
    button2 = new JButton("发送");
    southPanel.add(button2);
    button3 = new JButton("退出");
    southPanel.add(button3);
    button2.addActionListener(this);
    button3.addActionListener(this);
    myJFrame.add(southPanel, BorderLayout.SOUTH);

    // 设置窗口可见
    myJFrame.setVisible(true);
  }

  public void actionPerformed(ActionEvent e) {
    if (e.getSource() == button2) {
      send(); // 发送
    }
    if (e.getSource() == button3) {
      status = false;
      System.exit(0); // 退出
    }
  }

  public void send() { // 设置发送信号
    send = true;
  }

  public void onscreen(String person, String str) { // 图形界面输出
    textArea.append((sdf2.format(new Date())) + "\n" + person + ": " + str + "\n\n");
  }

  public String getText() {
    String str = jTextField2.getText();
    jTextField2.setText(""); // 清空输入栏
    return str;
  }
}

// 服务端
public class TalkServer {
  public static void main(String args[]) {
    try {
      Chat chat = new Chat(); // 创建图形界面实例
      ServerSocket server = null; // 创建socket
      try {
        server = new ServerSocket(4701);
      } catch (Exception e) {
        System.out.println("can not listen to:" + e);
      }
      Socket socket = null;
      try {
        socket = server.accept(); // 等待客户端接入
      } catch (Exception e) {
        System.out.println("Error." + e);
      }
      PrintWriter os = new PrintWriter(socket.getOutputStream());
      BufferedReader is = new BufferedReader(new InputStreamReader(socket.getInputStream()));

      class Send extends Thread { // 发送线程
        public void run() {
          os.println("connected");
          os.flush();
          String str = null;
          while (Chat.status) { // status由图形界面关闭按钮控制
            System.out.println("waiting to send"); // 命令行输出状态
            while (!Chat.send) { // 等待发送信号
              try {
                Thread.sleep(50); // 每50ms刷新状态
              } catch (InterruptedException e) {
                e.printStackTrace();
              }
            }
            Chat.send = false; // 重置等待信号
            str = chat.getText(); // 获取发送框文本
            chat.onscreen("You", str); // 显示发送内容
            try {
              os.println(str); // 发送
              os.flush();
            } catch (Exception e) {
              System.out.println("Sending error." + e);
            }
          }
        }
      }
      Send SendingThread = new Send(); // 创建并启动线程
      SendingThread.start();

      // 接收设在主线程，**不知道为什么创建新线程is.readLine()会出现异常**
      String str;
      while (Chat.status) {
        System.out.println("waiting to receive"); // 命令行输出状态
        str = is.readLine(); // 等待接收
        System.out.println("Client:" + str); // 命令行显示接收内容
        chat.onscreen("Client", str); // 图形界面显示接收内容
      }

      os.close(); // 关闭Socket输出流
      is.close();// 关闭Socket输入流
      socket.close(); // 关闭Socket
      server.close();
    } catch (IOException e) {
      e.printStackTrace();
      System.out.println("error");
    }
  }
}
// 客户端
public class TalkClient {
  public static void main(String args[]) {
    try {
      Chat chat = new Chat(); // 创建图形界面实例
      System.out.println("111");
      Socket socket = new Socket("127.0.0.1", 4701); // 创建socket，接入终端
      PrintWriter os = new PrintWriter(socket.getOutputStream());
      BufferedReader is = new BufferedReader(new InputStreamReader(socket.getInputStream()));

      class Send extends Thread { // 发送线程
        public void run() {
          os.println("connected");
          os.flush();
          String str = null;
          while (Chat.status) { // status由图形界面关闭按钮控制
            System.out.println("waiting to send"); // 命令行输出状态
            while (!Chat.send) { // 等待发送信号
              try {
                Thread.sleep(50); // 每50ms刷新状态
              } catch (InterruptedException e) {
                e.printStackTrace();
              }
            }
            Chat.send = false; // 重置等待信号
            str = chat.getText(); // 获取发送框文本
            chat.onscreen("You", str); // 显示发送内容
            try {
              os.println(str); // 发送
              os.flush();
            } catch (Exception e) {
              System.out.println("Sending error." + e);
            }
          }
        }
      }
      Send SendingThread = new Send(); // 创建并启动线程
      SendingThread.start();

      // 接收设在主线程，**不知道为什么创建新线程is.readLine()会出现异常**
      String str;
      while (Chat.status) {
        System.out.println("waiting to Receive"); // 命令行输出状态
        str = is.readLine(); // 等待接收
        System.out.println("Server:" + str); // 显示接受内容
        chat.onscreen("Server", str); // 图形界面显示接收内容
      }

      os.close(); // 关闭Socket输出流
      is.close();// 关闭Socket输入流
      socket.close(); // 关闭Socket
    } catch (IOException e) {
      e.printStackTrace();
      System.out.println("error");
    }
  }
}
```



