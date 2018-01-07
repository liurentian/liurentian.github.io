---
title: Android IPC机制学习笔记(一)
comments: true
date:  2017-10-09
categories: Android
tags: Android 进程间通信
img:
---

## **一、Android IPC简介**

　　IPC机制（inter-process-Communication），进程间通信或跨进程通信，指两个进程进行数据交换的过程。

　　IPC不是 Android 中独有的。在Android中最有特色的进程间通信方式就是 Binder 了，通过 Binder 可以轻松实现进程间通信。除了Binder，Android 还支持Socket，通过 Socket 也可以实现任意两个终端的通信。
<br>
## **二、Android 中的多进程模式**


　　IPC机制既然是进程间通信或跨进程通信，那么学习 IPC机制之前我们就先来看看Android 中的多进程模式。
<br>
**2.1、Android 开启多进程模式**

正常情况下，在 Android 中使用多进程的只有一种方法，就是给四大组件在 AndroidManifest中 指定 android：process 属性。其实还有另一种非常规的多进程方法，那就是通过 JNI 在 native 层去 fork 一个新的进程。但是这种方法比较特殊，也不是常用的创建多进程方法，所以暂不考虑。

在 AndroidManifest 中指定 android：process 属性有几种不同的方式:

```
不指定 android：process 属性 默认在默认进程（进程名为当前包名）下运行。

android:process=”:remote” , 组件运行在进程（进程名为 packageName:remote）下。

android:process=”com.lrt.chapter2.remote” ，组件运行在进程、（com.lrt.chapter2.remote）中。

```

**2.2、Android多进程模式的运行机制**

　　Android 为每个应用都分配了一个独立的虚拟机，或者说为每个进程都分配了一个独立的虚拟机。不同的虚拟机在内存上会有不同的地址，这就导致在不同的虚拟机中访问同一个类的对象就会产生多个副本，所以在多进程下无法直接操作静态变量。

　　线程同步机制完全失效。

　　SharedPreferences 的可靠性降低。因为 SharedPreferences 不支持两个进程同时去执行操作，否则会导致一定数据的丢失，这是因为 SharedPreferences 底层是通过读/写 xml 文件夹来实现的，并发读写都有可能出现问题。

　　Application 对多次创建。当一个组件跑在一个新的进程中时，由于系统要创建新的进程并分配独立的虚拟机，因此相当于系统又把这个应用重新启动了一边，即然启动了，那么自然会创建新的 Application。 这个问题可以这么理解： 运行在同进程的组件属于同一个虚拟机同一个 Application，运行在不同进程的组件属于两个不同的虚拟机和 Application。

## **三、IPC基础概念介绍**

{% blockquote %}
　IPC 中的基础概念主要包括三方面的内容：Serializable 接口、Parcelable 接口、Binder。 　Serializable　和 Parcelable 接口可以完成对象的序列化过程，当我们需要通过 Intent 和 Binder 传输数据时就需要使用 Serializable 和 Parcelable 接口。
{% endblockquote %}

**3.１、Serializable**

　　Serializable　是　java　提供的一个序列化接口，是一个空的接口。为对象提供标准的序列化和反序列化操作。

　　使用　Serializable 来实现对象的序列化非常简单，只要这个类实现Serializable 接口并声明一个serialVersionUID 即可。private static final long serialVersionUID =878458786668464L; 并且这个UID也不是必须的。 Serializable 的实现过程也很简单：如下:

```
//序列化过程
User user=new User();
ObjectOutputStream  out=new ObjectOutputStream(
new FileOutputStream(“cache.txt”));
out.writeObject(user);
out.close();

//反序列化过程
ObjectInputStream in=new ObjectInputStream(
new FileInputStream(“cache.txt”);
User user=(User) in.readObject();
in.close();
```


**3.２、Parcelable**

Parcelable 也是一个接口，只要实现这个接口，一个类的对象就可以实现序列化并可以通过　Intent 和 Binder 传递。
　　使用 Parcelable 序列化，要先实现 Parcelable 接口，然后要实现 Parcel 类的序列化、反序列化和内容描述几个方法。下图是一个基本用法:

![图片加载失败](1.jpg)

**3.3、Binder**

 Binder 是 Android 中的一个类，它实现了 IBinder 接口。从 IPC 角度来说，Binder 是 Android 中的一种跨进程通信方法，Binder 还可以理解为一种虚拟的物理设备，它的驱动设备是 /dev/binder，该通信方式在 Linux中 没有；从 AndroidFramwork 角度来说，Binder 是 ServiceManager 连接各种 Manager （ActivityManager、WindowManager等等）和相应 ManagerService 的桥梁；从 Android 应用层来说，Binder 是客户端和服务端进行通信的媒介，当 bindService 的时候，服务端会返回一个包含了服务端业务调用的 Binder 的对象，通过这个 Binder 对象，客户端就可以获取服务端提供的服务或数据，这里的服务包括普通服务和基于AIDL的服务。

在 Android 开发中，Binder 主要用于在 Service 中，包括 AIDL 和 Messenger，其中普通的 Service 中不涉及进程间通信，而 Messenger 的底层原理是 AIDL。 所以为了分析 Binder，需要新建一个AIDL示例。

