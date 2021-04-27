---
layout:     post
title:      谷歌MapReduce论文阅读笔记
subtitle:   大数据,MapReduce
date:       2020-01-21
author:     Chris
header-img: img/post-bg-mulan_lyf3.jpg
catalog: true
tags:
    - 大数据
    - MapReduce
---

## 1 标题
* 标题   
    MapReduce: Simplied Data Processing on Large Clusters

* 引用信息    
    Dean, Jeffrey, and Sanjay Ghemawat. "MapReduce: simplified data processing on large clusters." Communications of the ACM 51.1 (2008): 107-113.


## 2 主题类别
* 大数据
* MapReduce


## 3 背景
在过去五年中，作者写过很多特定目的的程序，用来处理大量的原始数据（例如爬取的文档和网络请求日志），用来产生指定的数据，例如倒排索引和网页之间的引用关系。这些程序的计算逻辑比较简单，但是需要分布到不同的机器上进行计算。面对的问题有：在机器上并行计算、分布数据到不同机器上、处理机器故障和程序bug。为了简化这些问题，抽象出一个通用的处理框架，于是有了这篇论文。


## 4 目标
抽象出通用的模型，能够简化问题的处理。模型框架能够简化用户使用，处理并行计算问题和分布数据的问题等问题。

## 5 方法
经验总结。通过借鉴Lisp等函数式语言中的map和reduce思想，将处理逻辑抽象成map和reduce两部分，MapReduce框架会处理并行计算的细节、容错、数据在机器的分布和负载均衡问题。

* MapReduce思想是什么    
    MapReduce计算模型输入为一系列的key/value对，输出也为一系列的key/value对。计算过程主要分为两部分：map和reduce。   
    * map   
        map函数由用户编写，接收一系列key/value对，产生一系列的中间结果，中间结果也为key/value对。map函数需要将拥有相同key的中间结果的value值放到一起，然后传递给reduce函数。例如，传递给reduce函数的数据可能为`(key, list(v1, v2, v2, ...))`。   
    * reduce   
        reduce函数也由用户编写，接收map函数传递的`(key, list(v1, v2, v2, ...))`，将`list(v1, v2, v2, ...)`处理为大小更小的集合，比如，结果可能就会输出一个数字或者没有输出。

    示例如下：   
    下图是论文中一个统计海量文档中的词频的例子，给出了map和reduce函数是怎么写的。map函数的实现中，每碰到一个单词时，将该单词的频率设置为1(类似于只有一个元素的list)，然后传给reduce函数。reduce函数将values中的元素值进行累加，然后输出该key所对应的词频。可见,map和reduce函数的实现，都是紧紧围绕“相同的key”来实现的。map函数的输出和reduce的输入输出都是相同的key。   

    <img src="https://tva1.sinaimg.cn/large/006tNbRwgy1gb4663b2poj30p00fu0t5.jpg" width="80%" height="80%"> 

* 执行流程

    <img src="https://tva1.sinaimg.cn/large/006tNbRwgy1gb46ep56l7j314a0u075j.jpg" width="100%" height="100%">

    上图是整个执行流程。   
    1. 用户程序中的MapReduce库首先将输入数据分割为16M到64M的小文件（大小也可以由用户控制，可选），假设有M个（例如输入总大小为t M，每个小文件大小为64M，则个数为t/64）。然后启动集群中的一些机器，将程序拷贝到这些启动的机器上。
    2. 集群中的机器分为一个master和多个worker，master获得的执行程序是特殊的。master用来分配任务，worker用来执行map任务或者reduce任务。master会选择一些worker执行map任务，另一些执行reduce任务。   
    3. map worker（指执行map任务的worker）会读取分割好的小文件，解析为key/value对，将key/value对传递给用户编写的Map函数。Map函数输出的中间结果会缓存在内存中。   
    4. 缓存在内存中的中间结果会周期性的写入本地磁盘，写入的位置由分区函数（例如hash(key) mod R）决定，R为分区的数量，可由用户决定。然后，中间结果在磁盘中的位置会回传给master，master将这些位置传递给reduce worker（执行reduce任务的worker）。   
    5. 当reduce worker从master那里收到中间结果的位置信息后，就会通过RPC（remote procedure call）从map worker的本地磁盘上读取中间结果。读取到中间结果之后，该worker就会根据key将中间结果排序分组。
    6. reduce worker遍历输入的中间结果。将key及其对应的一系列value值传递给用户编写Reduce函数，将Reduce函数的输出结果追加到最终的输出文件中。
    7. 当所有的Map任务和Reduce任务都完成之后，master唤醒用户程序，MapReduce调用返回。

* 容错性
    * worker容错性
        master会周期性的ping所有的worker。如果worker没有返回结果，则将该worker标记为fail。处于fail状态的worker执行的任务，满足以下条件时需要重新执行：   
        * 处于completed状态的map任务
        * 处于in-progress状态的map任务或者reduce任务
        
        completed状态的map任务需要重新执行，是因为中间结果存储在本地磁盘中；completed状态的reduce任务不需要重新执行，是因为最终结果存储在global file system中。
    * master容错性
        master会周期性的写入checkpoint检查点。如果master失败，则从最近的一个检查点恢复，继续执行。因为master只有一个，失败的概率是很小的，目前的实现就是：master失败时放弃执行，让用户重新触发任务。
