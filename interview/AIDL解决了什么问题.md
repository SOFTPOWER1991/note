AIDL (Android Interface Definition Language)

安卓接口定义语言。


Android系统中的进程间不能共享内存，所以需要提供一种机制在不同进程之间进行数据通信。

远程调用：

RPC——Remote Procedure Call (Java服务器端的，远程调用)

Android中有IDL——AIDL （来公开自己的服务接口）

AIDL：可以理解为双方的一个协议。双方都要持有这份协议。xxx.aidl文件（安卓内部编译时候会将AIDL翻译生成一个xxx.java文件——即代理模式，它是使用代理类在客户端和实现端传递数据）。这些和Binder驱动有关。

 是一种IDL 语言，用于生成可以在Android设备上两个进程之间进行进程间通信(interprocess communication, IPC)的代码。如果在一个进程中（例如Activity）要调用另一个进程中（例如Service）对象的操作，就可以使用AIDL生成可序列化的参数。 AIDL IPC机制是面向接口的，像COM或Corba一样，但是更加轻量级。它是使用代理类在客户端和实现端传递数据。 

