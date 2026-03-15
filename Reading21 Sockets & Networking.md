---
created: 2026-03-13 10:36
updated: 2026-03-15T22:58
status: Completed
topics: concepts of socket ; I/O stream ; Wire protocols ; Networking
---
>[!SUMMARY] Table of Contents
>    - [[Reading21 Sockets & Networking#Client/server design pattern|Client/server design pattern]]
>    - [[Reading21 Sockets & Networking#Network sockets|Network sockets]]
>        - [[Reading21 Sockets & Networking#IP addresses|IP addresses]]
>        - [[Reading21 Sockets & Networking#Hostnames|Hostnames]]
>        - [[Reading21 Sockets & Networking#Port numbers|Port numbers]]
>        - [[Reading21 Sockets & Networking#Network sockets|Network sockets]]
>    - [[Reading21 Sockets & Networking#I/O|I/O]]
>        - [[Reading21 Sockets & Networking#Buffers|Buffers]]
>        - [[Reading21 Sockets & Networking#Streams|Streams]]
>    - [[Reading21 Sockets & Networking#Blocking|Blocking]]
>    - [[Reading21 Sockets & Networking#Using network sockets|Using network sockets]]
>    - [[Reading21 Sockets & Networking#Wire protocols|Wire protocols]]
>    - [[Reading21 Sockets & Networking#Summary|Summary]]
## Client/server design pattern

客户端-服务器模式：客户端向服务器发出请求，服务器回复客户端。客户端可以连接多个服务器，服务器也可以处理多个客户端。

## Network sockets

### IP addresses

网络接口由ip地址标识，IPv4由四个八位部分组成。

例如18开头的都为MIT地址，127开头的都为回环地址

### Hostnames

主机名是可以被翻译为ip地址的名称。主机名可以映射到多个ip地址，多个主机名也可以映射到同一个ip地址。

例如：
```bash
$ dig +short localhost
127.0.0.1

$ dig +short web.mit.edu
www.mit.edu.edgekey.net.
e9566.dscb.akamaiedge.net.
104.115.233.29
```

将主机名翻译为ip地址是DNS(Domain name system)的工作，本章中没有展开DNS的原理，作以下拓展：

DNS的工作原理与有快表的内存管理结构很相似，当收到hostname时，
1. 先查浏览器和OS的缓存，如果有该主机名的访问记录，则直接使用对应ip
2. 在根服务器中找到顶级域名服务器(.com等)
3. 在顶级域名服务器下找权威域名服务器(域名真实所有者架构的服务器，存储最准确的ip)

### Port numbers

来自服务器的流量需要“门牌号”来分发流量到不同进程，而端口号就是这个门牌号

### Network sockets

Socket在java中就是一个通信对象，包含接受网络协议，远程端点，本地端点，状态，进程PID和缓冲区信息

以下是实时的部分socket信息：
```bash
camellia@LAPTOP-2AA7QA0V MINGW64 ~
$ netstat -ano | grep LISTENING
  TCP    10.135.5.39:56320      4.145.79.80:443        ESTABLISHED     28664
  TCP    10.135.5.39:58404      58.63.247.4:11001      ESTABLISHED     18744
  TCP    10.135.5.39:59245      58.63.247.4:11001      ESTABLISHED     18744
  TCP    10.135.5.39:59322      58.63.247.4:11001      TIME_WAIT       0
  TCP    10.135.5.39:59324      52.168.117.168:443     TIME_WAIT       0
  TCP    10.135.5.39:59378      58.63.247.4:11001      TIME_WAIT       0
  TCP    10.135.5.39:60389      48.210.190.78:443      ESTABLISHED     20776
  TCP    10.135.5.39:65372      58.63.247.4:11001      ESTABLISHED     18744
  TCP    127.0.0.1:4001         0.0.0.0:0              LISTENING       11448


```

从左到右依次为：网络协议，远程端点，本地端点，状态，进程PID

## I/O

### Buffers

在网络中交换的数据是以块为单位发送的，发送端写入一个较大的块，网络将其分解为较小的数据包，接收端将其重新组装为数据流。

由于数据的发送和接收往往是异步的，当接收端想要读取数据的时候，该数据可能早已抵达，也可能还没抵达，所以需要一个缓冲区来暂存数据。

### Streams

虽然底层中数据是被切分为一个个块来传输的，但是表面上是通过字节流来传输的

Java中，InputStream和OutputStream分别是规定输入、输出流的抽象类，其他所有流类型都是其子类

字节流、字符流、缓冲流、扫描和格式化、命令行I/O、数据流和对象流参考Oracle教程[I/O Streams](https://docs.oracle.com/javase/tutorial/essential/io/streams.html)

## Blocking

阻塞的概念见操作系统，socket的输入输出流造成的阻塞有两种原因：被读套接字的缓冲区为空、被写套接字的缓冲区已满

## Using network sockets

关于套接字的文档：[All About Sockets](https://docs.oracle.com/javase/tutorial/networking/sockets/index.html)

这里给出了一个代码实例，便于理解以上的所有内容：
```java
public class EchoServer {
    public static void main(String[] args) throws IOException {
         
        if (args.length != 1) {
            System.err.println("Usage: java EchoServer <port number>");
            System.exit(1);
        }
         
        int portNumber = Integer.parseInt(args[0]);
         
        try (
            ServerSocket serverSocket =
                new ServerSocket(Integer.parseInt(args[0]));
            Socket clientSocket = serverSocket.accept();     
            PrintWriter out =
                new PrintWriter(clientSocket.getOutputStream(), true);                   
            BufferedReader in = new BufferedReader(
                new InputStreamReader(clientSocket.getInputStream()));
        ) {
            String inputLine;
            while ((inputLine = in.readLine()) != null) {
                out.println(inputLine);
            }
        } catch (IOException e) {
            System.out.println("Exception caught when trying to listen on port "
                + portNumber + " or listening for a connection");
            System.out.println(e.getMessage());
        }
    }
}
```

EchoServer是一个回显服务器，将服务器传入的流原封不动传回去，有几点要注意：
1. try-with-resourses结构：socket和流对象都属于外部资源，如果没有正确关闭将会一直占据资源，而Java7之后的版本对于的try()部分创建的资源提供了自动关闭，当代码执行完或者触发异常的时候都会自动close这些资源
2. ServerSocket serverSocket：这是一个服务器socket，相当于接待员，监听某个端口
3. Socket clientSocket ：当有客户端socket连接serverSocket时，创建一个新socket与其对接，并让serverSocket等待下一个客户

```java
public class EchoClient {
    public static void main(String[] args) throws IOException {
         
        if (args.length != 2) {
            System.err.println(
                "Usage: java EchoClient <host name> <port number>");
            System.exit(1);
        }
 
        String hostName = args[0];
        int portNumber = Integer.parseInt(args[1]);
 
        try (
            Socket echoSocket = new Socket(hostName, portNumber);
            PrintWriter out =
                new PrintWriter(echoSocket.getOutputStream(), true);
            BufferedReader in =
                new BufferedReader(
                    new InputStreamReader(echoSocket.getInputStream()));
            BufferedReader stdIn =
                new BufferedReader(
                    new InputStreamReader(System.in))
        ) {
            String userInput;
            while ((userInput = stdIn.readLine()) != null) {
                out.println(userInput);
                System.out.println("echo: " + in.readLine());
            }
        } catch (UnknownHostException e) {
            System.err.println("Don't know about host " + hostName);
            System.exit(1);
        } catch (IOException e) {
            System.err.println("Couldn't get I/O for the connection to " +
                hostName);
            System.exit(1);
        } 
    }
}
```

EchoClient是一个客户端，通过主机名和端口号连接服务器，此时回显的流程为：
1. echoSocket连接serverSocket
2. serverSocket创造clientSocket和echoSocket对接，自己继续等待下一个连接申请
3. 通过stdIn接受用户输入
4. 将用户输入传给ehoSocket的输出流，输出给clientSocket
5. clientSocket接受到输出流，原封不动输出回echoSocket
6. echoSocket打印输入流

## Wire protocols

网络协议有三个要素：
1. 语法（Syntax）：接收者将01字节流翻译为指令的方式，包含分隔符、结束符和边界
2. 语义（Semantics）：规定每个信息代表的操作
3. 同步（Timing）：规定通信的先后顺序、状态同步和速率控制

试用了一下HTTP和SMTP，现在还不是很懂，具体后面再补充

## Summary

本章讲解了套接字和网络协议，socket可以理解为一个能够输入和输出的端点，两个socket通过ip地址和端口号相互连接之后就可以相互通信；网络协议规定了通信双方的交互方式，由于最底层是通过字节流传递信息，所以接收方要知道如何把01流翻译为指令，并知道指令对应是什么操作，以及应该按什么顺序对话，即语法、语义和同步