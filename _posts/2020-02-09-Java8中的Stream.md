---
layout:     post                    # 使用的布局（不需要改）
title:      Java8中的Stream            # 标题 
subtitle:   Java8中的Stream            #副标题
date:       2020-01-21              # 时间
author:     ListJ                      # 作者
header-img: img/post-bg-2015.jpg    #这篇文章标题背景图片
catalog: true                       # 是否归档
tags:                               #标签
    - java
---

### 什么是Stream?

> Java8中的Stream是对集合功能的一种增强，主要用于对集合对象进行各种非常便利高效的聚合和大批量数据的操作。结合Lambda表达式可以极大的提高开发效率和代码可读性。

**Stream跟Iterator的差别是:**

 - 无存储：Stream是基于数据源的对象，它本身不存储数据元素，而是通过管道将数据源的元素传递给操作。
   
 - 函数式编程：对Stream的任何修改都不会修改背后的数据源，比如对Stream执行filter操作并不会删除被过滤的元素，而是会产生一个不包含被过滤元素的新的Stream。

   

 - 延迟执行：Stream的操作由零个或多个中间操作（intermediate operation）和一个结束操作（terminal   
   operation）两部分组成。只有执行了结束操作，Stream定义的中间操作才会依次执行，这就是Stream的延迟特性。

  

 - 可消费性：Stream只能被“消费”一次，一旦遍历过就会失效。就像容器的迭代器那样，想要再次遍历必须重新生成一个新的Stream。

Stream可以认为是一个高级版本的 Iterator。它代表着数据流，流中的数据元素的数量可以是有限的，也可以是无限的。Stream提供了强大的数据集合操作功能，并被深入整合到现有的集合类和其它的JDK类型中。流的操作可以被组合成流水线（Pipeline）。

**假设我们需要把一个集合中的所有形状设置成红色，那么我们可以这样写：**

```java
for (Shape shape : shapes){
    shape.setColor(RED)
}
```
如果使用Java8扩展后的集合框架则可以这样写：

```java
shapes.foreach(s -> s.setColor(RED));
```
第一种写法我们叫**外部迭代**，for-each调用shapes的iterator()依次遍历集合中的元素。这种外部迭代有一些问题：

 - for循环是串行的，而且必须按照集合中元素的顺序依次进行；
 - 集合框架无法对控制流进行优化，例如通过排序、并行、短路求值以及惰性求值改善性能。

第二种写法我们叫**内部迭代**，两段代码虽然看起来只是语法上的区别，但实际上他们内部的区别其实非常大。用户把对操作的控制权交还给类库，从而允许类库进行各种各样的优化（例如乱序执行、惰性求值和并行等等）。总的来说，内部迭代使得外部迭代中不可能实现的优化成为可能。
外部迭代同时承担了做什么（把形状设为红色）和怎么做（得到Iterator实例然后依次遍历），而内部迭代只负责做什么，而把怎么做留给类库。这样代码会变得更加清晰，而集合类库则可以在内部进行各种优化。

### Stream的操作
拿前面的例子来说，如果我只想把蓝色改成红色：

```java
shapes.stream()
      .filter(s -> s.getColor() == BLUE)
      .forEach(s -> s.setColor(RED));
```
在`Collection`上调用`stream()`会生成该集合元素的流，接下来`filter()`操作会产生只包含蓝色形状的流，最后，这些蓝色形状会被`forEach`操作设为红色。

如果我们想把蓝色的形状提取到新的`List`里，则可以：

```java
List<Shape> blue = shapes.stream()
                          .filter(s -> s.getColor() == BLUE)
                          .collect(Collectors.toList());
```
`collect()`操作会把其接收的元素聚集到一起（这里是`List`），`collect()`方法的参数则被用来指定如何进行聚集操作。在这里我们使用`toList()`以把元素输出到List中。
如果每个形状都被保存在Box里，然后我们想知道哪个盒子至少包含一个蓝色形状，我们可以这么写：

```java
Set<Box> hasBlueShape = shapes.stream()
                               .filter(s -> s.getColor() == BLUE)
                              .map(s -> s.getContainingBox())
                              .collect(Collectors.toSet());

```
`map()`操作通过映射函数（这里的映射函数接收一个形状，然后返回包含它的盒子）对输入流里面的元素进行依次转换，然后产生新流。

如果我们需要得到蓝色物体的总重量，我们可以这样表达：

```java
int sum = shapes.stream()
                .filter(s -> s.getColor() == BLUE)
                .mapToInt(s -> s.getWeight())
                .sum();
```

### Stream vs Collection
流（Stream）和集合（Collection）的区别：

 - Collection主要用来对元素进行管理和访问；
 - Stream并不支持对其元素进行直接操作和直接访问，而只支持通过声明式操作在其之上进行运算后得到结果；
 - Stream不存储值
 - 对Stream的操作会产生一个结果，但是Stream并不会改变数据源；

- 大多数Stream的操作(filter,map,sort等)都是以惰性的方式实现的。这使得我们可以使用一次遍历完成整个流水线操作,并可以用短路操作提供更高效的实现。

