---
title: Hello View
comments: true
date: 2017-07-09
categories: Android
tags: Android View 专题
img:
---
{% blockquote %}
　作为一个 Android 开发者，几乎每一个 Android 项目都离不开 View，虽然说 View 不属于四大组件，但它的重要性丝毫不低于它们，因此今天让我们来重新认识一下 View。
{% endblockquote %}

## 1、什么是 View ？

	    View（视图）是 Android 中所有控件的基类，不管是简单的 TextView 和 Button 还是 RelativeLayout 和 ListView 他们的共同基类都是 View 。RelativeLayout 继承的 ViewGroup （控件组）也继承自 View ，ViewGroup 内部包含了许多个控件，即一组 View 。所以说 View 是一种界面层的控件的一种抽象，它代表了一个控件或一组控件。
下面一张图可以帮助你理解常用的控件关系

![图片加载失败](1.png)
<br>

## 2、View 的位置参数

	    View 的位置主要由它的四个顶点来决定，分别对应与 View 的四个属性： top、left、right、bottom。需要注意的是，这些坐标都是相对与 View 的父容器来说的，因此它是一种相对坐标。 View 的坐标和父容器的关系见下图：

![图片加载失败](2.png)

根据这张图，我们可以很容易的得出 View 的宽高和坐标的关系：

```
width = right - left 
height = bottom - top
```

那么如何得到 View 的这四个参数呢？ 也很简单，在 View 的源码中它们对应与 mLeft、mRight、mTop、mBottom这四个成员变量，获取方式如下：
```
left = getLeft();
right = getRight();
top = getTop();
bottom = getBottom();

```

　　从 Android3.0 开始, View 增加了几个额外的参数：　x、y、translationX　和　translationＹ，其中 x 和 y 是 View 左上角的坐标，而 translationX 和 translationY 是 View 左上角相对与父容器的偏移量。这几个参数也是相对与父容器的坐标，并且 translationX 和 translationY 的默认值是 0，和 View 的四个基本位置参数一样，View 也为它们提供了 get/set 方法，这几个参数的换算关系如下：

```
x = left + translationX 
y = top  + translationY
```

　　需要注意的是，View 在平移的过程中，top 和 left 表示的是原始左上角的位置信息，其值并不会发生改变，此时发生改变的是 x、y、translationX 和 translationY 这四个参数。
<br>

## 3、View 的四个构造函数

<br>
* **public MyView(Context context)**

　　java 代码直接 new 一个 Custom View 实例的时候,会调用第一个构造函数
* **public MyView(Context context,AttributeSet attrs)**

　　在 xml 创建但是没有指定 style 的时候被调用。多了一个 AttributeSet 类型的参数，自定义属性，在通过布局文件 xml 创建一个 view 时，会把 XML 内的参数通过 AttributeSet 带入到 View 内。
* **public MyView(Context context,AttributeSet attrs,int defStyleAttr)**

　　构造函数中第三个参数是默认的 Style，这里的默认的 Style 是指它在当前 Application 或 Activity 所用的 Theme 中的默认 Style ，且只有在明确调用的时候才会调用
* **public MyView(Context context, AttributeSet attrs, int defStyleAttr, int defStyleRes)**

　　如果没有使用 defStyleAttr ，则会应用 defStyleRes 到 View 的默认风格。该构造函数是在 API 21 的时候才添加上的


<br>
## 4、View 的生命周期

	相关方法见下图

![图片加载失败](3.png)

**生命周期的调用顺序**
1. View 默认为可见的，不是默认值时先调用 onVisibilityChanged()，但是此时该 View 的尺寸、位置等信息都不知道。
2. 可见性改变后才是调用带有两个参数的构造函数，当然，如果该 View 不是在 layout 中定义的话，会调用一个参数的构造函数。
3. 从 XMl 文件中 inflate 完成（onFinishInflate()）。
4. 将 View 加到 window 中（ View 是 gone 的，那么 View 创建生命周期也就结束）。
5. 测量 view 的长宽（onMeasure()）。
6. 定位 View 在父 View 中的位置（onLayout()），若 View 是 invisible，则 View 的创建生命周期结束。
7. 绘制 View 的 content（onDraw()），只有可见的 View 才在 window 中绘制。
8. View 的销毁流程和可见性没有关系。

**所以 View 的生命关键周期为：**

	[改变可见性] --> 构造View() --> onFinishInflate() --> onAttachedToWindow() --> onMeasure() --> onSizeChanged() -->onLayout() --> onDraw() --> onDetackedFromWindow()

***
**参考资料**
{% link 【Android 面试题总结之Android 进阶】 http://www.androidchina.net/5035.html %}
{% link 【Android 开发艺术探索】  %}
{% link 【Android View的生命周期】 https://www.jianshu.com/p/08e6dab7886e %}