---
title: android面试准备
date: 2017-08-28 13:35:40
categories: review 
tags: [android,review]
---


# android四大组件
## activity的生命周期
![activity的生命周期图](https://gss1.bdstatic.com/9vo3dSag_xI4khGkpoWK1HF6hhy/baike/c0%3Dbaike92%2C5%2C5%2C92%2C30/sign=c070b79dcb177f3e0439f45f11a650a2/1c950a7b02087bf41b6972c2f0d3572c11dfcf17.jpg)

## activity的四种启动模式
Standard（默认）、singleTop（栈顶复用）、singleTask（栈内复用）、singleInstance（单实例）。
当然，不但可以通过在AndroidManifest.xml中声明launchMode，也可以通过指定Activity的Flags来设置启动模式。例如：
FLAG_ACTIVITY_NEW_TASK 相当于singleTask模式；
FLAG_ACTIVITY_SINGLE_TOP 相当于singleTop模式；
FLAG_ACTIVITY_CLEAR_TOP 标记的activity在同一个任务栈中所有位于它上面的activity都要出栈；
FLAY_ACTIVITY_EXCLUDE_FROM_RECENTS 标记的activity不会出现在历史的activity列表中。

## activity的四种状态
runing、paused、stopped、kille

## activity的启动方式
显式和隐式启动。其中隐式启动的匹配规则：
intentFilter中过滤信息有action、category、data。
Intent中的action能够和过滤规则中的任何一个action相同即可；
Intent中所有的category都必须和过滤过则中的某一个category相同；
Intent中的data能够和过滤规则中的任何一个data相同即可。
在隐式启动的时候，可以用packageManager的resolveActivity或者Activity的resolveActivity方法先判断是否有activity相匹配，避免异常发生。

## activity的启动流程
参考[这里](http://blog.csdn.net/singwhatiwanna/article/details/18154335)

## activity异常停止时保存数据的方法
可以在onSaveInstanceState（在onStop之前被调用）中将要保存的数据保存起来，可以通过Bundle进行临时保存，然后在onCreate中的bundle中取出来进行恢复，但是oncreate中进行数据恢复要注意判空，也可以在onRestoreInstanceState（）取出数据进行恢复（这里就不要判空，因为只要回调这个方法就一定不为空）。[参考。](http://blog.csdn.net/reoger/article/details/51354658)

## Activity、Windows、View三者的区别和联系。
Activity是控制单元，通过attach方法创建window对象；window是承载模型，负责承载视图；view是要显示的视图，必须依附于window；
Window是一个抽象类，他的具体实现是phoneWindow，Android中所有的视图都是通过window来呈现的，不管是activity、Dialog还是Toast，他们的视图实际上都是附加在window上的，因为window实际是view的直接管理者。
最后总结一句话：activity像一个工匠（控制单元），window向窗户（承载模型），View像窗花（显示视图）。

## activity间通过Intent传递数据的大小限制
肯定是有限制的，大约大1m（1020k）左右。但是官方并没有给出说明。 

##  Fragment的生命周期
![fragment生命周期图](http://img.my.csdn.net/uploads/201211/29/1354170699_6619.png)

## 添加fragment的步骤。
```
//步骤1、创建待添加的的fragment实例
MyFragment m = new MyFragment();

//步骤2、开始一个事务
FragmentManager fragmentManager = getFragmentManager(); //获取fragment对象
FragmentTransaction fragmentTransaction = fragmentManager.beginTransaction(); //利用fragment开启事务

//步骤3、向容器内加入碎片,可以通过add或者replace方法实现。
fragmentTransaction.add(R.id.fragment, m);

//步骤4、提交事务
fragmentTransaction.commit();
```

## 添加Fragment时方法
一般有两种方法添加碎片，即通过replace和add方法，下面是两种方法的区别：
通过add方法添加的fragment，每个fragment都只能添加一次。因此如果想达到切换的效果需要通过fragment的hide和show方法结合使用。将要显示的show出来，将其他hide起来，这个过程fragment生命周期没有变化。
通过replace方法添加fragment，每个fragmnet可以添加任意次，因为每一次添加都会先将之前的fragmemnt移除，然后在添加。因此切换fragment，每次都会执行上一和fragment的onDstoryView，新的Fragemnt的onCreateView、onStart、onResume方法。

## Activity和Fragment之间的区别。
1．  Fragment显得更加灵活，可以在xml文件中添加<fragment>，动态的替换一部分界面，activity则不行；
2．  Fragment不需要在manfest中申明，而activity需要。

## Activity与Fragment通信方案。
1.  handler：该方案存在的缺点：
Fragment对具体的Activity存在耦合，不利于Fragment复用
不利于维护，若想删除相应的Activity，Fragment也得改动
没法获取Activity的返回数据
2.  广播方案，缺点：
用广播解决此问题有点大材小用了，个人感觉广播的意图是用在一对多，接收广播者是未知的情况
广播性能肯定会差
传播数据有限制（必须得实现序列化接口才可以）
3.  EventBus方案：
EventBus是用反射机制实现的，性能上会有问题
EventBus难于维护代码
没法获取Activity的返回数据
4.  接口方案
假如项目很大了，Activity与Fragment的数量也会增加

##  Service的生命周期。
Service的生命周期分为两种，一种是直接利用startCommand启动，另一种是通过bindService来启动。具体如图所示。值得注意的是，在已经启动的service，我们重新启动的时候，不会执行onCreate方法，而是直接执行onStartCommand方法。
![服务生命周期](https://developer.android.google.cn/images/service_lifecycle.png)
左图显示了使用 startService() 所创建的服务的生命周期，右图显示了使用 bindService() 所创建的服务的生命周期。

## Service和Thread的区别
Service是安卓中系统的组件，它运行在独立进程的主线程中，不可以执行耗时操作。Thread是程序执行的最小单元，分配CPU的基本单位，可以开启子线程执行耗时操作;
Service在不同Activity中可以获取自身实例，可以方便的对Service进行操作。Thread在不同的Activity中难以获取自身实例，如果Activity被销毁，Thread实例就很难再获取得到。

## Broadcast Receiver的种类
* 普通广播
* 有序广播
* 本地广播
* Sticky广播

## 注册广播有几种方式？他们有什么区别？
广播注册分为静态注册和动态注册，其中的静态注册的广播在应用安装时由系统自动完成注册，具体来说是由PMS来完成整个注册过程。
静态注册属于常驻型广播，也就是说当应用程序关闭后，如果有信息广播来，程序也会被系统调用自动运行（在android 3.1以后就可能不成立了，加入了一个[flag位](http://www.jianshu.com/p/92ccb8ca2ea5)）。动态注册不是常驻型广播，也就是说广播跟随程序的生命周期。

## Broadcast Receiver实现机制
1. 自定义广播类继承BroadcastReceiver，复写onReceiver()
2. 通过Binder机制向AMS进行注册广播
3. 广播发送者通过Binder机制向AMS发送广播
4. AMS查找符合相应条件的广播发送到BroadcastReceiver相应的循环队列中
消息队列执行拿到广播，回调BroadcastReceiver的onReceiver()

## LocalBroadcastManager特点
* 本地广播只能在自身App内传播，不必担心泄漏隐私数据
* 本地广播不允许其他App对你的App发送该广播，不必担心安全漏洞被利用
* 本地广播比全局广播更高效
* 以上三点都是源于其内部是用Handler实现的


## ContentProvider是如何实现数据共享的。
在android中如果想就将自己应用的数据（一般多位数据库中的数据）提供给第三方应用，那么我们只能通过ContentPrivider来实现。
ContentProvider是应用程序之间共享的接口，使用的时候首先自定义一个类继承ContentProvider，然后覆写query、insert、update、delete等方法，因为其是四大组件之一，因此必须在AndroidManifest文件中进行注册，把自己的数据通过uri的形式共享出去。第三方可以通过ContentResplver来访问该Provider。

##   Application的生命周期？
onCreate() --> 创建application  --> onTerminate() --> 结束。
其中，在application中，可能会回调``onConfigurationChanged()``（在设备配置发生改变的时候调用）、``onLowMemory()``（手机内存低时调用））、``OnTrimMemory()``（在操作系统清理内存的时候时候调用）。


# View相关面试题

##  简述View的事件分发机制。
当一个点击事件产生后，他的传递过程应该是：Activity --> windos --> View;
顶级View接到事件后，就会按照事件分发机制去分发事件，具体如下图所示。
值得注意的是，ViewGroup默认不拦截任何事件，而子View中没有OnIntercepterTouchEvent方法，一旦有事件传递给他，那么他的onTouchEvent就会马上调用。某个View一旦决定拦截事件，那么这一个事件序列都只能由他来处理。
简单来说：点击事件分发过程如下 dispatchTouchEvent—->OnTouchListener的onTouch方法—->onTouchEvent-->OnClickListener的onClick方法。也就是说，我们平时调用的setOnClickListener，优先级是最低的，所以，onTouchEvent或OnTouchListener的onTouch方法如果返回true，则不响应onClick方法。
![Touch事件传递机制流程图](./img/view事件分发机制.png)

##  view测量的宽/高和实际上的宽/高是一样的吗？
不一定一样，虽然在大部分情况下是一样的，但是在某些情况下会导致两者的值不相同。
在View的默认实现中，View的测量宽/高和最终的宽/高是相等的，只不过测量宽/高形成于measure过程，而最终宽/高形成与View的layout过程，即两者的赋值时机不同，测量宽/高的赋值时机要稍微早一点。当我们重写layout方法时，可以导致测量的宽/高和实际的宽/高不同（虽然没有什么具体的含义，但是证明了view测量的宽/高和实际的宽/高是可以不相同的）。

## 在activity启动的时候就需要获取某个View的宽/高，该怎么做？
注意，我们一定不能在onCreate、onStart、onResume中去获取。因为View的measure过程和Activity的生命周期方法不是同步的执行的，因此无法保证Activity在某个生命周期方法中这个View已经测量完毕了。
我们可以在onWindowFocusChanged方法中获取；我们也可以通过view.post(runnable)将runnable投递到消息队列的尾部，然后等待Looper调用此runnable获取View的高和宽；我们也可以使用ViewTreeObserver的众多回调来获取View的高，例如使用onGlodalLayoutListener这个接口。最后，我们当然也可以手动对view进行measure来得到View的宽/高。

## 自定义View要注意哪些地方？
让view支持wrap_content；
直接继承View或者ViewGroup的控件，如果不在onMeasure中对wrap_content做特殊的处理，那么当外界在布局中使用wrap_content时就无法达到预期的效果。
让view支持padding和margin；
直接继承view的话，如果不在draw方法中处理paddding，那么padding属性是无法起作用的。另外，直接继承ViewGroup的控件需要在onMeasure和onLayout中考虑padding、
和子元素margin对其造成的影响，不然将会导致这两个属性失效。
  尽量不要在View中使用handler，view中如果有线程或者动画，一定要及时停止，否则可能会造成内存泄露；
  View带有滑动嵌套的时候，要注意处理滑动冲突。

##  自定义view的步骤。
简单来说，主要有三大步骤：
* onMeasure
* onLayout
* onDraw
具体详细，可以参考[博客](http://blog.csdn.net/qq_30379689/article/details/54588736)

# 动画相关
##  Android中的三种动画。
Android中的动画有三种：View动画、帧动画和属性动画。
 View动画的作用对象是View，他支持4中动画效果：分别是平移动画（TranslateAnimation）、缩放动画（ScaleAnimation）、旋转动画（RotateAnimation）和透明度动画（AlphaAnimation）。
帧动画是顺序播放一组预先定义好的图片，类似于电影播放。
属性动画是在API 11新加入的特性，属性动画可以对任何对象做动画，甚至可以没有对象。除了动画对象进行了扩展以外，属性动画的动画效果也进行了加强，不像View动画只支持四种动画。

##  可以对任意属性做动画吗？
可以。如果想对Object的属性abd做动画，如果想让动画生效，要同时满足两个条件：
object必须要提供setAbd方法，如果动画的时候没有传递初始值，那么还要提供getAbd方法，因为系统要去取abd属性的初始值（如果这个条件不满足，程序直接Crash）；
object的setAbd对属性adc所做的改变必须能够通过某种方法反映出来，比如会带来UI的改变之类的（如果这条不满足，动画无效果但不会Crash）。
针对上面的两个条件，google官方告诉我们三种解决方案：
给你的对象加上get和set方法，如果有权限的话；
用一个类来包装原始对象，间接为其提供get和set方法；
采用ValueAnimator，监听动画过程，自己实现属性的改变。

##  使用动画的注意事项。
在开发的过程中尽量少用帧动画，避免OOM；
在activity退出时及时停止动画，避免内存泄露；
在android3.0以下的系统中使用属性动画需做好适配工作；
在view动画中，view只是对view的影像做动画，并没有真正的改变view的状态；
使用动画的时候，建议开启硬件加速（在声明文件中添加android:hardwareAccelerated="true"属性即可），提高动画的流畅性。

##  怎么为activity设置切换效果，怎么给ViewGroup子元素添加子元素的出场动画？
Activity有默认的切换效果，但是这个效果我们可以自定义的，主要用到overriderPendingTransition()这个方法，这个方法必须在startActivity或者finish()之后调用才能生效。至于Fragment的切换效果，我们可以通过setCustormAnimation方法来添加切换动画。
至于在ViewGroup中为子元素出场添加动画，可以在xml中对布局的LayoutAnimation引用相应的动画资源文件即可，或者我们也可以在java中利用LayoutAnimationController将获取到的动画资源文件设置到viewGroup中即可。

## 动画中的差值器和估值器怎么理解？
时间插值器（TimeInterpolator）的作用是根据时间流逝的百分比来计算出当前属性值改变的百分比，系统预置的有LinearInterpolator（线性插值器：匀速动画），
AccelerateDecelerateInterpolator（加速减速插值器：动画两头慢中间快），
DecelerateInterpolator(减速插值器：动画越来越慢）。
估值器（TypeEvaluator）的作用是根据当前属性改变的百分比来计算改变后的属性值。系统预置有IntEvaluator 、FloatEvaluator 、ArgbEvaluator。
具体来说 对于一个作用在view上改变其宽度属性、持续40ms的属性动画来说，就是当时间t=20ms时，时间流逝了50%，那么view的宽度属性应该改变了多少呢？这个就由Interpolator和Evaluator的算法来决定。

##  滑动冲突是如何产生的，该如何避免？
在界面中只有有内外两层同时可以滑动，这个时候就产生了滑动冲突。例如，scrollView嵌套一个listView，因为listView和scrollView都是可以滑动的，这个时候就出现了滑动冲突。
常见的解决滑动冲突的方法有两种：外部拦截和内部拦截。
外部拦截法是指点击事件都先经过父容器的拦截处理（ACTION_DOWN不能被消耗），如果父容器需要此事件就拦截，如果不需要此事件就不拦截，这样就可以解决滑动冲突问题，这种方法比较符合点击事件的分发机制。外部拦截法需要重写父容器中的onIntercepetTouchEvent方法，在内部做相应的拦截即可。
外部拦截法是指父容器不拦截任何事件，所有的事件都传递给子元素，如果子元素需要此事件就直接消耗掉，否则就交给父容器进行处理，这种方法和Android分发机制不一致，需要配合requetDisallowInterceptTounchEvent方法才能正常工作，使用起来较外部拦截法稍显复杂。


# 通信机制
## IPC指什么？android中实现IPC有哪些
IPC(Inter-Process Communication)，含义为进程间通信或者跨进程通信，是指两个进程之间进行数据交换的过程。
Android中主要通过Bundler(用于android四大组键间的进程间通信)、文件共享、AIDL、Messenger、ContentProvider和Socket来进行进程间通信。

##   Binder指什么？
直观的说，binder是android中的一个类，他实现了IBinder接口。从IPC角度来说，Binder是Android中的一种跨进程通信方式；从android应用层来说，Binder是客户端和服务端进行通信的媒介，当bindService的时候，服务端会返回一个包含了服务端业务调用的Binder对象，通过这个Binder对象，客户端就可以获取服务端提供的服务或者数据，这里的服务包括普通服务和基于AIDL的服务。

## Binder机制
参考[这里](http://www.mamicode.com/info-detail-1513074.html).

##  进程间通信中可以使用sharedPreferences吗？
不能。虽然从本质上来将，sharedPreferences也属于文件的一种，但是由于喜用对他的读/写有一定的缓存策略，即在内存中会有一份sharedPreferences文件的缓存，因此在多进程模式下，系统对他的读/写就变得不可靠，当面对高并发的读/写访问，sharedPreferences有很大的几率会丢失数据，因此，不建议在进程间中使用sharedPreferences。

##  AIDL支持的数据类型有哪些？
除short以外的基本数据类型（int、long、char、boolean、double等）;
String和CharSequence;
ArrayList、HashMap：里面的每个元素都必须被AIDL支持。
Parcelable： 所有实现了Parcelable接口的对象。
AIDL： 所有的AIDL接口本身也可以在AIDL文件中使用。

## handler原理
 Handler创建的时会采用当前线程Looper来构建内部的消息循环系统，如果当前线程没有Looper，那么就会报错。Handler创建完毕后，这个时候其内部的Looper以及MessageQueue就可以和Handler一起协同工作了，然后通过Handler的post方法将一个Runnable投递到Handler内部的Looper去处理，也可以通过Handler的send方法发送一个消息，这个消息同样会在Looper中去处理。其实post方法最终也是通过send方法完成的。Send方法被调用时，MessageQueue的enqueueMessage方法将这个消息放入消息队列中，然后Looper发现有新消息到来时，就会处理这个消息，最终消息中的Runnable或者Handler的handlerMessage方法就会被调用。注意，Looper是运行在创建Handler所在的线程中，这样依赖Handler中的业务逻辑就被切换到创建Handler所在的线程去执行了。
![handler原理](http://ovec6nnof.bkt.clouddn.com/hanlder.png)

 ## AsyncTask的工作原理。
 AsyncTask是对Thread和Handler的组合包装，方便我们在后台线程中执行操作，然后将结构发送给主线程，从而在主线程中进行UI更新等操作。
  实现原理，AsyncTask中有两个线程和一个Handler，其中一个线程池（SerialExecuor）用于任务的排队，另一个线程池（THREAD_POOL_EXECUTOR）用于真正的执行任务。InternalHandler用于将要执行的环境从线程池切换到主线程。关于AsynchTask有以下几点需要注意：AsyncTask的对象必须在主线程中创建；execute方法必须在UI线程中调用；一个axyncTask对象只能执行一次，即执行一次execute方法；在android 1.6之前AsyncTask是串行执行任务的，而在android 1.6的时候开始采用线程池里处理并行任务，但在android 3.0以后，为了避免并发错误，asyncTask又采用了一个线程来穿行执行任务，同时也提供一个串行执行任务的方法executeOnExecutor方法。

## HandlerThread是什么？简述原理并举出一个应用的例子。
HandlerThread是一种可以使用Handler的Thread。他的实现也很简单，就是在run方法中通过Looper.prepare()来创建消息队列，并通过Looper.loop()来开启消息循环，这样在实际的使用中就允许在HandlerThread中创建Handler了。
HandlerThread在Android中的一个具体的使用场景是IntentService。IntentService封装了HandlerThread和Handler。


## IntentService使用和原理。
IntentService是一种特殊的Service，他继承了Service并且他是一个抽象类，因此必须创建他的子类才能使用IntentService。IntentService可用于执行后台耗时的任务，当任务执行后他会自动停止，同时由于IntentService是服务的原因，这导致他的优先级比单纯的线程要高很多，所以IntentService比较适合执行一些高优先级的后台任务。
原理上来说，IntentService封装了Handler和Handler。在onCreate方法中创建一个HadnlerThread，然后使用它的Looper来构建一个Handler对象mServicehandler,这样通过mServiceHandler发送的消息最终都会在HandlerThread中执行。而在OnStart方法中，IntentService仅仅只是通过mServiceHandler发送了一个消息，这个消息会在HandlerThread中被处理。


# android中的线程与线程池
##  Android中的线程池有什么优点？
可以重用线程池中的线程，避免因为线程的创建和销毁所带来的性能开销；
能有效控制线程池的最大并发数，避免大量的线程之间因互相抢夺资源而导致的阻塞现象；
能够对线程进行简单的管理，并提供定时执行已经指定间隔循环执行等功能。
[ThreadPoolExecutor](http://blog.csdn.net/Reoger/article/details/77145627)是线程池的真正实现。


# 性能优化

## 什么是ANR，产生的条件有那些
ANR即Application not responding，应用程序无响应。
产生条件：在Ui线程中5s无响应，在broadcastReceiver中10s无相应，或者在service中20s无相应都会导致ANR。
解决方法：运行在主线程里的任何方法都尽可能少做事情，耗时的操作应该放在子线程中进行操作，然后通过hander或者其他方法来通知主线程。
应用程序应该避免在broadcastReceiver里做耗时的操作或者计算，代替的是，如果想用Internt广播需要执行一个耗时的动作的话，应用程序应该启动一个servicer。

## 造成ANR的主要原因
主线程被IO操作阻塞
* Activity的所有生命周期回调都是执行在主线程的
* Service默认执行在主线程中
* BoardcastReceiver的回调onReceive()执行在主线程中
* AsyncTask的回调除了doInBackground，其他都是在主线程中
没有使用子线程Looper的Handler的handlerMessage，post(Runnable)都是执行在主线程中

## OOM指什么？什么时候会出现OOM。
OOM指Out of memory（内存溢出），当前占用内存加上我们申请的内存资源超过了Dalvik虚拟机的最大内存限制就会抛出Out of memory异常
出现OOM的大部分情况都是内存泄露引起的、也有可能是操作系统本身内存就不足了。
这里只介绍内存泄露。

## 内存泄露出现与避免
总的来说，出现内存泄露的原因是本来应该被GC回收的资源，因为某种原因不能被回收，而导致内存越来越小，进而导致内存泄露，最终可能会导致OOM。
  出现内存泄露一般来自于两个原因：没有即使释放分配的内存（Cursor没有及时关闭）；
当应用不再需要这个对象，却没有释放该对象的所有引用。
检测内存泄露可以通过android studio的monitors工具，生成hprof文件，我们可以通过分析这个文件来分析内存泄露。当然，在android studio中分析内存泄露不是很直观，专业一点的工具就是MAT，我们可以将我们得到的hporf文件导入到MAT中进行分析。
当然，我们也可以通过在android项目中使用leakCanary来检测内存泄露。当应用程序出现内存泄露的时候，leakCanary会在通知栏通知我们，而且会提示我们内存泄露的相关信息，可以让我们迅速定给到内存泄露的的位置和原因。

## 具体说明什么时候出现内存泄露
1. Static 变量没有及时清空
但在类中定义了静态的Activity变量，把当前运行的Activity实例赋值给给这个静态常量。如果这个静态变量在activity的生命周期结束后没有清空，即会导致内存泄露；
2. 注册了没有及时解注册
如动态注册中，如果我们在activity中注册了广播，在activity销毁之前就应该解注册，否则就会出现内存泄露；
3. 资源或者属性动画没有及时关闭
在一类属性动画中，可以无线循环播放，如果在activity中播放此类动画且在activity生命周期中没有停止动画，那么动画会一直播放下去，尽管我们已经看不到效果了。至于资源回收典型的例子就是数据的Cursor了，当操作完数据库没有及时关闭Cursor也可能会导致内存泄露；
4. 静态类或者内部类导致的内存泄露
这类导致的内存泄露究其原因也是因为static变量没有及时回收，这里不做介绍；
5. Handler或者AsyncTask导致的内存泄露
我们试想一种情况，在我们的activity结束之后，handler又给Ui线程发送了一个消息，或者是在activity结束之后，我们的AsynchTask任务才执行完。这两种情况都会导致activity实例无法被回收，导致内存泄露。

## 如何处理应用可能存在的crash
一般实现Thread类中UncaughtExceptionHandler对象，在发生崩溃时，系统会调用UncaughtExceptionHandler的uncaughtException方法，在uncaughtException方法中可以捕获到异常信息，可以选择把异常信息存储到SD卡中，然后在合适的时机通过网络将crash信息上传到服务器，这样就可以分析crash的原因了。当然，在crash发生时，我们可以先弹出一个对话框告诉用户crash了，然后在退出。



# 新技术

## kotlin语言的优势
* 简洁性
例如在``kotlin``中的数据类会自动获得所需的``getter、setters、toString``。
* 安全性
对象不能为空，避免出现空指针异常。
* 更优雅，遵循Effective Java设计
1. 类默认不可继承
2. 更有效的使用构建器模式
3. 默认提供单利模版
4. 重载必须使用``Override``
* 完全兼容Java


# 开源库源码解析

## **gile相关问题**

## 如何使用OkHttp进行异步网络请求，并根据请求结果刷新UI

##介绍一下OkHttp的整个异步请求流程
  前面两个问题都比较简单，具体可以参考[博客](http://reoger.tk/categories/%E6%BA%90%E7%A0%81%E8%A7%A3%E8%AF%BB/)中的解答。

##OkHttp对于网络请求都有哪些优化，如何实现的


## OkHttp框架中都用到了哪些设计模式
建造者模式、

## Okhttp的优势与劣势
Android 开发中是可以直接使用现成的api进行网络请求的。是Square公司开源的针对Java和Android程序，封装的一个高性能http请求库，它的职责跟HttpUrlConnection 是一样的，支持同步、异步，而且 OkHttp 又封装了线程池，封装了数据转换，封装了参数使用、错误处理等，api使用起来更加方便。可以把它理解成是一个封装之后的类似HttpUrlConnection的东西，但是在使用的时候仍然需要自己再做一层封装，这样才能像使用一个框架一样更加顺手。

## Volley VS OkHttp
volley不支持Post大数据，所以不适合上传文件。
okhttp支持同步、异步，封装了线程池，参数使用、错误处理等。
volley封装的扩展性强，支持HttpClient、HttpUrlConnection， 甚至支持OkHttp，而且Volley里面也封装了ImageLoader。
Okhttp基于NIO和OKio，所以性能上要比Volley更快，功能更加齐全，只是使用需要进一步的封装。

-----
## **Gilde相关问题**


# 更多的android面试资料
* <https://github.com/GeniusVJR/LearningNotes>
* <https://github.com/Reoger/android-interview-questions-cn>
* <http://www.jianshu.com/p/13786463635d>
* <https://mp.weixin.qq.com/s/bvB2U0-6ZJ1j06iVV4NmjQ>