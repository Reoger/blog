---
title: android 中的 Unsafe 
date: 2020-07-12 10:23:33
categories: android
tags: android, TextView
---

# unSafe 类之初见

假设有如下的代码：
```
data class Person(var name: String, var age: Int)

private fun test() {
 val person = Gson().fromJson("{  \"age\":1 }", Person::class.java)
        Log.d(TAG, "persion :${person.age} & ${person.name} ")
}
```
上述代码，执行 ``test`` 函数之后会怎么样？ 
1. 在执行 ``fromJson`` 时就抛出空指针异常？因为 ``name``为``null`` ?
2. 返回的 ``person`` 为 ``null``, 导致后续执行抛出空指针?
3. ``test()`` 正常执行，输出 ``persion: 1 null ``
4. ``test()`` 正常执行，输出 ``persion: 1``

**答案应该是 3**

通过源码的角度来快速了解一下 ``gson`` 转换成对应对象的过程
``Gson``类中的 ``fromJson``相关实现：
```
  public <T> T fromJson(String json, Class<T> classOfT) throws JsonSyntaxException {
     //转化成功都在 fromJson 方法中, 重点
    Object object = fromJson(json, (Type) classOfT);
    return Primitives.wrap(classOfT).cast(object);
  }

    @SuppressWarnings("unchecked")
  public <T> T fromJson(String json, Type typeOfT) throws JsonSyntaxException {
    if (json == null) {
      return null;
    }
    StringReader reader = new StringReader(json);
    // 从 StringReader 转化成对应的对象, 重点
    T target = (T) fromJson(reader, typeOfT);
    return target;
  }

  public <T> T fromJson(Reader json, Type typeOfT) throws JsonIOException, JsonSyntaxException {
    JsonReader jsonReader = newJsonReader(json);
    T object = (T) fromJson(jsonReader, typeOfT);
    assertFullConsumption(object, jsonReader);
    return object;
  }

   public <T> T fromJson(JsonReader reader, Type typeOfT) throws JsonIOException, JsonSyntaxException {
    boolean isEmpty = true;
    boolean oldLenient = reader.isLenient();
    reader.setLenient(true);
    try {
      reader.peek();
      isEmpty = false;
      TypeToken<T> typeToken = (TypeToken<T>) TypeToken.get(typeOfT);
      // 通过 typeToken 获得 TypeAdapter<T>,TypeAdapter 就是抓换成目标对象的关键
      TypeAdapter<T> typeAdapter = getAdapter(typeToken);
      T object = typeAdapter.read(reader);
      return object;
    } catch (EOFException e) {
        ...
        //省略了异常处理相关代码
    }
   }

```
从上述源码来看，``fromJson`` 通过将传入的数据转化成 ``StringReader`` ，通过 ``getAdapter``方法就传入的 ``class``示例话成对象，然后数据``read`` 到示例化出来的对象之后，返回，就达到了将 json 数据转换成 bean 对象的目的。了解了大致流程之后，我们主要关注 ``getAdapter(typeToken)`` 是如何获取到合适的对象，并最终实例化出来的。
下面是 ``getAdapter(typeToken)`` 部分代码：
```
 @SuppressWarnings("unchecked")
  public <T> TypeAdapter<T> getAdapter(TypeToken<T> type) {
   ...
      for (TypeAdapterFactory factory : factories) {
        TypeAdapter<T> candidate = factory.create(this, type);
        if (candidate != null) {
          call.setDelegate(candidate);
          typeTokenCache.put(type, candidate);
          return candidate;
        }
      }
   ...
  }
```
这里是通过 ``ReflectiveTypeAdapterFactory`` 来构建具体的 ``adapter`` 的。
```
  @Override public <T> TypeAdapter<T> create(Gson gson, final TypeToken<T> type) {
    Class<? super T> raw = type.getRawType();

    if (!Object.class.isAssignableFrom(raw)) {
      return null; // it's a primitive!
    }

    ObjectConstructor<T> constructor = constructorConstructor.get(type);
    return new Adapter<T>(constructor, getBoundFields(gson, type, raw));
  }
```
重点看一下 ``constructorConstructor.get(type)`` 这个方法的实现：
```
public <T> ObjectConstructor<T> get(TypeToken<T> typeToken) {
    ...
    ObjectConstructor<T> defaultConstructor = newDefaultConstructor(rawType);
    if (defaultConstructor != null) {
      return defaultConstructor;
    }

    ObjectConstructor<T> defaultImplementation = newDefaultImplementationConstructor(type, rawType);
    if (defaultImplementation != null) {
      return defaultImplementation;
    }

    // finally try unsafe
    return newUnsafeAllocator(type, rawType);
  }
```
可以看到最终，有三个 ``Constructor``
1. newDefaultConstructor 尝试获取了无参的构造函数，则通过 ``newInstance``反射方式构建对象
2. newDefaultImplementationConstructor 集合类相关的构建，如果传入类的不是 ``Map``、``Collection``等相关的类，则会返回 ``null``,返回返回对应的集合。
3. newUnsafeAllocator 最后通过 ``unsafe`` 的方式来构建

我们主要看最后一个方法的实现：
```
  private <T> ObjectConstructor<T> newUnsafeAllocator(
      final Type type, final Class<? super T> rawType) {
    return new ObjectConstructor<T>() {
      private final UnsafeAllocator unsafeAllocator = UnsafeAllocator.create();
      @SuppressWarnings("unchecked")
      @Override public T construct() {
        try {
          Object newInstance = unsafeAllocator.newInstance(rawType);
          return (T) newInstance;
        } catch (Exception e) {
          throw new RuntimeException(("Unable to invoke no-args constructor for " + type + ". "
              + "Registering an InstanceCreator with Gson for this type may fix this problem."), e);
        }
      }
    };
  }

```
看起来就是通过 ``UnsafeAllocator.create();`` 获得了一个构造器，最终通过调用 ``newInstance()`` 将其构造出来了，那么是什么样的构造器，竟然能跳过 ``kotlin`` 的空值判断，创建一个属性值为 null 的对象，下面来看其实现代码：
```
  public static UnsafeAllocator create() {
    // try JVM
    // public class Unsafe {
    //   public Object allocateInstance(Class<?> type);
    // }
    try {
      Class<?> unsafeClass = Class.forName("sun.misc.Unsafe");
      Field f = unsafeClass.getDeclaredField("theUnsafe");
      f.setAccessible(true);
      final Object unsafe = f.get(null);
      final Method allocateInstance = unsafeClass.getMethod("allocateInstance", Class.class);
      return new UnsafeAllocator() {
        @Override
        @SuppressWarnings("unchecked")
        public <T> T newInstance(Class<T> c) throws Exception {
          assertInstantiable(c);
          return (T) allocateInstance.invoke(unsafe, c);
        }
      };
    } catch (Exception ignored) {
    }
    ...
  }
```
看到着，终于发现 ``UnsafeAllocator `` 的庐山真面目了，原来就是他通过 ``sun.misc.Unsafe`` 类绕过了构造方法，直接构建了一个对象。

了解了他的实现原来之后，我们可以给予他，也创建过不需要执行构造方法的对象：
```
private fun testCreate() {
        val unsafeAllocator = UnsafeAllocator.create()
        val person1 = unsafeAllocator.newInstance(Person::class.java)
        person1.age = 112;
        Log.d(TAG, " person1 ${person1.name} and ${person1.age}")
}
```
当然，也可以完全不实用他的构建方法，自己通过 ``UnSafe``实现也是一样。

## Unsafe 类的作用
![Java魔法类：Unsafe应用解析](https://p1.meituan.net/travelcube/f182555953e29cec76497ebaec526fd1297846.png)

java 中的 CAS 操作也基本上是通过 unSafe 类来实现的，包括 ``AtomicInteger``,``

# 参考文档
- [Java魔法类：Unsafe应用解析](https://tech.meituan.com/2019/02/14/talk-about-java-magic-class-unsafe.html)
- [Android避坑指南，发现了一个极度不安全的操作](https://mp.weixin.qq.com/s/jVRTFTiwTtr7P7vyAj8G7A)
- [深入理解sun.misc.Unsafe原理](https://blog.csdn.net/zyzzxycj/article/details/89877863)
