---
layout:     post
title:      java8新特性之lambda表达式
subtitle:   
date:       2019-08-18
author:     Chris
header-img: img/post-bg-nezha.jpeg
catalog: true
tags:
    - java
    - java8
    - lambda表达式
---


# lambda表达式

## 什么是lambda表达式
lambda表达式定义了一种操作，传入数据，就可以返回一个结果，类似于一个函数。   
定义：形式如    

```java
(parameters) -> { expression }
```
这样的，就是一个lambda表达式，只需要写参数列表和函数体。例如，

```java
(int a, int b) -> { a + b }
```
其中，有些情况下，为了简洁，可以省略一些部分。
- 参数的类型：不需要声明参数列表中参数的类型，编译器可以自动识别。例如，(a, b) -> { a + b }
- 参数的圆括号：当参数列表只有一个参数时，可以省略圆括号；但有零个或者多个参数时，不可以省略圆括号。例如，a -> { a + 1 }， (a, b) -> { a + b }
- 函数体中的大括号：如果函数体只包含一条语句，则可以省略大括号。例如， (a, b) -> a + b
- 函数体的返回值：如果主体只有一个表达式返回值则编译器会自动返回值。例如，(a, b) -> a + b返回a和b的和。

## 为什么使用lambda表达式

1）lambda表达式的引入，是为了更加方便开发者书写简洁的代码，来定义某种操作。   
例如，在java8之前，如果我们要将操作A传入某个方法m，那么必须将这个操作A封装成对象o再传递进去，这样很繁琐。那么，在java8中就可以直接将这个操作A写成一个lambda表达式传入那个方法m。这样很简洁。例如，排序的操作，在java8之前需要使用一个匿名对象来封装这个操作，在java8中就可以传入一个lambda表达式。

```java
// in java7
List<Integer> ages = Lists.newArrayList(24, 35, 29, 22, 23, 23);
ages.sort(new Comparator<Integer>() {
  @Override
  public int compare(Integer o1, Integer o2) {
    return o1 - o2;
  }
});

// in java8。代码行数一下子变为了两行
List<Integer> ages = Lists.newArrayList(24, 35, 29, 22, 23, 23);
ages.sort((o1, o2) -> o1 - o2);
```
2）高阶函数的引入，允许用户书写更加灵活的代码。

可以使用lambda表达式来代替匿名对象，从而做到更加灵活的复用代码。

## lambda表达式的实现

那么，怎么实现lambda表达式呢？lambda表达式理论上只是一个操作，一个函数。但是，在java中没有函数这个类型，java的设计者不想再引入一个函数这样的类型，于是，提出了函数式接口的概念，即符合某个条件的接口（函数式接口）可以接受lambda表达式。

### 函数式接口

函数式接口：有且只有一个抽象方法的接口为函数式接口。那么，可以得到如下特性：   
1）函数式接口有且只有一个抽象方法。(函数式接口中可以额外定义多个抽象方法，但这些抽象方法签名必须和Object的public方法一样)   
这里要求只有一个抽象方法，是为了让编译器可以自动将这个抽象方法和lambda表达式对应起来。

接口上可以通过注解@FunctionalInterface来表明这个是一个函数式接口，更加清晰明确的表明某接口是函数式接口，告诉后面维护的人不要随意添加抽象方法，编译器在编译时会检查接口里面是否真的只有一个抽象方法。若有多个抽象方法，则编译报错。不加注解@FunctionalInterface也是允许的，但是，会给后面维护的人带来麻烦。

例如，java8中的Comparator接口，除了Object类的equals(Object)方法外，就只有一个抽象方法int compare(T, T)。那么Comparator就是一个函数式接口。我们可以将lambda表达式赋给Comparator。lambda表达式对应的就是图中的抽象方法int compare(T, T)

![](http://ww1.sinaimg.cn/large/006tNc79gy1g63to3r9rdj30ja0litcr.jpg)


```java
List<Integer> list = Lists.newArrayList(1, 5, 2, 9, 3);
list.sort((a, b) -> a - b);  // list.sort(Comparator)需要接受一个Comparator对象，这里传入了一个lambda表达式。
```
2）函数式接口也是一个接口，仍然遵循接口的其他特性，比如，允许有static属性、static方法和默认方法。

下面再看一个例子。

首先声明一个函数式接口Foo。它具有一个抽象方法compute(int a, int b)，接收两个参数，然后返回一个计算结果。
```java
@FunctionalInterface
public interface Foo {
    int compute(int a, int b);
}
```
然后，定义一个使用该函数式接口的类Bar，someOperation(Foo foo, int x, int y)接受一个函数式接口（操作）和变量x和y（数据），然后返回操作结果。如下所示，代码打印出2和0。
```java
public class Bar {
    int someOperation(Foo foo, int x, int y) {
        return foo.compute(x, y);
    }

    public static void main(String[] args) {
        Foo foo = (x, y) -> x * y;  // 定义一个乘法操作
        result = new Bar().someOperation(foo, 1, 2);
        System.out.println(result);
        Foo foo1 = (x, y) -> x / y;  // 定义一个整除操作
        result = new Bar().someOperation(foo1, 1, 2);
        System.out.println(result);
    }
}
```
如代码中所示，someOperation(Foo foo, int x, int y)拥有一个函数式接口类型的参数，因此可以接受lambda表达式作为参数。lambda表达式的具体内容，是由调用方来决定的，因此，这样，就提高了调用方的灵活性。类似的，排序时所用到的Comparator是一个函数式接口，怎么排序由用户来决定。这有点儿类似于策略模式。

### 从字节码看lambda表达式

lambda表达式的实现，并没有采用匿名内部类的方式，虽然匿名内部类可以用来实现lambda表达式。但是，jvm的实现者为了更好的实现lambda表达式的灵活和高扩展性，采用了java7引入的invokedynamic指令。

1）编译期

lambda表达式在编译为字节码之后，就变成了一个invokedynamic指令。invokedynamic指令会链接到实际的函数式接口的对应的抽象方法所在的地方。如下面的例子所示。

源码如下：
```java
public class LambdaDemo {
    public static void main(String[] args) {
        int i = 1;
        Counter counter = a -> a + 1;
    }
}

@FunctionalInterface
public interface Counter {
    int incr(int a);
}
```
翻译后的字节码如下：
```java
public class com.chris.com.chris.service.java8.LambdaDemo {
  public com.chris.com.chris.service.java8.LambdaDemo();
    Code:
       0: aload_0
       1: invokespecial #1                  // Method java/lang/Object."<init>":()V
       4: return

  public static void main(java.lang.String[]);
    Code:
       0: iconst_1
       1: istore_1
       2: invokedynamic #2,  0              // InvokeDynamic #0:incr:()Lcom/chris/com/chris/service/java8/Counter;
       7: astore_2
       8: return
}
```

2）运行期

在代码运行时，当遇到lambda表达式时，根据invokedynamic指令指向的接口，构建一段可执行的代码，然后，执行代码。

## 匿名内部类和lambda表达式

使用匿名内部类指定数组的排序顺序。
```java
List<Integer> ages = Lists.newArrayList(24, 35, 29, 22, 23, 23);
ages.sort(new Comparator<Integer>() {
  @Override
  public int compare(Integer o1, Integer o2) {
    return o1 - o2;
  }
});
```
对于只有一个抽象方法的接口，可以使用lambda表达式来代替。如下所示。
```java
List<Integer> ages = Lists.newArrayList(24, 35, 29, 22, 23, 23);
ages.sort((o1, o2) -> o1 - o2);
```
匿名内部类看起来和lambda表达式很像，可能很多人为认为lambda表达式是匿名内部类的语法糖，但实际上并不是。

匿名内部类和lambda表达式的比较
![lambda vs anonymous inner class](http://ww4.sinaimg.cn/large/006y8mN6ly1g6avl89oykj315u0lkgtj.jpg)

effectively final的解释是，如果一个变量没有声明为final的，但是，在初始化之后都没有改变过，那么就是effectively final（相当于final）的变量。
```java
public static void main(String[] args) {
        int i = 1;
        Counter counter = a -> a + i;  // 这一行编译会报错，因为，i++在第6行已经被改变了，不是effectively final的变量
        i++;
}

@FunctionalInterface
interface Counter {
    int incr(int a);
}
```
在idea IDE中，在使用匿名内部类的地方，如果可以替换为函数式接口，则IDE会有提示。

## 闭包和lambda表达式

lambda表达式是可以算作是一种闭包。函数及其持有的外部变量，就是闭包。闭包的英文是closure，有着封闭的意思，即函数内部如果持有外部变量，则会持有一份副本。外部变量状态的改变不影响函数所在环境（闭包）的状态。

看一个关于闭包的例子，js写的关于闭包的例子。
```js
function makeAdder(x) {
  return function(y) {
    return x + y;
  };
}

var add5 = makeAdder(5);
var add10 = makeAdder(10);

console.log(add5(2));  // 7
console.log(add10(2)); // 12
```
makeAdder(x)中返回了一个函数，返回的函数使用了函数外部的变量x。var add5 = makeAdder(5);调用之后，理论上，函数makeAddr()的局部变量x，应该不能被访问了，因为此时函数makeAddr()已经返回结果了。但是，因为有闭包，所以，函数add5()仍然可以访问局部变量x。

再回顾一下闭包的定义，函数以及其持有的外部环境变量。其实，这样就是将函数以及所需要的数据关联起来了，类似于面向对象编程。在面向对象编程中，对象允许我们将某些数据（对象的属性）与一个或者多个方法相关联。

但是，在java中，在闭包内部只能访问final或者effectively final的变量。并不能像js那样访问任何变量。

## 方法引用

先看一个例子。对Person对象，根据age字段来从小到大排序。
```java
@Data
@AllArgsConstructor
@NoArgsConstructor
class Person {
    private String name;
    private int age;
  
   static int compareByAge(Person p1, Person p2) {
        return p1.getAge() - p2.getAge();
    }
}

public class LambdaDemo {
    public static void main(String[] args) {
        List<Person> personList = Lists.newArrayList(
                new Person("chris", 12),
                new Person("bob", 19),
                new Person("Tina", 17));
				// 没有使用已经存在的排序方法
        personList.sort((o1, o2) -> o1.getAge() - o2.getAge());
        System.out.println(personList);
    }
}
```
上面的Lambda表达式还可以化简。如下代码所示。
```java
...
// 利用已经存在的比较方法
personList.sort((p1, p2) -> Person.compareByAge(p1, p2));
...

...
// 利用已经存在的比较方法
personList.sort(Person::compareByAge);
...
```
上面的代码使用了所谓的方法引用。该方法引用，利用了已经存在的方法。Person::compareByAge就相当于一个lambda表达式，类似于(p1, p2) -> Person.compareByAge(p1, p2)。是lambda表达式的语法糖。

### 什么时候使用方法引用代替lambda表达式

1）当lambda表达式里面只是调用了对象（类）的某个方法

例如，上面的代码中，lambda表达式里面只是使用 了compareByAge方法，那么，就可以直接使用方法引用。即使用 Person::compareByAge。

IDE可以自动将满足条件的lambda表达式转化为方法引用。

## lambda最佳实践

1）最好保持lambda表达式的简短，最好为个位数的行数。

2）若有太多行，比如10行，则考虑抽出一个方法进行封装。

# 参考
- [Lambda Expressions and Functional Interfaces: Tips and Best Practices](https://www.baeldung.com/java-8-lambda-expressions-tips)
- [Java 8函数式接口functional interface的秘密](https://colobu.com/2014/10/28/secrets-of-java-8-functional-interface/)
- [Java8新特性第1章(Lambda表达式)](https://zhuanlan.zhihu.com/p/20540175)
- [深入探索 Java 8 Lambda 表达式](https://www.infoq.cn/article/Java-8-Lambdas-A-Peek-Under-the-Hood/)
- [https://www.ibm.com/developerworks/cn/linux/l-cn-closure/](https://www.ibm.com/developerworks/cn/linux/l-cn-closure/)
- [What Lambda Expressions are compiled to? What is their runtime behavior?](https://www.logicbig.com/tutorials/core-java-tutorial/java-8-enhancements/java-lambda-functional-aspect.html)

