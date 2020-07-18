---
title: java中的反射
date: 2018-08-15 14:44:55
categories: java 
tags: [android,reflection]
---

# java中的反射
在java中，有一种可以跨越常规语法的用法，它就是反射。通过反射，我们可以访问到我们原本访问不到的类和对象，也不能调用我们原本调用不到的方法。虽然反射很强大，但是它违反了程序设计的初衷，且用反射的性能会有一定的影响，所以我们应该避免使用反射。但是反射功能实在强大，正确的使用能使得我们的代码变得很简洁，所以我们还是有必要学习一下反射的相关知识。

# 反射的原理
[深入分析Java方法反射的实现原理](http://www.importnew.com/23902.html)

## 获取类实例
获取类实例一般有两种写法，一种是``类名.class``,另一种则是``class.forName()``。假如我们已经有了一个``Person``的类，那么我们可以通过如下三种方式来获取类实例。
```
//方式1
Class person1 = Person.class;
//方式2
Class person2 = Class.forName("day18.Person");
//方式3,如果能用这种肯定用这种！
Person person3 = new Person();
```
两种方式的区别是，一种是我们能直接拿到那个类对象，另一种是我们无法中拿到类对象，必须通过反射才能获取到类对象。

## 通过反射获取构造方法
假设我们有这个一个``Person``类，代码如下：
```
class Person {
    public String name;
    public char sex;
    public int height;
    protected float id;
    private String des;

    private Person() {
        this.name = "jone";
        this.sex = 'f';
        this.height = 155;
        this.id = 420458188603268922F;
        this.des = "just a normal person";
    }

    private Person(String name) {
        this.name = name;
    }

    private Person(String name, char sex) {
        this.name = name;
        this.sex = sex;
    }

    private Person(String name, char sex, int height) {
        this.name = name;
        this.sex = sex;
        this.height = height;
    }
}
```
那么我们如何去创建对应的``Person``对象呢，不能因为我们没有对象就不创建对象呀！答案是我们可以利用反射获得对应的构造方法，然后利用``newInstance()``方法，即可创建对应的``Person``对象。例如我们可以这么写，来获得对应的构造方法：
```
 try {
    //获取对应的类对象，如果直接获取不到，可以用Class.forName("packageName.Person"); 来获取
    Class person1 = Person.class;
    //获取无参的构造方法
    Constructor constructor = person1.getDeclaredConstructor();
    //设置属性为可访问，因为其构造方法是private的，所以这句必不可少，
    //否则会报  can not access a member of class day18.Person with modifiers "private" 的错误
    constructor.setAccessible(true);
    //利用newInstance 创建对应的对象
    Person p1 = (Person) constructor.newInstance();

    Constructor constructor2 = person1.getDeclaredConstructor(String.class);
    constructor2.setAccessible(true);
    Person p2 = (Person) constructor2.newInstance("p2");

    System.out.println(p1.name);
    System.out.println(p2.name);
} catch (Exception e) {
    e.printStackTrace();
}
```
输出如下：
```
jone
p2
```
我们可以利用``getDeclaredConstructor()``，来获取对应的构造方法。当然，还有其他方法可以获取到构造方法，我们这里以他为例。``getDeclaredConstructor()``，中间可以添加任意个参数，表示有多少个参数的构造方法。例如``getDeclaredConstructor(String.class)`` ，对应的构造方法就是``private Person(String name)``,然后在``newInstance``，传入对应的参数即可。

**获取构造方法的四个方法**：

|方法名|作用|备注|
|:-----:|----|----|
|``public Constructor<T> getDeclaredConstructor(Class<?>... parameterTypes)``|获得对应的构造方法|获取的构造方法必须是当前类申明的，无法获取到父类的构造方法|
|``public Annotation[] getDeclaredAnnotations()``|获得所有的构造方法|获取所有的构造方法都是当前类申明的，无法获取到父类的构造方法|
|``public Constructor<T> getConstructor(Class<?>... parameterTypes)``|获得指定的构造方法|获得的构造方法必须是``public``的，可以获取从父类继承的构造方法|
|``public Constructor<?>[] getConstructors()``|获得所有的构造方法|获取构造方法都是``public``的，可以获取到父类的``phblic``构造方法|

## 通过反射获取属性
还是沿用上次的``Person``类，我们这次来获取他的``des``属性。观察一下，发现``des``属性为``private``,常规就无法获取到其属性值，下面我们通过反射来获取其``des``属性的值。
```
 try {
    //获取对应的类对象，如果直接获取不到，可以用Class.forName("packageName.Person"); 来获取
    Class person1 = Person.class;
    //获取无参构造方法
    Constructor constructor = person1.getDeclaredConstructor();
    //设置构造方法可访问
    constructor.setAccessible(true);
    //创建Person对象
    Person p1 = (Person) constructor.newInstance();
    //获取des 属性值
    Field des = person1.getDeclaredField("des");
    //设置 des 可访问
    des.setAccessible(true);


    System.out.println("des= "+des.get(p1));

} catch (Exception e) {
    e.printStackTrace();
}

```
注释很详细，不在赘述了，这里打印的结果就是``Person``类里无参构造方法赋值的：
```
des= just a normal person
```
**获取属性的方法**

|方法名|作用|备注|
|:-----:|----|----|
|``public Field[] getDeclaredFields()``|获取所有的属性|只能获取本类的属性（包括private的），不能获取到父类的属性|
|``public Field getDeclaredField(String name)``|获取指定的的属性|只能到获取本类的属性（包括private的），不能获取到父类的属性|
|``public Field[] getFields()``|获取所有的属性|只能到获取``public``的属性，也可以获取到父类的``public``属性|
|``public Field getField(String name)``|获取指定的的属性|只能到获取``public``的属性，也可以获取到父类的``public``属性|

## 通过反射获取方法
我们还是是假设有一个``Person``类，我们将其改造成如下：
```
class Person {
    public String name;
    public char sex;
    public int height;
    protected float id;
    private String des;

    private Person() {
        this.name = "jone";
        this.sex = 'f';
        this.height = 155;
        this.id = 420458188603268922F;
        this.des = "just a normal person";
    }
    

    private void showInfo(){
        System.out.println("name="+name);
        System.out.println("sex="+sex);
        System.out.println("height="+height);
        System.out.println("id="+id);
        System.out.println("des="+des);
    }

    private void showInfo(String name){
        System.out.println("name = [" + name + "]");
    }

    private String showInfo(String name,String des){
        return "the people name is"+name+" and des is "+des;
    }

}
```
如上，我们实现了三个``ShowInfo()``的重载方法，但是都是``private``类型的，一般情况下在外部是无法调用的，下面我们通过反射来进行调用。
```
try {
    //获取对应的类对象，如果直接获取不到，可以用Class.forName("packageName.Person"); 来获取
    Class person1 = Person.class;
    //获取无参构造方法
    Constructor constructor = person1.getDeclaredConstructor();
    //设置构造方法可访问
    constructor.setAccessible(true);
    //创建Person对象
    Person p1 = (Person) constructor.newInstance();

    //获取无参的showInfo方法
    Method method1 = person1.getDeclaredMethod("showInfo");
    //设置showInfo方法访问
    method1.setAccessible(true);
    //执行无参的showInfo方法
    method1.invoke(p1);
    System.out.println();

    //获取一个参数的的showInfo方法
    Method method2 = person1.getDeclaredMethod("showInfo",String.class);
    //设置showInfo方法访问
    method2.setAccessible(true);
    //执行无参的showInfo方法
    method2.invoke(p1,"call from outside:");
    System.out.println();

    //获取两个参数的的showInfo方法
    Method method3 = person1.getDeclaredMethod("showInfo",String.class,String.class);
    //设置showInfo方法访问
    method3.setAccessible(true);
    //执行无参的showInfo方法
    String dsc = (String) method3.invoke(p1," arche","call from outside:");
    System.out.println(dsc);

} catch (Exception e) {
    e.printStackTrace();
}
```
我们通过反射分别调用了上述的三个参数，输出的结果如下：
```
name=jone
sex=f
height=155
id=4.20458194E17
des=just a normal person

name = [call from outside:]

the people name is arche and des is call from outside:
```
下面总结一下反射方法相关的函数：


## 通过反射获取参数

|方法名|作用|备注|
|:-----:|----|----|
|`` public Method getDeclaredMethod(String name, Class<?>... parameterTypes)``|获取指定的方法|只能获取本类的方法（包括private的），不能获取到父类的方法|
|``public Field[] getDeclaredFields()``|获取所有的的方法|只能获取本类的方法（包括private的），不能获取到父类的方法|
|``public Method getMethod(String name, Class<?>... parameterTypes)``|获取指定的的方法|只能获取``public``的方法，也能获取到父类的``public``方法|
|`` public Method[] getMethods()``|获取所有的的方法|只能获取``public``的方法，能获取到父类的``public``方法|
|``public Object invoke(Object obj, Object... args)``|执行对应的方法|执行方法的参数应和获取方法时的参数类型相同,可以为null|
|``public Class<?> getReturnType()``|获取方法的返回值类型|使用示例`` Class<?> type = method1.getReturnType();``|
|``public Class<?>[] getParameterTypes()``|获得方法的传入参数类型|使用示例`` Class<?>[] Parametertypes = method1.getParameterTypes();``|
|``public Class<?>[] getParameterTypes()``|获得方法的传入参数类型|使用示例`` Class<?>[] Parametertypes = method1.getParameterTypes();``|

## 通过反射获取注解




## 通过反射获取泛型
假设``Person``对象如下所示：
```
class Person {
     private List<String> friends;
}
```
我们要获取其 friends对象的泛型，可以用下面的方式实现：
```
 try {
    Class<?> p = Person.class;
    Field friends = p.getDeclaredField("friends");
    Type genericType = friends.getGenericType();//获取对应的类型
    ParameterizedTypeImpl parameterizedType = (ParameterizedTypeImpl) genericType; //强转成具体的实现类
    Type[] genericTypes = parameterizedType.getActualTypeArguments();  //取得包含的泛型类型
    System.out.println(genericTypes[0]);

} catch (Exception e) {
    e.printStackTrace();
}
```
如此我们就获取的了``friends``的泛型类，打印的结果为：
> class java.lang.String


# 参考链接
* [JAVA中的反射机制](https://blog.csdn.net/liujiahan629629/article/details/18013523)
* [Java 反射机制详解](https://blog.csdn.net/gdutxiaoxu/article/details/68947735)
* [深入理解java反射机制](https://blog.csdn.net/u012585964/article/details/52011138)
* [深入分析Java方法反射的实现原理](http://www.importnew.com/23902.html)