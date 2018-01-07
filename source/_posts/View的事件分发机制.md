---
title: View的事件分发机制
comments: true
date: 2017-07-15
categories: Android
tags: Android View 专题
img:
---
{% blockquote %}
一说到事件分发，首先我们要明白这里要分析的对象就是 MotionEvent ，即分发的事件。 

所谓 View 的事件分发，其实就是对 MotionEvent 事件的分发过程，即当一个 MotionEvent 产生了以后，系统需要把这个事件传递给一个具体的 View ，而这个传递的过程就是分发的过程。

为了便于理解记忆，我觉得将 View 的事件分发机制 分为三个问题来一步一步的学习效果会更好：

***1. 分发的事件***

***2. 是谁在分发事件***

***3. 分发的过程***

{% endblockquote %}
<br>
## **一、分发的事件 （MotionEvent）**

关于 MotionEvent 。官方是这样介绍的：

Object used to report movement (mouse, pen, finger, trackball) events.
对象用于报告动作（鼠标、笔、手指、轨迹球）的事件。

简单的来说，我们在手机屏幕上的各种触摸操作都会产生一个个的事件，这些事件被封装成了一个个的 MotionEvent 对象，然后被传递给需要它的 View。

以手指为例,在手指接触屏幕后所产生的一系列事件中，典型的事件类型有如下几种：

	ACTION_DOWN —— 手指刚接触屏幕
	ACTION_MOVE —— 手指在屏幕上移动
	ACTION_UP —— 手指从屏幕上松开的一瞬间

正常情况下，一次手指触摸屏幕的行为会触发一系列点击事件，考虑如下几种情况：

	点击屏幕后离开松开，事件序列为 DOWN –> UP；
	点击屏幕滑动一会再松开，事件序列为 DOWN –> MOVE –> ……> MOVE –> UP；

上述两种情况就是典型的事件序列，同时通过 MotionEvent 对象我们可以得到点击事件发生的 x 和 y 坐标。为此系统提供了 getX/getY 和 getRawX/getRawY 两种方法。其中 getX/getY 返回的是相对与当前 View 左上角的 x 和 y 坐标，而 getRawX/getRawY 返回的是相对与手机屏幕左上角的 x 和 y 坐标。

关于 MotionEvent 就先了解到这，下面我们来看第二个问题。

<br>

## **二、是谁在分发事件？**