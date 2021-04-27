---
layout:     post
title:      java8新特性之stream操作
subtitle:   stream操作
date:       2019-08-24
author:     Chris
header-img: img/post-bg-hemeitian.jpg
catalog: true
tags:
    - java
    - java8
    - stream
---


# stream介绍

java8引入了流(stream）处理，大大简化了集合操作。比如，遍历操作。一般会使用for-each循环，但是，在java8中，流处理就可以做到更加简洁。如下代码所示。
```java
List<String> list = Lists.newArrayList("Chris Kate", "Bob Nana", "Tina Peter", "Bruce", "Jaker Alicia", null);

// 传统的使用forEach循环
int totalLen = 0;
for (String s : list) {
  if (StringUtils.isEmpty(s)) {
    continue;
  }
  totalLen += s.length();
}
System.out.println(totalLen);

// 使用java8流处理
long count = list.stream().filter(item -> StringUtils.isNotEmpty(item))
  .map(item -> item.length())
  .reduce((len1, len2) -> len1 + len2)
  .orElse(0);
System.out.println(count);
```
在java8之前，对集合进行遍历，只能使用forEach循环。在java8之后，就可以使用流来对集合进行循环。

在上面的例子中，
1）流处理首先使用stream()创建一个指向该集合的流   
2）使用filter()来过滤空指针   
3）使用map()将每个元素映射到元素长度。映射之后，新的流里面存储的就是每个元素的长度了。   
4）使用reduce()操作，将集合中元素的长度相加。   
5）reduce()返回的是一个Optional对象，orElse(0)可以取出Optional对象里面的值；若里面的值为空，则返回0。   

# stream中的操作

## 怎么创建流

有多种方式可以创建一个流

- 从Collection集合创建
这通常是最常见的创建流的方式。

```java
List<String> list = new ArrayList<>(); 
list.add("Bob"); 
list.add("Tina"); 
list.add("Chris"); 
Stream<T> stream = list.stream(); 
```

- 使用指定的元素来创建流
  
```java
Stream<Integer> stream = Stream.of(1, 2, 3, 4, 5, 6, 7, 8, 9); 
```

- 使用数组来创建流
  
```java
String[] arr = new String[] { "a", "b", "c" }; 
Stream<T> streamOfArray = Arrays.stream(arr); 
```

- 创建一个空的流

```java
Stream<String> streamOfArray = Stream.empty(); 
```

- 使用Stream.builder来创建流（使用指定元素）
  
```java
Stream.Builder<String> builder 
            = Stream.builder(); 
Stream<String> stream = builder.add("a") 
  .add("b") 
  .add("c") 
  .build(); 
```
上面说的是比较常见的创建流的方法，还有其他从Iterable来创建流的方法。

## 流的操作

创建了流之后，会对流进行操作，然后得到我们想要的流，或者转化为其他数据类型，比如，转化为集合，数组等。从使用的角度划分，流的操作可以分为中间操作和终止操作。中间操作相当于向流添加一个监听器，添加监听器时不会执行监听器。终止操作会对流进行迭代，执行监听器，返回结果。

一般，流的操作流程为：   
1）从集合创建一个流   
2）使用多个中间操作，得到相应的中间阶段的流。   
3）使用一个终止操作，得到一个结果。结果可能为一个int类型的结果，或者一个集合，不会是一个Stream类型了。   

### 中间操作

- Stream<T> filter(Predicate<? super T> predicate)

接受一个Predicate类型的lambda表达式，返回一个新的Stream。filter()会对每个元素进行迭代，当predicate返回true时，元素会保留在新的stream中；反之，元素不会保留在新的stream中。

```java
List<String> list = Lists.newArrayList("Chris Kate", "Bob Nana", "Tina Peter", "Bruce", "Jaker Alicia", null);
list.stream().filter(item -> StringUtils.isNotEmpty(item))  // 过滤null元素
```

- <R> Stream<R> map(Function<? super T, ? extends R> mapper)

接受一个mapper，mapper将流中的每个元素转换为相应的映射结果。返回一个新的流，新的流中的元素为转换后的结果。如下面的例子所示，先过滤null元素，然后使用 item -> item.length() 将流中的元素转换为字符串长度，加入新的流中。

```java
Stream<Integer> newStream = list.stream().filter(item -> StringUtils.isNotEmpty(item))
  .map(item -> item.length());
```

- <R> Stream<R> flatMap(Function<? super T, ? extends Stream<? extends R>> mapper)
  
flatMap()的作用也是将流元素转化为另一个元素，但是，mapper对每个元素进行处理后，需要返回一个流对象。因此，在使用场景方面，mapper处理的元素往往是一个“复合”型的元素，比如，元素类型为字符串、list、set，这样的元素可以得到一个流。

```java
List<String> list = Lists.newArrayList("Chris Kate", "Bob Nana", "Tina Peter", "Bruce", "Jaker Alicia", null);
list.stream().filter(item -> StringUtils.isNotEmpty(item))
  .flatMap(item -> Arrays.stream(item.split(" ")));
```

上面的代码中，mapper将字符串流中的每个元素，使用空格分隔为数组，然后将数组转化为流返回。这样，这条语句执行之后（实际执行阶段在调用终止操作的时候），流中的元素就变成 ("Chris", " Kate", "Bob", "Nana", "Tina", "Peter", "Bruce", "Jaker", "Alicia")

至于为什么起名叫flatMap()，本人觉得效果类似于将每个“复合“型的元素给压扁了。使用flat很形象。

- vStream<T> distinct()
  
distinct()的作用是将流中的元素去重。元素进行相等比较的时候，是调用了它们的equals()方法。

- Stream<T> sorted(Comparator<? super T> comparator)
  
sorted(comparator)的作用是将流中的元素进行排序。排序的依据是comparator。比如，下面的操作将元素按照字符串长度从小到大排序。

```java
list.stream().filter(item -> StringUtils.isNotEmpty(item))
                .sorted((s1, s2) -> s1.length() - s2.length())
```

- Stream<T> limit(long maxSize)
  
limit(maxSize)限制流的最大元素个数。比如，下面的操作取字符串长度最小的两个。
```java
list.stream().filter(item -> StringUtils.isNotEmpty(item))
                .sorted((s1, s2) -> s1.length() - s2.length()).limit(2)
```

> **注意**：中间操作只是添加一个监听器，不会实际执行监听器。监听器的执行时间在终止操作调用的时候。

中间操作可以分为两类：无状态(stateless)操作和有状态(stateful)操作

无状态操作是指，流里面每个元素的处理是相互独立的，不会受到前一个元素处理的影响。比如，filter()，map(), flatMap()，每个元素的处理不依赖于前一个元素的处理。

有状态操作是指，流里面每个元素的处理是依赖于其他元素的。比如，distinct()、sorted()、sorted(comparator)、limit()会依赖于流里面其他元素处理完，然后才可以执行去重操作、排序操作或者取前几个元素。

## 终止操作
终止操作会对流进行迭代，执行监听器，返回结果。返回的结果一般都是int等基本类型，或者一个集合类型，不会是一个Stream类型。因此，使用了终止操作之后，就没有办法继续对流进行下一步流操作了。

- boolean anyMatch(Predicate<? super T> predicate)
  
接受一个predicate。当流元素任意一个满足predicate条件判断时，则anyMatch()返回true；当流元素都不满足predicate条件判断时，anyMatch()返回false。

```java
List<String> list = Lists.newArrayList("Chris Kate", "Bob Nana", "Tina Peter", "Bruce", "Jaker Alicia", null);
list.stream().filter(item -> StringUtils.isNotEmpty(item))
      .anyMatch(item -> item.contains("Chris"));
```
上面的代码中，判断集合元素中是否有元素包含”Chris"。

- boolean allMatch(Predicate<? super T> predicate)
  
接受一个predicate。当流元素所有都满足predicate条件判断时，则allMatch()返回true；否则，返回false。

```java
list.stream().filter(item -> StringUtils.isNotEmpty(item))
     .allMatch(item -> !item.contains("dick"));
```
上面的操作，判断每个元素都不含有"dick"。

- boolean noneMatch(Predicate<? super T> predicate)
  
接受一个predicate。当流元素没有任何一个满足predicate条件判断时，则nonMatch()返回true；否则，返回false。

- Optional<T> findFirst()
  
获取流中的第一个元素。

```java
Optional<String> first = list.stream().filter(item -> StringUtils.isNotEmpty(item))
  .findFirst();
System.out.println(first.orElse(""));
```
- Optional<T> findAny()
  
获取流中的某个元素。

- <R, A> R collect(Collector<? super T, A, R> collector)
  
collect()用于将流中的元素收集到集合中。接受一个Collector类型的参数，Collector是java8提供的一个工具类，提供了很多获取lambda表达式（函数式操作）的方法。

```java
list.stream().filter(item -> StringUtils.isNotEmpty(item))
  .collect(Collectors.toList());
```
- long count()
  
统计流中的元素数量

- Optional<T> min(Comparator<? super T> comparator)
  
接收一个comparator比较器，返回流中最小的元素。

- Optional<T> max(Comparator<? super T> comparator)
  
接收一个comparator比较器，返回流中最大的元素。

- void forEach(Consumer<? super T> action)
  
对流中的每个元素进行迭代。action是一个操作，没有返回结果。

```java
list.stream().filter(item -> StringUtils.isNotEmpty(item))
  .forEach(item -> System.out.println(item));
```
- Optional<T> reduce(BinaryOperator<T> accumulator)
  
reduce()会对流中的元素进行迭代，最终结果为一个元素。至于对元素进行怎么样的处理，则看传入的参数accumulator是什么样的操作。

下面的例子展示了计算所有非空元素的长度的总和。
```java
Optional<Integer> reduce = list.stream().filter(item -> StringUtils.isNotEmpty(item))
        .map(item -> item.length()).reduce((i1, i2) -> i1 + i2);
System.out.println(reduce.orElse(0));
```
再比如，下面的例子展示了选取list中元素长度最小的那个元素
```java
Optional<String> reduce = list.stream().filter(item -> StringUtils.isNotEmpty(item))
  .reduce((s1, s2) -> s1.length() < s2.length() ? s1 : s2);
System.out.println(reduce.orElse(""));
```
- Object[] toArray()
  
用于将流转化为一个数组。

终止操作可以分为短路操作（short-circuiting）和非短路操作。

短路操作是指，得到预期的结果之后，就会返回结果，不会接着处理剩下的元素了。比如，anyMatch()、allMatch()、noneMatch()都是短路操作。

非短路操作是指，处理完所有数据才可以拿到结果。比如，collect()、count()、forEach()、max()、min()、reduce()、toArray()都是非短路操作，处理完所有元素才能拿到结果。

# stream vs foreach循环

遍历一个集合，对集合元素进行操作，既可以使用java8之前的for-each循环，也可以使用stream操作。如下面的代码所示。
```java
List<String> list = Lists.newArrayList("Chris Kate", "Bob Nana", "Tina Peter", "Bruce", "Jaker Alicia", null);
// 传统的使用for-each循环
int totalLen = 0;
for (String s : list) {
  if (StringUtils.isEmpty(s)) {
    continue;
  }
  totalLen += s.length();
}
System.out.println(totalLen);

// 使用java8流处理
long count = list.stream().filter(item -> StringUtils.isNotEmpty(item))
  .map(item -> item.length())
  .reduce((len1, len2) -> len1 + len2)
  .orElse(0);
System.out.println(count);
```
那么，它们各个方面有什么不同呢？知道它们的不同，可以让我们在合适的场景下选择合适的方式。

![for-each vs stream](http://ww1.sinaimg.cn/large/006y8mN6ly1g6av8rwmpwj31460dzdn8.jpg)

> **我们推荐**
- 1）当lambda表达式简洁时，可以使用stream流处理；当lambda表达式比较复杂时，则使用传统的for-each循环。
- 2）在一般情况下，不必为了一丁点儿的性能改善来使用stream流处理。
- 3）当需要访问外部变量时，则不要使用stream流处理，因为lambda表达式中只能访问final或者effectively final的变量。

# 参考
[Java Stream API](http://tutorials.jenkov.com/java-functional-programming/streams.html)   
[Streams 的幕后原理](https://www.ibm.com/developerworks/cn/java/j-java-streams-3-brian-goetz/index.html)   
[原来你是这样的 Stream —— 浅析 Java Stream 实现原理](http://hack.xingren.com/index.php/2018/10/17/java-stream/)   
[Java 8 Stream探秘](https://colobu.com/2014/11/18/Java-8-Stream/)    
[Java 8 Iterable.forEach() vs foreach loop](https://stackoverflow.com/questions/16635398/java-8-iterable-foreach-vs-foreach-loop)   
[10 Ways to Create a Stream in Java](https://www.geeksforgeeks.org/10-ways-to-create-a-stream-in-java/)   


