---
created: 2026-03-17 08:06
updated: 2026-03-17T20:56
status: Completed
topics: Thread model ; Message passing
---
obsidian://web-open?url=<https://ocw.mit.edu/ans7870/6/6.005/s16/classes/22-queues/>
## Two models for concurrency

前面[[Reading19 Cocurrency#Two Models for Concurrent Programming]]中提到两种并发模型，即共享内存模型和消息传递模型。上一章[[Reading21 Sockets & Networking]]中的服务器-客户端模型就是一种消息传递模型，通过socket传递消息。

消息传递模型更安全，这是因为：
1. 消息传递模型显式传递信息，而共享内存模型隐式修改，容易造成难以溯源的bug
2. 消息传递模型传递不可变信息，而共享内存需要使用可变类型，可变类型更危险

本章中使用阻塞队列实现进程内部多线程的消息传递模型

## Message passing with threads

使用同步队列进行线程间的消息传递，java中带阻塞的队列为BlockingQueue接口，继承自Queue，额外支持在队列满时插入阻塞，和队列空时取出阻塞。需要注意，阻塞队列的入队和出队为put和take，与队列的add和remove不同。

BlockingQueue有两种实现类，[ArrayBlockingQueue](https://docs.oracle.com/javase/8/docs/api/?java/util/concurrent/ArrayBlockingQueue.html)和[LinkedBlockingQueue](https://docs.oracle.com/javase/8/docs/api/?java/util/concurrent/LinkedBlockingQueue.html)，分别基于定长数组和链表实现，所以前者put会阻塞，后者不会

套接字只能收发字节流，而同步队列可以持有任何类型的对象，并且不需要设计网络协议（~~废话，根本不联网~~）。不过同步队列中的对象类型，必须为不可变类型，而且线程安全。

### Bank account example

这里以银行账户为例子，每个自动取款机和账户都是独立模块，模块之间通过同步队列通信。

自动取款机中设计了get-balance和withdraw方法，其中withdraw方法在取款前会检查账户余额，但是仍然可能出现交错两个自动取款机同时取同一个账户，出现竞争条件。所以withdraw方法需要在底层上设计为一个原子操作。

## Implementing message passing with queues

这里举了一个传递平方数的实例

这是一个用于平方整数的消息传递模块，有输入队列和输出队列，对于一个Squarer线程，从阻塞队列中取出整数，将输入的整数平方后，将该整数和平方数一起放入输入队列
```java
/** Squares integers. */
public class Squarer {

    private final BlockingQueue<Integer> in;
    private final BlockingQueue<SquareResult> out;
    // Rep invariant: in, out != null

    /** Make a new squarer.
     *  @param requests queue to receive requests from
     *  @param replies queue to send replies to */
    public Squarer(BlockingQueue<Integer> requests,
                   BlockingQueue<SquareResult> replies) {
        this.in = requests;
        this.out = replies;
    }

    /** Start handling squaring requests. */
    public void start() {
        new Thread(new Runnable() {
            public void run() {
                while (true) {
                    // TODO: we may want a way to stop the thread
                    try {
                        // block until a request arrives
                        int x = in.take();
                        // compute the answer and send it back
                        int y = x * x;
                        out.put(new SquareResult(x, y));
                    } catch (InterruptedException ie) {
                        ie.printStackTrace();
                    }
                }
            }
        }).start();
    }
}
```

输出消息是一个SquareResult类型对象，为不可变类型
```java
/** An immutable squaring result message. */
public class SquareResult {
    private final int input;
    private final int output;

    /** Make a new result message.
     *  @param input input number
     *  @param output square of input */
    public SquareResult(int input, int output) {
        this.input = input;
        this.output = output;
    }

    @Override public String toString() {
        return input + "^2 = " + output;
    }
}
```

这是调用Squarer的主方法
```java
public static void main(String[] args) {

    BlockingQueue<Integer> requests = new LinkedBlockingQueue<>();
    BlockingQueue<SquareResult> replies = new LinkedBlockingQueue<>();

    Squarer squarer = new Squarer(requests, replies);
    squarer.start();

    try {
        // make a request
        requests.put(42);
        // ... maybe do something concurrently ...
        // read the reply
        System.out.println(replies.take());
    } catch (InterruptedException ie) {
        ie.printStackTrace();
    }
}
```
## Stopping

如果想要关闭Squarer使其不再接受新的输入，单单退出一个线程是不够的，应该使所有Squarer线程退出。

一种策略是使用poison pill，即输入一个特殊值，然后在start方法中判断输入值是否为特殊值，这种方式是可以接受的，但是特殊值不一定方便取。

另一种策略是使用抽象数据类型，即将输入队列中的元素分为正常整数类型和stop类型，接口中具有input操作和shouldStop操作

## Thread safety arguments with message passing

基于消息传递的线程安全论证的几个方面：
1. 同步队列必须是并发安全的
2. 消息/数据类型应该是不可变的
3. 数据应该被限制在单个线程中，只有通过队列的消息可以被共享
4. 不得不使用可变数据类型：在将可变数据传递给另一线程后立即丢弃所有引用