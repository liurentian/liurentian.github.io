---
title: Android IPC机制学习笔记(二)
comments: true
date: 2017-10-19
categories: Android
tags: Android 进程间通信
img:
---
##  **前言**
{% blockquote %}
	前面已经简单了解了 Binder 的概念及用途。下面我们就下面两点来深入了解一下。
{% endblockquote %}

##  **目录**
{% blockquote %}
	1.通过 AIDL 分析 Binder

	2.手动实现 Binder
{% endblockquote %}


### **一、通过 AIDL 分析 Binder**

**首先新建一个 AIDL 示例**


**1、新建一个 Book.java 表示一个图书信息的类，实现 Parcelable 接口**


```
public class Book implements Parcelable {
    private int bookId;

    private String bookName;

    public Book(int bookId, String bookName) {
        this.bookId = bookId;
        this.bookName = bookName;
    }

    protected Book(Parcel in) {
        bookId = in.readInt();
        bookName = in.readString();
    }

    public static final Creator<Book> CREATOR = new Creator<Book>() {
        @Override
        public Book createFromParcel(Parcel in) {
            return new Book(in);
        }

        @Override
        public Book[] newArray(int size) {
            return new Book[size];
        }
    };

    @Override
    public int describeContents() {
        return 0;
    }

    @Override
    public void writeToParcel(Parcel parcel, int i) {
        parcel.writeInt(bookId);
        parcel.writeString(bookName);
    }
}

```

**2、创建 Book.aidl，Book.aidl 是 Book 类在 AIDL 中的声明。**

```
// Book.aidl.aidl
package com.ydsd.binderdemo.aidl;

parcelable Book;


```


**3、创建 IBookManager.aidl, IBookManager.aidl 是我们定义的一个接口，里面有两个方法：getBookList 和 addBook。**
```
// IBookManager.aidl
package com.ydsd.binderdemo.aidl;

// 注意即使 Book 跟 IBookmanager 在同一个包中，也需要引入。
import com.ydsd.binderdemo.aidl.Book;

interface IBookManager {
    List<Book> getBookList();
    void addBook(in Book book);
}

```

到此为止，AIDL就创建完成了，这时候我们用 AndroidStudio 编译项目， 编译过后，系统会自动给我们在 gen 目录下的 aidl 包中为 IBookManager.aidl 生成一个叫 IBookManager.java 的 Binder 类。这就是 AIDL 的特殊之处。

最后看一下项目目录:

![图片加载失败](1.png)

**4、编译项目, 找到并分析 IBookManager.java 类**



下面我们找到 gen 目录下的 aidl 包中系统为我们生成的的 IBookManager.java 类。来分析一下。

![图片加载失败](2.png)

```
public interface IBookManager extends android.os.IInterface {
    /**
     * Local-side IPC implementation stub class.
     */
    public static abstract class Stub extends android.os.Binder implements com.ydsd.binderdemo.aidl.IBookManager {
        private static final java.lang.String DESCRIPTOR = "com.ydsd.binderdemo.aidl.IBookManager";

        /**
         * Construct the stub at attach it to the interface.
         */
        public Stub() {
            this.attachInterface(this, DESCRIPTOR);
        }

        /**
         * Cast an IBinder object into an com.ydsd.binderdemo.aidl.IBookManager interface,
         * generating a proxy if needed.
         */
        public static com.ydsd.binderdemo.aidl.IBookManager asInterface(android.os.IBinder obj) {
            if ((obj == null)) {
                return null;
            }
            android.os.IInterface iin = obj.queryLocalInterface(DESCRIPTOR);
            if (((iin != null) && (iin instanceof com.ydsd.binderdemo.aidl.IBookManager))) {
                return ((com.ydsd.binderdemo.aidl.IBookManager) iin);
            }
            return new com.ydsd.binderdemo.aidl.IBookManager.Stub.Proxy(obj);
        }

        @Override
        public android.os.IBinder asBinder() {
            return this;
        }

        @Override
        public boolean onTransact(int code, android.os.Parcel data, android.os.Parcel reply, int flags) throws android.os.RemoteException {
            switch (code) {
                case INTERFACE_TRANSACTION: {
                    reply.writeString(DESCRIPTOR);
                    return true;
                }
                case TRANSACTION_getBookList: {
                    data.enforceInterface(DESCRIPTOR);
                    java.util.List<com.ydsd.binderdemo.aidl.Book> _result = this.getBookList();
                    reply.writeNoException();
                    reply.writeTypedList(_result);
                    return true;
                }
                case TRANSACTION_addBook: {
                    data.enforceInterface(DESCRIPTOR);
                    com.ydsd.binderdemo.aidl.Book _arg0;
                    if ((0 != data.readInt())) {
                        _arg0 = com.ydsd.binderdemo.aidl.Book.CREATOR.createFromParcel(data);
                    } else {
                        _arg0 = null;
                    }
                    this.addBook(_arg0);
                    reply.writeNoException();
                    return true;
                }
            }
            return super.onTransact(code, data, reply, flags);
        }

        private static class Proxy implements com.ydsd.binderdemo.aidl.IBookManager {
            private android.os.IBinder mRemote;

            Proxy(android.os.IBinder remote) {
                mRemote = remote;
            }

            @Override
            public android.os.IBinder asBinder() {
                return mRemote;
            }

            public java.lang.String getInterfaceDescriptor() {
                return DESCRIPTOR;
            }

            @Override
            public java.util.List<com.ydsd.binderdemo.aidl.Book> getBookList() throws android.os.RemoteException {
                android.os.Parcel _data = android.os.Parcel.obtain();
                android.os.Parcel _reply = android.os.Parcel.obtain();
                java.util.List<com.ydsd.binderdemo.aidl.Book> _result;
                try {
                    _data.writeInterfaceToken(DESCRIPTOR);
                    mRemote.transact(Stub.TRANSACTION_getBookList, _data, _reply, 0);
                    _reply.readException();
                    _result = _reply.createTypedArrayList(com.ydsd.binderdemo.aidl.Book.CREATOR);
                } finally {
                    _reply.recycle();
                    _data.recycle();
                }
                return _result;
            }

            @Override
            public void addBook(com.ydsd.binderdemo.aidl.Book book) throws android.os.RemoteException {
                android.os.Parcel _data = android.os.Parcel.obtain();
                android.os.Parcel _reply = android.os.Parcel.obtain();
                try {
                    _data.writeInterfaceToken(DESCRIPTOR);
                    if ((book != null)) {
                        _data.writeInt(1);
                        book.writeToParcel(_data, 0);
                    } else {
                        _data.writeInt(0);
                    }
                    mRemote.transact(Stub.TRANSACTION_addBook, _data, _reply, 0);
                    _reply.readException();
                } finally {
                    _reply.recycle();
                    _data.recycle();
                }
            }
        }

        static final int TRANSACTION_getBookList = (android.os.IBinder.FIRST_CALL_TRANSACTION + 0);
        static final int TRANSACTION_addBook = (android.os.IBinder.FIRST_CALL_TRANSACTION + 1);
    }

    public java.util.List<com.ydsd.binderdemo.aidl.Book> getBookList() throws android.os.RemoteException;

    public void addBook(com.ydsd.binderdemo.aidl.Book book) throws android.os.RemoteException;
}
```


可以看到系统根据IBookManager.aidl 系统给我们生成了 IBookManager.java 这个类。 

IBookManager.java 继承了 IInterface 接口，同时它自己还是个接口，所有可以在 Binder 中传输的类都要继承 IInterface 接口。

按结构来分析的话，首先它声明了两个方法 getList 和 addBook。 显然这就是我们在IBookManager.aidl 中声明的方法。接着它声明了两个整形的 id 用于分别标识这两个方法。这两个id用于标识在 transact 过程中客户端所请求的到底是哪个方法，接着它声明了一个stub 内部类，这个Stub就是一个 Binder 类，当客户端和服务端都位于同一进程中，方法调用不会走跨进程的 transact 过程，而当两者位于不同进程中，方法调用需要走 transact 过程，这个逻辑由 Stub 的内部代理类 Proxy 来完成。


这么来看 这个接口的核心实现就是它的内部类 Stub 和 Stub 的内部代理类 Proxy。

下面来详细看这两个类的方法含义。

* DESCRIPTOR: —-> Binder的唯一标识，一般用当前类名表示。

* asInterface(android.os.IBinder obj): —-> 用于将服务端的Binder 对象转换成客户端所需的AIDL 接口类型的对象。这种转换过程是区分进程的，如果客户端和服务端在统一进程，那么返回的就是 Stub 对象本身，否则返回的是系统封装好的 Stub.proxy 对象。

* asBinder: —-> 此方法用于返回当前Binder对象。

* onTransact: —-> 这个方法运行在服务端的线程池中，当客户端发起跨进程请求时，远程请求会通过系统底层封装后交由此方法来处理。该方法的原型为 public Boolean onTransact(int code,Parcel data,Parcel reply,int flags)。服务端通过code 可以确定客户端所请求的目标方法是什么，接着从data 里面取出目标方法所需的参数，然后执行目标方法。等目标方法执行完毕后，就向reply写入返回值。需要注意的是，如果此方法返回false，那么客户端的请求会失败，我们可以利用这个特性来做权限验证。

* Proxy# getBookList/addBook: —-> 这个方法运行在客户端，当客户端调用此方法时，它的内部是这样实现的： 首先创建该方法所需要的输入型Parcel 对象 _data、输出型Parcel 对象 __reply、和返回值对象List； 然后把该方法的参数信息写入_data中；接着调用transact方法来发起RPC（远程过程调用）请求，同时当前线程挂起；然后服务端的onTransact 方法会被调用，直到RPC过程返回后，当前线程继续执行，并从_reply 中取出RPC过程返回的结果；最后返回_reply 中的数据。

通过对上面方法的分析，我们应该已经了解 Binder 机制了。但是有两点需要注意：首先，当客户端发起远程请求时，由于当期线程会被挂起直至服务端进程返回数据，所以如果一个远程方法是很耗时的，那么不能在UI线程中发起此请求；其次，由于服务端的Binder 方法运行在Binder的线程池中，所以Binder方法不管是否耗时都应该采用同步的方法去实现，因为它已经运行在一个线程池中了。为了更好地说明Binder 下面有一个Binder的工作机制图。

![图片加载失败](4.jpg)

### **二、手动实现 Binder**
{% blockquote %}

从上面的分析过程来看，我们完全可以不使用AIDL 文件即可实现Binder，之所以提供AIDL文件，是为了方便系统为我们生成代码。系统根据AIDL 文件生成java 文件的格式是固定的，我们可以抛开AIDL文件直接写一个Binder出来。
{% endblockquote %}

　根据上面的思想，手动实现一个Binder可以通过以下步骤来完成：

**(1) 声明一个 AIDL 性质的接口，只需要继承 IInterface 接口即可，IInterface 接口中只有一个 asBinder 方法。代码如下所示：**

```
public interface IBookManager extends IInterface{
    static final String DESCRIPTOR ="com.aidldemo.IBookManager";

    static final int TRANSACTION_getBookList= IBinder.FIRST_CALL_TRANSACTION+0;

    static final int TRANSACTION_addBook=IBinder.FIRST_CALL_TRANSACTION+1;

    public List<Book> getBookList() throws RemoteException;

    public void addBook(Book book) throws RemoteException;

}
```

** (2) 实现Stub 类和Stub类中的Proxy代理类，这段代码只需要参考系统生成的代码即可，代码如下图所示：**

```

/**
 * IBookManager 的内部类 相当于 Stub
 */

public class IBookManagerImpl extends Binder implements IBookManager {

    public IBookManagerImpl() {
        this.attachInterface(this,DESCRIPTOR);
    }

    public static IBookManager asInterface(IBinder obj){
        if (obj==null){
            return null;
        }
        IInterface iin=obj.queryLocalInterface(DESCRIPTOR);
        if ((iin!=null) && (iin instanceof IBookManager) ){
            return (IBookManager) iin;
        }
        return new IBookManagerImpl.Proxy(obj);
    }

    @Override
    protected boolean onTransact(int code, Parcel data, Parcel reply, int flags) throws RemoteException {
        switch (code){
            case INTERFACE_TRANSACTION:
            {
                reply.writeString(DESCRIPTOR);
                return true;
            }
            case TRANSACTION_addBook:
            {
                data.enforceInterface(DESCRIPTOR);
                Book _argo;
                if (0!=data.readInt()){
                    _argo=Book.CREATOR.createFromParcel(data);
                }else {
                    _argo=null;
                }
                this.addBook(_argo);
                reply.writeNoException();
                return true;
            }
            case TRANSACTION_getBookList:
            {
                data.enforceInterface(DESCRIPTOR);
                List<Book> result=this.getBookList();
                reply.writeNoException();
                reply.writeTypedList(result);
                return true;
            }
        }
        return super.onTransact(code, data, reply, flags);
    }

    @Override
    public List<Book> getBookList() throws RemoteException {
        return null;
    }

    @Override
    public void addBook(Book book) throws RemoteException {

    }

    @Override
    public IBinder asBinder() {
        return this;
    }


    private static class Proxy implements IBookManager{
        private IBinder mRemote;
        Proxy(IBinder remote){
            this.mRemote=remote;
        }

        public String getInterfaceDescriptor(){
            return DESCRIPTOR;
        }

        @Override
        public List<Book> getBookList() throws RemoteException {
            Parcel data=Parcel.obtain();
            Parcel reply=Parcel.obtain();
            List<Book> result;
            try {
                data.writeInterfaceToken(DESCRIPTOR);
                mRemote.transact(TRANSACTION_getBookList,data,reply,0);
                reply.readException();
                result=reply.createTypedArrayList(Book.CREATOR);
            }finally {
                reply.recycle();
                data.recycle();
            }

            return result;
        }

        @Override
        public void addBook(Book book) throws RemoteException {
            Parcel data=Parcel.obtain();
            Parcel reply=Parcel.obtain();
            try {
                data.writeInterfaceToken(DESCRIPTOR);
                if (book!=null){
                    data.writeInt(1);
                    book.writeToParcel(data,0);
                }else {
                    data.writeInt(0);
                }
                mRemote.transact(TRANSACTION_addBook,data,reply,0);
            }finally {
                reply.recycle();
                data.recycle();
            }
        }

        @Override
        public IBinder asBinder() {
            return mRemote;
        }
    }
}

```

