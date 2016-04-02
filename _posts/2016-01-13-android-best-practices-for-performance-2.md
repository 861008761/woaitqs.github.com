---
layout: post
title: "Android 最佳实践 (2)"
description: "Managing Your App's Memory"
category: "program, java, android"
tags: [android, program]
---
{% include JB/setup %}

这篇文章主要讲一些能改善性能的一些小 Tips。使用正确的算法和数据结构是第一优先级的事情，但不在本文的讨论范围中。本文中的一些小 Tips 能够显著改善一些编程习惯。

简单而言，是2方面的工作

1. 不做不必要的工作
2. 不分配不必要的内存

在 Android 系统中进行开发面临的一个显著问题就是我们所开发的 App 将在多个机型上运行，这些机型的分辨率，性能等差异巨大。

<!--break-->

## 避免创建不必要的对象

创建对象不是一个消费很低的事情。当你给应用中的 App 分配内存的时候，可能会引发强制垃圾回收，给用户造成卡顿的影响。尽管在 Android 2.3 开始引入的并行垃圾回收器对于这种现象有一定帮助，但是我们还是要尽量避免不必要的工作。

例如如下的一些例子：

1. 如果一个方法返回String类型的数据，并且你知道这个返回值将用于添加到其他String中，那么直接返回 StringBuffer 就可以了，这样可以避免创建临时对象。
2. 如果从一组输入数据中解析相应的值，那么返回原始数据中的一部分，而不是返回一份拷贝。如果你创建了新的String对象，那么 Char[] 会被重复利用，但这也导致即使你只有一部分数据被使用，整个String 还是会被缓存起来。
3. 两个单独的int数组要比一对（int，int）的更具有效率。

一般来讲，应该尽量避免临时创建对象。更少的创建对象意味着更低频率的垃圾回收，那么对用户体验也有更多的好处。

## 尽量申明 Static 变量

如果你不用访问对象的内部的变量，那么把方法声明为 static，测试标明这种方式能够提速 15%-20%。而且这也是一条最佳实践，这能通过签名能够告诉方法调用者这个类不会修改对象内部状态。

## 对变量使用 Static Final

考虑如下的几个变量申明：

```java
static int intVal = 42;
static String strVal = "Hello, world!";
```

编译器会生成一个 <clinit> 的方法，这个方法会在类第一次被加载的时候调用。这个方法会将 42 这个值存储到 intVal 中，并且持有一个对strVal的引用。其后这些变量也会被方法表引用。

通过添加 final 关键字可以解决这个问题。

```java
static final int intVal = 42;
static final String strVal = "Hello, world!";
```

这个类就不再需要 <clinit> 方法，因为这些变量将直接在 dex 文件中的初始化。42 这个字会被直接存下来，strVal 将使用相对不那么笨重的常量指令而不是 field lookup表。

## 避免内部使用的Getter和Setter方法

使用 ProGuard 可以帮我们优化。