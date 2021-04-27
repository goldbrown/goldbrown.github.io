---
layout:     post
title:      异步编程学习3：使用Future和CompletableFuture来获取结果
subtitle:   
date:       2020-03-27
author:     Chris
header-img: img/post-bg-liuyifei.jpg
catalog: true
tags:
    - 异步
    - 线程池
    - Future
    - CompletableFuture
---


在计算图比较复杂的时候，如下图所示，任务之间存在相互依赖，即任务C依赖于任务A的执行结果。这时候，需要获取异步任务A的执行结果之后再执行任务C。一种方式是通过Future来获取，另一种方式是CompletableFuture
<img src="https://tva1.sinaimg.cn/large/00831rSTgy1gd9jg9zo0xj317i0ikgm3.jpg" height="80%" width="80%">


# 1 初步体验Future
如果要执行下面的计算图，
<img src="https://tva1.sinaimg.cn/large/00831rSTgy1gd9jh3k4k8j30lk0auq2x.jpg" height="80%" width="80%">

可以考虑将任务A在线程池中执行，然后在主线程中执行任务B。接着，获取任务A的执行结果，执行任务C。代码如下。

```java
public class FutureDemo {
    private final static int PROCESSOR = Runtime.getRuntime().availableProcessors();
    private final static ThreadPoolExecutor EXECUTOR = new ThreadPoolExecutor(PROCESSOR, PROCESSOR * 2, 1, TimeUnit.MINUTES,
            new LinkedBlockingQueue<>(5), new ThreadPoolExecutor.CallerRunsPolicy());
    public static void main(String[] args) throws ExecutionException, InterruptedException {
        long start = System.currentTimeMillis();
        // 将Callable类型的任务包装成一个FutureTask。可以提交FutureTask任务到线程池，然后从FutureTask获取执行结果。
        FutureTask<String> futureTaskA = new FutureTask<>(() -> {
            String result = doSomethingA();
            return result;
        });
        // 提交FutureTask任务到线程池
        EXECUTOR.execute(futureTaskA);
        doSomethingB();
        // 从FutureTask获取执行结果。这里会阻塞获取结果
        String resultA = futureTaskA.get();
        doSomethingC(resultA);
        System.out.println(System.currentTimeMillis() -start);
    }

    private static String doSomethingB() {
        try {
            Thread.sleep(2000);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        System.out.println("---doSomethingB---");
        return "doSomethingB_Result";
    }

    private static String doSomethingC(String resultA) {
        try {
            Thread.sleep(2000);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        System.out.println("---get resultA:" + resultA + ",doSomethingC---");
        return "doSomethingC_Result";
    }

    private static String doSomethingA() {
        try {
            Thread.sleep(2000);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        System.out.println("---doSomethingA---");
        return "doSomethingA_Result";
    }
}
```

# 2 FutureTask简介

FutureTask的类图如下   
<img src="https://tva1.sinaimg.cn/large/00831rSTgy1gd9jhua2z1j30eo0c2aa1.jpg" height="50%" width="50%">

* Runnable：接口类，实现该接口的类会被当做一个任务类，可以被线程池执行。该任务的执行没有返回结果。
* Future：接口类，代表异步任务的执行结果。可以通过isDone()来查询异步任务是否完成，通过get()和get(long timeout, TimeUnit unit)来获取任务的执行结果。其方法如下：   
    <img src="https://tva1.sinaimg.cn/large/00831rSTgy1gd9jiaxc8oj30aw07qaa5.jpg" height="50%" width="50%">
    
* RunnableFuture：接口类，同时实现了Runnable和Future。因此，该类代表了一个任务，同时，代表一个异步任务的执行结果。实现该接口的类，可以提交给线程池执行，也可以从该类获取结果。
* FutureTask：实现了RunnableFuture，代表一个可被取消的异步计算任务，同时提供了启动和取消任务的方法、查询任务是否完成的方法和获取计算结果的方法。其方法如下：   
    <img src="https://tva1.sinaimg.cn/large/00831rSTgy1gd9jk3ac4lj30cu0ds0t5.jpg" height="50%" width="50%">
    
    
# 3 FutureTask在异步编程上的局限
* FutureTask提供了get()方法来获取返回结果，但只能阻塞获取。
* 对于复杂的计算图，例如多个任务的情况且任务之间存在依赖关系，FutureTask无法清楚表达计算图的逻辑，换句话说，编排任务太困难。即使能够实现，代码也会很复杂。

为了解决FutureTask难以编排多个异步任务编排的问题，java8提供了CompletableFuture。

# 4 初步体验CompletableFuture
为了完成下面的计算图，使用CompletableFuture可以这么做。   
<img src="https://tva1.sinaimg.cn/large/00831rSTgy1gd9jl0b2t1j31340jkmxj.jpg" height="50%" width="50%">

```java
public class CompletableFutureDemo {
    public static void main(String[] args) throws ExecutionException, InterruptedException {
        long start = System.currentTimeMillis();
        // 在线程池中，先执行任务A；得到任务A的结果之后再执行任务C。thenCompose()用来实现当一个CompletableFuture执行完毕之后，执行另外一个CompletableFuture任务。
        CompletableFuture<String> resultC = doSomethingA("paramA").thenCompose((id -> doSomethingC(id)));
        // 在线程池中，执行任务B
        CompletableFuture<String> resultB = doSomethingB("paramB");
        // 阻塞获取任务C和任务B的结果
        System.out.println(resultC.get());
        System.out.println(resultB.get());
        // 这里的耗时比4000ms多一些，大约为任务A和任务C的执行总耗时
        System.out.println(System.currentTimeMillis() - start);
    }

    private static CompletableFuture<String> doSomethingC(String id) {
        // CompletableFuture.supplyAsync()用来实现有返回值的异步计算
        return CompletableFuture.supplyAsync(() -> {
            try {
                // 模拟执行耗时
                Thread.sleep(2000);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            return "resultC of " + id;
        });
    }

    private static CompletableFuture<String> doSomethingB(String id) {
        return CompletableFuture.supplyAsync(() -> {
            try {
                Thread.sleep(2000);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            return "resultB of " + id;
        });
    }

    private static CompletableFuture<String> doSomethingA(String id) {
        return CompletableFuture.supplyAsync(() -> {
            try {
                Thread.sleep(2000);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            return "resultA of " + id;
        });
    }
}
```
从上面的代码中可以看出，CompletableFuture用来表达有/无返回值的异步计算和编排计算图很方便。值得注意的是，默认情况下supplyAsync(Supplier< U> supplier)在ForkJoinPool.commonPool()线程池中执行。如果想要在自定义的线程池中执行任务，可以使用supplyAsync(Supplier< U> supplier,Executor executor)方法。



# 5 CompletableFuture使用简介

## 类图介绍
CompletableFuture的类图如下：  
<img src="https://tva1.sinaimg.cn/large/00831rSTgy1gd9jm30ksdj30fu06yt8o.jpg" height="80%" width="80%">

* Future：接口类，代表异步任务的执行结果。可以通过isDone()来查询异步任务是否完成，通过get()和get(long timeout, TimeUnit unit)来获取任务的执行结果。其方法如下：   
    <img src="https://tva1.sinaimg.cn/large/00831rSTgy1gd9jmzdjcoj30aw07qaa5.jpg" height="50%" width="50%">
* CompletionStage：一个CompletionStage代表一个异步计算节点，当其他的计算节点完成之后执行某个操作或者计算某个值；该节点计算完成之后，也可能会触发其他的计算节点。例如，下图中的CompletionStage B，依赖于CompletionStage A执行完成；CompletionStage B执行完成之后，触发CompletionStage C的执行。   
    <img src="https://tva1.sinaimg.cn/large/00831rSTgy1gd9q0pswuaj30vm05wmx7.jpg" height="80%" width="80%">
    
    * 每个计算节点的执行
        * 一个节点的计算可以由Function来表示，由apply()方法来代表，接收一个参数，返回一个结果；
        * 或者一个节点的计算可以由Consumer来表示，由accept()代表，接收一个参数，无返回结果；
        * 或者一个节点的计算可以由Runnable来表示，由run()来代表，无参数，无返回结果。
        例如，`stage.thenApply(x -> square(x)).thenAccept(x -> System.out.print(x)).thenRun(() -> System.out.println())`
    * stage的触发
        * stage触发可以由其他某个stage完成引起，编排使用的方法一般带有前缀`then`，例如`thenAccept`和`thenApply`。
        * 或者两个stage都完成后(and)触发，编排使用的方法一般带有`both`或者`combine`，例如`thenAcceptBoth`。
        * 或者两个stage某个完成后(or)触发，编排使用的方法一般带有`either`，例如`acceptEither`。
    * 下一个计算节点的执行方式
        * 计算节点可以采用`默认的执行方式`。不含`async`后缀的所有方法都将以这种方式执行，该类方法会阻塞等待前一个计算节点完成。执行属性由CompletionStage的实现类决定。
        * 默认的异步执行。含有`async`后缀且没有Executor参数的所有方法都将以这种方式执行。执行属性由CompletionStage的实现类决定。
        * 自定义执行方式。含有`async`后缀且含有Executor参数的所有方法都将以这种方式执行。执行属性由传入的Executor决定。
    * 异常处理
        * 无论上一个计算节点是否正常完成，有如下两个方法总是可以正常执行：
            * whenComplete：它要求接收上一个计算节点的执行节点，是一个消费型的节点，由方法定义`CompletionStage<T> whenComplete (BiConsumer<? super T, ? super Throwable> action);`可以看出。因此，计算逻辑action不对结果产生影响，会将它接收的结果传递给下一个节点。
            * handle前缀的方法：它是一个函数型的节点，见方法定义`public <U> CompletionStage<U> handle (BiFunction<? super T, Throwable, ? extends U> fn)`。因此，它可以对结果产生影响。

* CompletableFuture：实现了Future，因此具有Future的行为，可以阻塞获取结果，此外可以通过设置结果和状态来显式的改变Future状态，例如Future任务还没有执行完，但我们可以通过`complete(T value)`方法来显式的让Future任务结束并设置返回结果；实现了CompletionStage，因此，可以当做计算节点，可以用来编排任务。
    CompletableFuture实现了CompletionStage的这些特性：
    * 不含`async`的方法的执行，可能在当前future的线程中执行，或者在调用者的线程中执行。
        例如，下面的代码所示。
        
        ```java
        CompletableFuture<Integer> future = CompletableFuture.supplyAsync(() -> {
                    // 这里的线程名是ForkJoinPool.commonPool-worker-1
                    System.out.println(Thread.currentThread().getName() + " handle async task");
                    return 2;
                });
                future.thenAccept(x -> {
                    try {
                        Thread.sleep(1000);
                    } catch (InterruptedException e) {
                        e.printStackTrace();
                    }
                    // 这里的线程名是main（主线程）
                    System.out.println(Thread.currentThread().getName() + " get result of future:" + x);
                    
                });
        ```
    * 含有`async`且没有形参Executor的方法的执行，会在ForkJoinPool.commonPool()线程池中执行。
    
    
## 常用方法介绍

### 用于计算的方法

| 方法 |描述  |
| --- | --- |
| runAsync | 实现无参数、无返回值的异步计算 |
| supplyAsync | 实现无参数、有返回值的异步计算 |

### 用于编排任务的方法

| 方法 |描述  |
| --- | --- |
| thenRun | 任务A执行完成之后，触发任务B。任务B无法拿到任务A的执行结果。 |
| thenAccept | 任务A执行完成之后，触发任务B。任务B能够拿到任务A的执行结果。 |
| thenApply | 任务A执行完成之后，触发任务B。任务B能够拿到任务A的执行结果，且任务B会返回计算结果。 |
| whenComplete | 无论上一个计算节点是否，总会执行whenComplete设置的任务。 |

### 编排多个CompletableFuture的方法

| 方法 |描述  |
| --- | --- |
| thenCompose |CompletableFuture A执行完之后，执行CompletableFuture B。  |
| thenCombine |CompletableFuture A和B**都**执行完之后，执行CompletableFuture C。  |
| allOf |等待多个CompletableFuture**都**执行完之后，执行另一个CompletableFuture。  |
| anyOf |等待多个CompletableFuture**某一个**执行完之后，执行另一个CompletableFuture。  |


# 6 使用CompletableFuture编排复杂的任务
接下来，我们使用CompletableFuture来编排下面的计算图。
<img src="https://tva1.sinaimg.cn/large/00831rSTgy1gd9q35dokbj311e0gcq3a.jpg" height="80%" width="80%">

代码如下
```java
public class DependencyCompletableFutureDemo {
    public static void main(String[] args) throws ExecutionException, InterruptedException {
        long start = System.currentTimeMillis();
        // 创建异步任务A
        CompletableFuture<String> futureA = doSomethingA("A");
        // 创建异步任务C，且依赖于A执行完成
        CompletableFuture<String> futureC = futureA.thenCompose((id) -> doSomethingC(id));
        // 创建异步任务D，且依赖于A执行完成
        CompletableFuture<String> futureD = futureA.thenCompose((id) -> doSomethingD(id));
        // 创建异步任务B和E，且E依赖于B执行完成
        CompletableFuture<String> futureE = doSomethingB("B").thenCompose((id) -> doSomethingE(id));
        // 创建异步任务F，且F依赖于C,D,E完成
        CompletableFuture<String> futureF = CompletableFuture.allOf(futureC, futureD, futureE).thenCompose((id) -> doSomethingF());
        // 这里阻塞，获取F的结果
        futureF.get();
        // 下面的打印时间大约为3000ms，为计算节点最多的链路所花费的时间
        System.out.println(System.currentTimeMillis() - start);
        Thread.currentThread().join();
    }

    private static CompletableFuture<String> doSomethingA(String id) {
        // 使用CompletableFuture.supplyAsync生成一个异步任务，需要返回结果
        return CompletableFuture.supplyAsync(() -> {
            try {
                Thread.sleep(1000);
                System.out.println(Thread.currentThread().getName() + " doSomethingA");
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            return "result of param " + id;
        });
    }
    private static CompletableFuture<String> doSomethingB(String id) {
        return CompletableFuture.supplyAsync(() -> {
            try {
                Thread.sleep(1000);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            System.out.println(Thread.currentThread().getName() + " doSomethingB");
            return "result of param " + id;
        });
    }
    private static CompletableFuture<String> doSomethingC(String id) {
        return CompletableFuture.supplyAsync(() -> {
            try {
                Thread.sleep(1000);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            System.out.println(Thread.currentThread().getName() + " doSomethingC");
            return "result of param " + id;
        });
    }
    private static CompletableFuture<String> doSomethingD(String id) {
        return CompletableFuture.supplyAsync(() -> {
            try {
                Thread.sleep(1000);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            System.out.println(Thread.currentThread().getName() + " doSomethingD");
            return "result of param " + id;
        });
    }
    private static CompletableFuture<String> doSomethingE(String id) {
        return CompletableFuture.supplyAsync(() -> {
            try {
                Thread.sleep(1000);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            System.out.println(Thread.currentThread().getName() + " doSomethingE");
            return "result of param " + id;
        });
    }
    private static CompletableFuture<String> doSomethingF() {
        return CompletableFuture.supplyAsync(() -> {
            try {
                Thread.sleep(1000);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            System.out.println(Thread.currentThread().getName() + " doSomethingF");
            return "result of F";
        });
    }
}
```

其中一种结果如下
```
ForkJoinPool.commonPool-worker-2 doSomethingB
ForkJoinPool.commonPool-worker-1 doSomethingA
ForkJoinPool.commonPool-worker-3 doSomethingE
ForkJoinPool.commonPool-worker-1 doSomethingC
ForkJoinPool.commonPool-worker-2 doSomethingD
ForkJoinPool.commonPool-worker-2 doSomethingF
3068
```
在上面的结果中，F总是在最后一个执行完成，总时间比3000ms多一些。

# 7 总结
使用Future可以获取任务的执行结果，从而编排简单的任务依赖关系；使用CompletableFutre，不需要显式的创建线程池，而且，编排任务变得更加轻松，可以编排依赖关系较复杂的任务。

# 8 参考    
[Java并发包之阶段执行之CompletionStage接口](https://www.cnblogs.com/txmfz/p/11266411.html)   
[Interface CompletionStage](https://docs.oracle.com/javase/8/docs/api/java/util/concurrent/CompletionStage.html)   