---
title: Activity学习笔记
comments: true
date: 2017-09-10 
categories: Android
tags:
img:
---
## **一、Activity 的生命周期**

***典型情况下的生命周期：***
```
onCreate() –> activity 正在被创建，执行初始化工作，如 setContentView

onRestart() –> activity 正在重新启动

onStart() –> activity 正在启动，已经后台可见，但是用户看不到

onResume() –> activity 前台可见。可以和用户交互

onPause() –> activity 正在停止，此时可以执行一些不耗时的操作，如存储数据，onPause 必须先执行完 新的activity 的onResume 才能执行

onStop() –> activity 即将停止，可以做一些稍微耗时的回收工作，同样不能太耗时

onDestory() –> activity 即将被销毁，回收工作和最终资源的释放
```



***异常情况下的生命周期：***

```
资源相关的系统配置发生改变导致 activity 被杀死 并重新创建（横竖屏切换）
资源内存不足导致低优先级的 activity 被杀死
```

　如果不想在系统配置发生变化时重启 activity 指定 configChanges 属性，比如我们不想在屏幕旋转时重新创建，可以给 configChanges 属性添加 orientation 这个值,如下所示。	

<br>
## **二、Activity 的启动模式**

* **standard : 标准模式。**每次启动都会创建一个实例，在这种模式下，谁启动了这个activity，那么这个activity就运行在启动它的那个activity 所在的栈上。当我们用非Activity类型的Context 启动activity 时，并没有所谓的栈，所以会报错，解决方式是 为待启动的activity 指定一个FLAG_ACTIVITY_NEW_TASK 标记位,这样启动的时候就会为它创建一个新的任务栈。这时候带启动的activity 实际上是以 singleTask模式启动的。

* **singleTop ：栈顶复用模式。**在这个模式下，如果新activity 已经位于任务栈的栈顶，那么此activity 不会被重新创建，同时它的 onNewIntent方法会被回调，通过此方法，我们可以取出当前请求的信息。需要注意的是，这个activity 的 onCreate、onStart（）不会被重新调用，因为它没有发生改变。

* **singleTask ：栈内复用模式。**这是一种单实例模式，在这种模式下，只要activity 在一个栈中存在，那么多次启动此activity 都不会创建实例，和singleTop，系统也会回调其onNewIntent。SingleTask 模式默认具有clearTop 的效果，会导致启动 activity 上面的所有activity 出栈。

* **singleInstance : 单实例模式。**这是一种加强的 SingleTask 模式，具有此模式的 activity 只能单独地位于一个任务栈中。

<br>
## **三、activity 的 Flags**

```
FLAG_ACTIVITY_NEW_TASK ：和在xml 中指定的singleTask 效果相同。

FLAG_ACTIVITY_SINGLE_TOP ：和在xml 中指定的singleTop效果相同。

FLAG_ACTIVITY_CLEAR_TOP ： 当有此标记的 activity 启动时，在同一个栈中位于它上面的 activity 都要出栈。一般与 FLAG_ACTIVITY_NEW_TASK 配合使用，singleTask 默认具有此效果。

FLAG_ACTIVITY_EXCLUDE_FROM_RECENTS：具有这个标记为的 activity 不会出现在历史 activity 列表中，当某些情况下我们不希望用户通过历史列表回到我们的 activity 的时候这个标记比较有用。
```
<br>

## **四、IntentFilter的匹配规则**

![图片加载失败](1.jpg)

{% blockquote %}
如图所示：　一个 activity 中可以有多个 intent-filter (过滤规则)，一个 intent-filter 可以有多个 action、category、data。 一个 intent 只要匹配任何一组的 intent-filter 即可成功启动对应的 activity。只有同时匹配一个 intent-filter 中的 action、data、category 才算匹配成功。

IntentFilter 中的过滤信息有 action、category、data。
{% endblockquote %}

### **4.1、action 的匹配规则：**

```
action 是一个字符串，匹配规则是，一个Intent 中必须存在action 且和过滤规则中的某一个action 相同。
```

### **4.2、category 的匹配规则：**

```
category 也是一个字符串，匹配规则和 action 不同：一个 Intent 可以设置或不设置 category，如果设置 category 那么每个设置的 category 都必须要跟过滤规则 中的 category 相同才算匹配通过。如果不设置 category，则在 startActivity 或 startActivityforResult 时会默认加上 android.Intent.category.DEFAULT。所以为了 activity 能够被隐式开启，我们要在 Intent-filter 中指定 “ android.intent.category.DEFAULT ” 这个 category。
```

### **4.3、data 的匹配规则：**
<br>
**4.3.1**  在学习data的匹配规则前，我们先来了解下data的结构。

![图片加载失败](2.jpg)

{% blockquote %}
data 由 mimeType 和 URI 组成。mimeType 指的是媒体类型，如 image/jpeg 、video/* 等。而URI 中包含的数据就比较多了下面是URI 的结构 ：
{% endblockquote %}

```
<scheme>://<host>:<port>/[<path>|<pathPrefix>|<pathPattern>]

例如:

content://com.example.project:200/folder/sunfolder/etc

http://www.baidu.com:80/search/info
```

* **Scheme**	
URI 的模式，如 http、file、content。如果没有指定scheme，那么此URI 的其他参数无效，也就是URI无效。

* **Host**
URI 的主机名，如 www.baidu.com ，如果没有指定Host，那么此URI也是无效的。

* **Port**
URI 的端口号，比如80，仅当URI 中指定了 scheme和host的时候port 才是有效的。

* **Path、pathPrefix、pathPattern**
这三个参数都表示路径信息。其中Path 表示完整的路径信息。pathPattern 也表示 完整的路径信息，但是它可以包含 通配符， 表示 0个或多个字符。pathPrefix 表示路径的前缀。
<br>

**4.3.2**  了解完了data的结构，我们再来看data的匹配规则
```
　data 的匹配规则 和 action 类似，它也要求 Intent 中必须含有 data数据，并且data 数据能够完全匹配过滤规则的某一个data 。

　如果过滤规则 data中没有指定 URI ，但是却有默认值，默认值是 content 和 file 。也就是说 Intent 中的URI部分的scheme 必须为 content 或者file 才能匹配。

　如果要为Intent指定完整的data，那么必须用 setDataAndType（） 方法，因为 setData 和 setType 会彼此清除 对方的值。
```

　最后，当我们用隐式意图开启 activity时，可以做一下判断，看看是否有activity匹配我们的Intent。判断方法有两种：packageManager 的resolveActivity或者 Intent的resolveActivity方法。如果找不到匹配信息会返回null。PackageManager 还有一个 queryIntentActivities 方法可以返回所有匹配成功的activity信息。