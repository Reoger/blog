---
title: java中的泛型
date: 2018-08-15 14:44:55
categories: java
tags: android,java,T,泛型
---

# 什么是泛型
Java 泛型（generics）是 JDK 5 中引入的一个新特性, 泛型提供了编译时类型安全检测机制，该机制允许程序员在编译时检测到非法的类型。
泛型的本质是参数化类型，也就是说所操作的数据类型被指定为一个参数。
---
假定我们有这样一个需求：写一个排序方法，能够对整型数组、字符串数组甚至其他任何类型的数组进行排序，该如何实现？

答案是可以使用 Java 泛型。

使用 Java 泛型的概念，我们可以写一个泛型方法来对一个对象数组排序。然后，调用该泛型方法来对整型数组、浮点数数组、字符串数组等进行排序。
# 泛型的分类
根据泛型的使用场景，一般可以分成三类，分别是:**泛型类**、**泛型方法**、**和泛型接口**。

# 泛型类型擦出
Java中的泛型，只在编译阶段有效。在编译过程中，正确检验泛型结果后，会将泛型的相关信息擦出，并且在对象进入和离开方法的边界处添加类型检查和类型转换的方法。也就是说，泛型信息不会进入到运行时阶段。
用代码来说明就是：
```
public class test {

    public static void main(String args[]){
        List<Integer> a = new ArrayList<>();
        List<String>  b = new ArrayList<>();
        System.out.println(a.getClass().toString());
        System.out.println(a.getClass().toString());

        System.out.println(b.getClass().equals(a.getClass()));
    }
}

```
运行的结果如下：
```
class java.util.ArrayList
class java.util.ArrayList
true
```
可以看到，``List<Integer>``和`` List<String>``的类是相同的。

总结成一句话就是：**泛型类型在逻辑上看以看成是多个不同的类型，实际上都是相同的基本类型。**
## 泛型类
泛型类的使用非常简单,在类名添加``<T>`` 或者类似于这类的泛型类声明即可。
```
class Person<T>{
    private T t;

    public Person(T t) {
        this.t = t;
    }

    public T getKey(){
        return t;
    }
}
```
我们在使用这个泛型类的时候，就可以传入不同类型的参数来进行初始化，可以显示声明数据类型，也可以不声明，``jvm``会自动帮我们声明。使用实例代码如下：
```
Person p1 = new Person<Integer>(12345);
Person p2 = new Person<Double>(8989.9);
Person p3 = new Person<>(true);
Person p4 = new Person<>("i am ok!");

System.out.println("p1 -> "+p1.getKey());
System.out.println("p2 -> "+p2.getKey());
System.out.println("p3 -> "+p3.getKey());
System.out.println("p4 -> "+p4.getKey());
```
输出的结果如下：
```
p1 -> 12345
p2 -> 8989.9
p3 -> true
p4 -> i am ok!
```
可以看到，使用泛型类非常简单，只需要在定义类的时候添加``<T>``、``<K>``、``<V>``这类的泛型类的定义即可。当然，如果有需要，我们可以限制其使用范围，例如``<T extends Number>`` 就规定了传入的类型是只能是``Number``的子类。

## 泛型方法
泛型方法的规则如下：
1. 所有泛型方法声明都有一个类型参数声明部分（由尖括号分隔），该类型参数声明部分在方法返回类型之前（在下面例子中的<E>）。
2. 每一个类型参数声明部分包含一个或多个类型参数，参数间用逗号隔开。一个泛型参数，也被称为一个类型变量，是用于指定一个泛型类型名称的标识符。
3. 类型参数能被用来声明返回值类型，并且能作为泛型方法得到的实际参数类型的占位符。
4. 泛型方法体的声明和其他方法一样。注意类型参数只能代表引用型类型，不能是原始类型（像int,double,char的等）。

泛型方法与泛型类的定义类似，泛型类的定义是在类名后加``<T>``这类的泛型修饰词，泛型方法的定义就是在方法的返回值添加``<T>``这类的修饰词。先来看泛型方法的定义示例：
```
 private <T> void helloPerson(T t){
        System.out.println("t = [" + t + "]");
    }
```
注意，这里我们称他为泛型方法是因为``void``前面的``<T>``，而不是``helloPerson(T t)``中的``T``。
我们在泛型类的``Person``中的``public T getKey()`` 就不能算一个泛型方法，因为这里的``T``只是``getKey``方法的一个返回值而已。
然后泛型方法的使用也很简单：
```
helloPerson(123);
helloPerson(12.3);
helloPerson(true);
helloPerson("i am ok");
```
上面方法的运行结果如下：
```
t = [123]
t = [12.3]
t = [true]
t = [i am ok]
```


## 泛型接口
泛型接口的定义同样很简单，只需要在接口的名字后加上``<T>``这类的泛型定义接口。例如下面是一个泛型接口的示例代码：
```
public interface IFunction<T> {
    void doSomeThing(T t);
}
```
在泛型接口里面定义了泛型，我们就可以在对应的方法中使用其类型。
当然，接口的定义好之后还是需要去实现这个接口的，实现的方式有两种，一种是声明类型，一种是不声明类型。
下面是第一种，申明类型的实现：
```
public class Eat implements IFunction<String> {

    @Override
    public void doSomeThing(String s) {
        System.out.println("s = [" + s + "]");
    }
}
```
可以看到，在``Eat``的实现中，我们讲泛型显示声明成了``String``，如此我们在使用``Eat``类中的``doSomeThing``方法时，就只能传入``String``类型的。
另一种，是不声明具体类型的实现：
```
public class Read<T> implements IFunction<T> {

    @Override
    public void doSomeThing(T t) {
        System.out.println("t = [" + t + "]");
    }
}
```
可以看到，其实``Read``就已经被声明成了一个泛型类，因为实现了泛型接口，我们要继续使用泛型来实现的话，就需要将类实现成泛型类。
我们来看其调用：
```
IFunction f1 = new Eat();
IFunction f2 = new Read();

f1.doSomeThing("f1-> only eat!");
//f1.doSomeThing(123456);
//这句编译不会报错，运行会抛出java.lang.Integer cannot be cast to java.lang.String 异常

f2.doSomeThing(132);
f2.doSomeThing("read");
f2.doSomeThing(true);
f2.doSomeThing(52.1);
```
输出的结果如下：
```
s = [f1-> only eat!]
t = [132]
t = [read]
t = [true]
t = [52.1]
```
可以看到，我们在继承泛型接口时，显示申明了其类型之后，我们调用时也只能使用对应的类型，如果传入其他的类型，编译器并不会报错，但是运行时会抛出 类型强转失败的异常 。



# 泛型通配符
类型通配符一般是使用``?``代替具体的类型参数。例如 List<?> 在逻辑上是List<String>,List<Integer> 等所有List<具体类型实参>的父类。注意，``?``可以代表``String``,``Interger``等其他具体的类型，但它是不确定的，而``<T>``表示一个类型参数，不能当作实际参数使用。说白了，就是``?``,表示不确定的类型，和``String``，``Integer``这种类型一样，是实实在在的类型，而``T``，只是表示类型参数，在使用时还是会转化成对应的实际参数的。
使用实例：
```

public class TypeTest {
    public static void main(String args[]){
        TypeTest typeTest  = new TypeTest();
        typeTest.helloMan(Arrays.asList("1","hello"));
        typeTest.helloMan(Arrays.asList(1,2,3));

    }

    private void helloMan(List<?> a){
        System.out.println("a = [" + a + "]");
    }
}
```
可以看到，我往同一个方法出入不同类型的参数也时可以正常运行的，输出结果为：
```
a = [[1, hello]]
a = [[1, 2, 3]]
```
还有，补充两点;
1. ``<? extends T>``表示该通配符所代表的类型是``T``类型的子类。
2. ``<? super T>``表示该通配符所代表的类型是``T``类型的父类

# 参考链接
* [java 泛型详解](https://blog.csdn.net/s10461/article/details/53941091)
* [Java 泛型](http://www.runoob.com/java/java-generics.html)