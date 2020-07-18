---
title: java中的动态代理
date: 2018-08-07 16:44:55
categories: java 
tags: android,java,动态代理
---
# 代理模式
什么是代理模式？代理模式给某一个对象提供一个代理对象，并由代理对象控制对原对象的引用。通俗的来讲代理模式就是我们生活中常见的中介。
下面以一个简单的例子来说明代理模式。
小明有两件事要做，用代码表示为：
```
public interface IDoSomething {
    String[] doShopping(int money);
    Object doOtherThing(String thing);
}
```
当然，小明要做这些事情，他也知道怎么去做，购物和做其他事情的实现如下：
```
public class DoSomethingImpl implements IDoSomething {

    @Override
    public String[] doShopping(int money) {
        System.out.println("买买买！！");
        System.out.println(String.format("花费%d $",money));
        return new String[]{"零食","游戏机","mac"};
    }

    @Override
    public Object doOtherThing(String thing) {
        System.out.println(" Other Thing ->"+thing);

        return thing+" is done";
    }
}
```
虽然，小明有这两件事要做，他也知道怎么去做，但是他现在就是不想自己去做。那怎么办？
找人帮他做呗，也就是找个``代理``，代理的简单实现如下：
```
public class ProxyDoSomething implements IDoSomething{
    private IDoSomething base;

    public ProxyDoSomething(IDoSomething base) {
        this.base = base;
    }


    @Override
    public String[] doShopping(int money) {
        //先扣点手续费
        int realMoney = (int) (money*0.1);
        String [] object = base.doShopping(realMoney);
        if (object != null && object.length > 1){
            object[0]= "把这个换成拼多多的货";
        }
        return object;
    }

    @Override
    public Object doOtherThing(String thing) {
        String str = thing+" 这件事是代理出面做的";
        String res = (String) base.doOtherThing(str);
        return res;
    }
}
```
找人做代理，难免要点手续费，然后可能还会动点手脚。当然，最终事情还是帮我们做好了，接下来我们看不要小明亲自出面，怎么来实现的。

```
public static void main(String args[]){
        IDoSomething iDoSomething = new DoSomethingImpl();
        ProxyDoSomething proxyDoSomething = new ProxyDoSomething(iDoSomething);
        System.out.print(proxyDoSomething.doOtherThing(" clean house").toString());
        System.out.println(Arrays.toString(proxyDoSomething.doShopping(1000)));
    }
```
可以看到，本来是小明要做的事情，最终代理给了``ProxyDoSomething``去完成，当然，代价也比较惨重。
最终打印的结果为：
```
 Other Thing -> clean house 这件事是代理出面做的
 clean house 这件事是代理出面做的 is done买买买！！
花费100 $
[把这个换成拼多多的货, 游戏机, mac]
```

# 动态代理
通过前一个例子理解了什么是代理，那么什么是动态代理呢？
通俗的理解就是，这个代理的对象可以代理很多事情，不会局限于静态代理一对一的关系，动态代理可以实现一 对多。类似于一个万能的代理，既能给代理购物，也能给别人代理做家务等等。
废话不多说，先上代理的核心代码：
```
import java.lang.reflect.InvocationHandler;
import java.lang.reflect.Method;

/**
 * 动态代理实现的核心，很多事情就是在这里进行实现的。
 * 首先，我们需要拿到需要代理的对象
 * 然后，需要判断对应的方法
 * 最后，通过反射进行调用
 */
public class DoSomethingHandler implements InvocationHandler {
    private IDoSomething doSomething;

    public DoSomethingHandler(IDoSomething doSomething) {
        this.doSomething = doSomething;
    }

    @Override
    public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
        if ("doShopping".equals(method.getName())){
            int money = (int) ((int) args[0]*0.8);
            doSomething.doShopping(money);
            String[] things = (String[]) method.invoke(doSomething, money);
            if (things != null && things.length > 1){
                things[0] = "这个我要掉包，换个来自拼多多的好货";
            }
            return things;
        }else if("doOtherThing".equals(method.getName())){
            String str = args[0] +" by DosomeThingHadler";
            Object o = method.invoke(doSomething,str);
            return o+" 来自 DosomeThingHadler 的问候";
        }
        return null;
    }
}
```
简单的介绍在代码里进行介绍了，这里就不多介绍了。我们来看调用的方法。
```
  System.out.println("开始啦啦啦啦~ ");
        IDoSomething people = new DoSomethingImpl();
        //不使用代理，自己来
        System.out.println(Arrays.toString(people.doShopping(1000)));
        //动态代理
        people = (IDoSomething) Proxy.newProxyInstance(IDoSomething.class.getClassLoader(),people.getClass().getInterfaces()
                ,new DoSomethingHandler(people));
        System.out.println(Arrays.toString(people.doShopping(1000)));
        System.out.print(people.doOtherThing(" clean house"));
```
通过``Proxy.newProxyInstance``创建出动态代理实例，然后进行调用即可。
上例运行的结果如下：
```
开始啦啦啦啦~ 
买买买！！
花费1000 $
[零食, 游戏机, mac]

买买买！！
花费800 $
买买买！！
花费800 $
[这个我要掉包，换个来自拼多多的好货, 游戏机, mac]
 Other Thing -> clean house by DosomeThingHadler
 clean house by DosomeThingHadler is done 来自 DosomeThingHadler 的问候
```
# 总结
动态代理的实现主要是基于``Proxy.newProxyInstance``创建出我们需要的代理对象，而创建这个对象又依赖于一个``InvocationHandler``，通过他来实现我们要代理的事情。


# 参考链接
* <http://weishu.me/2016/01/28/understand-plugin-framework-proxy-hook/>
