想了好久终于想明白了，记下来呗！

先说下Android 消息机制涉及到的类：`Handler`、`Looper`、`MessageQueue`、`Message`、`ThreadLocal`。

我之前在想这个问题的时候想过的几个问题，这几个问题搞明白了，消息机制就搞定了：

> 1. 在哪儿发的消息？
> 2. 消息发到哪儿去了？
> 3. 消息最后怎么被处理的？
> 4. 怎么就把子线程发的消息变换到了主线程？
> 5. 哪儿来的Looper？
> 6. Looper的loop方法又是在哪儿调用的呢？
> 7. 上面各个类都是什么作用呢？
> 8. Handler在哪儿？
> 9. Looper在哪儿？
> 10. MessageQueue在哪儿？
> 11. ThreadLocal在哪儿？

下面就挨个把这些问题搞明白：

# 在哪儿发的消息

在子线程发出的消息handler.sendMessage(XXX)

# 消息发到哪儿去了

发到了MessageQueue中，一个由链表实现的消息队列中

# 消息最后怎么被处理的

消息被Handler的 disPatchMessage进行分发到handMessage中进行了处理

# 怎么把子线程中发的消息变换到了主线程

在子线程发出的消息，发到了MessageQueue，而MessageQueue是在Looper的构造函数中初始化的；而Looper是在主线程中的，总的来说，就是MessageQueue的功劳。

# 哪儿来的Looper

主线程中Looper是在ActivityTread创建的时候，Looper.preperMainLooper()创建的

# Looper的loop方法又是在哪儿调用的呢

在MessageQueue中有消息的时候，进行调用，一个典型的：生产者、消费者模型；

# 涉及到的类的作用：

* Handler，Android中的消息处理机制
* Looper，消息泵，用来从MessageQueue中读取消息
* MessageQueue，消息队列，内部用链表实现
* Message,主线程和子线程之间的通信载体
* ThreadLocal: 每一个线程可以直接把数据存取到ThreadLocal中，而数据时隔离的互不干扰，用作线程的数据隔离。

# Handler在哪儿

Handler在主线程中创建

# Looper在哪儿

在主线程中

# MessageQueue在哪儿

在主线程中，在Looper的构造方法中初始化

# ThreadLocal

在Looper中，用来存取Looper


