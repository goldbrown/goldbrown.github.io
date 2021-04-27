---
layout:     post
title:      java8新特性之stream实现原理
subtitle:   stream实现原理
date:       2019-08-31
author:     Chris
header-img: img/post-bg-liuyifei.jpg
catalog: true
tags:
    - java
    - java8
    - stream
---

前一篇文章讲过，stream的操作分为中间操作和终止操作。中间操作的调用时，效果类似于添加一个监听器，实际的执行阶段在终止操作调用的时候。例如下面的filter在调用时，并不会真正的去对流进行过滤。而是在collect()调用的时候，将filter()和collect()一起执行。

```java
list.stream().filter(s -> s != null)
  .filter(s -> s.length() > 5).collect(Collectors.toList());
```

下面我们从实现上讲讲stream的原理。

# 1、类的继承关系

类的继承关系如下图所示
![](https://tva1.sinaimg.cn/large/006y8mN6gy1g6j8j843asj30p00llgn2.jpg)
![](https://tva1.sinaimg.cn/large/006y8mN6gy1g6j8jzb9ggj30rz09z3z7.jpg) 

`BaseStream`：最基础的stream接口。定义了流最基础的操作，比如，将流转化为并行流的操作parallel()和将流转化为串行流的操作sequential()。   
`Stream`：定义了流最常用的用户可以使用的操作，比如，filter()，map()，reduce()。   
`PipelineHelper`：流操作的抽象辅助类，主要是定义了内部使用的辅助方法，不是给用户使用的。比如，将用户操作封装成Sink的方法wrapSink()方法和将流元素放入Sink的方法copyInto()。   
`AbstractPipeline`：实现了BaseStream接口和PipelineHelper抽象类，是最核心的方法实现类，包括wrapSink()和copyInto()的实现。   
`ReferencePipeline`：实现了Stream接口和AbstractPipeline抽象类。主要是实现了Stream接口定义的操作方法，比如filter()，map()，reduce()   
`StatelessOp`：无状态的中间操作。继承了ReferencePipeline，属于ReferencePipeline操作的一种。   
`StatefulOp`：有状态的中间操作。继承了ReferencePipeline，属于ReferencePipeline操作的一种。   
`Head`：ReferencePipeline的源节点。继承了ReferencePipeline，属于ReferencePipeline操作的一种。

`TerminalOp`：终止操作   
`MatchOp`，`ForEachOp`，`ReduceOp`：依次代表匹配终止操作、迭代操作和reduce操作，实现了TerminalOp接口。

# 2、执行过程

stream的执行过程主要分为如下几步：   
1）将多个操作构建成作用链，该作用链是一个双向链表    
2）将每个元素依次让作用链进行处理    
3）终止操作收集作用链处理的结果    

下面，以如下的例子，仔细讲这个过程。

```java
List<String> list = Lists.newArrayList("Chris Kate", "Bruce", "Tina");
list.stream().filter(s -> s.length() > 5).sorted((s1, s2) -> s1.length() - s2.length()).collect(Collectors.toList());
```

其中，filter()为无状态中间操作，sorted()为有状态中间操作。collect()为终止操作。

## 构建作用链

一般，stream的操作为：一个源节点，多个中间操作，最后一个终止操作。因此，构建的双向链表一般如下图所示。
![](https://tva1.sinaimg.cn/large/006y8mN6gy1g6j8n2oi4yj313q05i3z6.jpg)
1）当调用list.stream()时，就会创建一个源节点。  
![](https://tva1.sinaimg.cn/large/006y8mN6gy1g6j8o5gpfvj308g056aa0.jpg) 
2）调用filter()无状态中间操作     
当调用filter()时，只是将操作加到链表末尾。将Head和FilterOp连接起来。   
![](https://tva1.sinaimg.cn/large/006y8mN6gy1g6j8pkoml6j30fr05mwel.jpg)
3）调用sorted()有状态中间操作   

当调用sorted()时，也是将排序Op操作加到链表末尾。此外，一些属性会记录排序操作需要的数据，比如，比较器Comparator。
![](https://tva1.sinaimg.cn/large/006y8mN6gy1g6j8qbzywoj30q605yaad.jpg)
## 作用链循环处理每个元素

4）调用collect()终止操作    

终止操作是收集处理结果的操作。因此，在调用collect()终止操作时，就会
- 遍历流元素，依次让流元素经过作用链的处理。
- 由于排序Op是有状态的中间操作，因此，排序Op会存储前面的操作处理过的流元素，进行一些操作。然后再传递给其下游进行处理。

> **注意**：终止操作只是用于收集结果，终止操作处理之后，返回的并不是一个流。那么，这个流的处理就结束了。

下面看看代码的实现。

用来收集作用链处理结果的数据结构叫做Sink（水槽），其继承关系如下。
![](https://tva1.sinaimg.cn/large/006y8mN6gy1g6j8sc20raj30mp0y1q53.jpg)
`Sink`是用来收集处理结果的水槽，其定义了收集的接口，包括begin(),accept()和end()。客户端得保证调用顺序依次是begin()，accept()和end()，类似于模板模式。    
`AccumulatingSink`是用来收集reduce结果的水槽接口。定义了combine(K other)这个具体reduce的动作接口。   
`ReducingSink`实现了具体的begin()，accept()和combine()动作。    

## 调用collect()操作之后的事情

4.1）判断流是串行流还是并行流，分别调用各自的处理方法

```java
return isParallel()
  ? terminalOp.evaluateParallel(this, sourceSpliterator(terminalOp.getOpFlags()))
  : terminalOp.evaluateSequential(this, sourceSpliterator(terminalOp.getOpFlags()));
```

4.2）如果是串行流，则会调用终止操作对应的evaluate方法。
![](https://tva1.sinaimg.cn/large/006y8mN6gy1g6j8umulxaj30ih04pwfd.jpg)
我们这里考虑ReduceOp的情况。

3）在evaluate方法中，会创建一个ReducingSink，之后会用来收集reduce的处理结果。
![](https://tva1.sinaimg.cn/large/006y8mN6gy1g6j8uhasqlj30u105gjsb.jpg)
4）接下来，就是使用wrapSink()对Sink进行封装。封装的结果就是形成一个水槽的单向链表。
![](https://tva1.sinaimg.cn/large/006y8mN6gy1g6j8w1b4qgj313e097405.jpg)

![](https://tva1.sinaimg.cn/large/006y8mN6gy1g6j8wvwjeij30vy08lgm6.jpg)
5）使用copyInto()方法，对元素进行迭代，依次由水槽单向链表处理。
![](https://tva1.sinaimg.cn/large/006y8mN6gy1g6j8xcz94fj30y00bh40p.jpg)

![](https://tva1.sinaimg.cn/large/006y8mN6gy1g6j8xu7evhj30sm0jxn0j.jpg)
在forEachRemaing()中，会对流元素进行迭代处理。每个元素都经过action来处理。这里的action就是前面提到的封装后的sink单向链表。


**copyInto()是执行过程中很关键的方法，封装了整个处理逻辑**。其对于元素的处理可以通过下面的示意图来表示。   
从图中可以看出，    
1）每个Sink都会调用各自的begin()，accept()和end()方法。begin()是通知Sink流将要过来了，请Sink做好准备，比如，有状态的操作需要创建一个List来存储元素。一般，accept()是正式的处理流的操作。end()通知Sink流的处理结束了，之前创建的一些临时数据结构可以销毁了。    
2）对于有状态的操作（水槽），对应的Sink会有存储中间结果的数据结构。比如，Sort Sink会使用一个List来存储流元素。在end()调用之后，才进行用户传入的操作，例如排序，因为必须等待所有元素到来之后才可以进行排序。之后，再调用下游的begin()，forEach()（accept()方法）和end()方法，开始新的流处理。    
3）对于终止操作，在begin()调用时，会创建一个对应的数据结构来存储结果；在forEach()的过程中，收集元素；在end()调用时，返回结果。   
![](https://tva1.sinaimg.cn/large/006y8mN6gy1g6j8yx9oksj31db0u0n2t.jpg)


# 总结
本文主要是讲述了常见的流实现类的继承关系，以及结合具体操作，讲述了流的实现原理。最后一张图比较形象的体现了流的执行过程，读者可以结合源码好好体会。

# 参考
https://www.ibm.com/developerworks/cn/java/j-java-streams-3-brian-goetz/index.html   
http://hack.xingren.com/index.php/2018/10/17/java-stream/   
https://colobu.com/2014/11/18/Java-8-Stream/   
https://zhuanlan.zhihu.com/p/52579165   