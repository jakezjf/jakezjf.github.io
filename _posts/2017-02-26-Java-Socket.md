---
layout:     post
title:      "Java Socket 实现客户端服务端交互"
subtitle:   "学习 Java Socket 的使用"
date:       2017-02-26
author:     "JianFeng"
header-img: ""
catalog: true
tags:
    - java
---

> 使用 Java Socket 实现客户端服务端交互



# Java Socket 

Java 在网络编程中占有很大优势，JDK 中 net 包封装了比较完善的网络编程工具类，例如：Socket、URL、



##### ServerSocket 类

从名字上看 ServerSocket 是 服务端的 Socket，





# Socket类

##### 构造方法

Socket 构造方法主要有如下几个：

```
//
public Socket() {setImpl();}
//绑定远程连接的网络地址以及端口号
public Socket(InetAddress address, int port){}
//绑定远程连接、本地的网络地址以及端口号
public Socket(InetAddress address, int port, InetAddress localAddr,int localPort)
//绑定远程连接的网络地址、端口号以及
private Socket(SocketAddress address, SocketAddress localAddr,boolean stream)


//

```





使用 Socket 实现客户端和服务端的交互需要进行以下步骤：

- 连接远程机器
- 发送数据
- 接收数据
- 关闭连接
- 绑定端口
- 监听入站数据
- 在绑定端口上监听来自其他远程机器的连接







# Socket 客户端和服务端交互

通过 Socket 可以实现客户端和服务端的相互交互



##### 服务端实现

- 创建 ServerSocker 实现接口的监听
- 创建字节流或字符流读取从 socket 中获取到的输入流
- 关闭流
- 关闭连接



```
public class ServerPerson implements Runnable {

    private ServerSocket client;

    private Socket socket;

    public ServerPerson(ServerSocket client){
        this.client = client;
    }

    @Override
    public void run() {
        if(client == null){
            return;
        }
        try {
            socket = client.accept();
            System.out.print("客户端：");
            BufferedReader br = new BufferedReader(
                    new InputStreamReader(socket.getInputStream()));
            String str = br.readLine();
            while(str!=null){
                System.out.println(str);
                str = br.readLine();
            }
            socket.close();
        } catch (IOException e) {
            e.printStackTrace();
        }
    }

}
```





##### 客户端实现

在服务端的代码中，我们可以看见服务端设置了 Socket 的监听接口，意味着要给每个监听接口设置一个端口号，服务主机可使用的端口个数影响建立 Socket 连接的个数。



客户端实现的步骤：

- 建立 Socket 连接，包括服务端的网络地址和端口号
- 创建输出流，写入数据
- 关闭流
- 关闭连接



```
public class ClientPerson implements Runnable{

    private Socket client;

    public ClientPerson(Socket client){
        this.client = client;
    }

    @Override
    public void run() {
        BufferedWriter bw = null;
        try {
            bw = new BufferedWriter(new OutputStreamWriter(client.getOutputStream()));
            Scanner in = new Scanner(System.in);
            String str = in.next();
            bw.write(str);
            bw.close();
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
} 
```

