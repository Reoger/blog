---
date: 2017-08-19 18:50
status: public
categories: 源码解读
title: glide原理解读
tags: source,glide
---

>glide是一个快速，高效的开源媒体管理和Android的图像加载框架，将媒体解码，内存，磁盘缓存和资源池封装成一个简单易于使用的接口。

相对于同为图片加载的ImageLoader、Picasso来说，他支持的图片格式更加齐全（包括video、GIF、SVG等），支持缩略请求，内存管理也更加优秀。所以现在对其源码进行一个比较简单的分析。

# 源码导入
本次源码解析均基于``3.7.0``的版本。在``build.gradle``中添加如下的依赖：
```
compile 'com.github.bumptech.glide:glide:3.7.0'
```
点击同步，然后我们就可以开始我们的源码解析之旅了。
如果想看更多的版本信息，可以去他的[github主页](https://github.com/bumptech/glide)进行查看选择。

# 简单使用示例
相信使用过gilde来加载图片，对于gilde的使用都很清楚吧。下面是加载一张图片到ImageView中的关键代码：
```
        ImageView imageView = (ImageView) findViewById(R.id.imageView);
        Glide.with(this).load(R.mipmap.ic_launcher).into(imageView);
```
  下面我就上面的简单的使用进行源码分析。
  
# 源码分析
```
        Glide.with(this).load(R.mipmap.ic_launcher).into(imageView);
```

## 总体设计图
![资料来源网络](http://ovec6nnof.bkt.clouddn.com/19-11-17.jpg)
<a algin="center" >资料来源于网络</a>
## Glide.with()
我们首先来看Glide中的第一步，with方法的实现。源码如下：
```
public static RequestManager with(Context context) {
        RequestManagerRetriever retriever = RequestManagerRetriever.get();
        return retriever.get(context);
    }
    
public static RequestManager with(Activity activity) {
        RequestManagerRetriever retriever = RequestManagerRetriever.get();
        return retriever.get(activity);
    }

public static RequestManager with(FragmentActivity activity) {
        RequestManagerRetriever retriever = RequestManagerRetriever.get();
        return retriever.get(activity);
    }
      
@TargetApi(Build.VERSION_CODES.HONEYCOMB)
public static RequestManager with(android.app.Fragment fragment) {
        RequestManagerRetriever retriever = RequestManagerRetriever.get();
        return retriever.get(fragment);
    }
    
 public static RequestManager with(Fragment fragment) {
        RequestManagerRetriever retriever = RequestManagerRetriever.get();
        return retriever.get(fragment);
    }     
```
可以看到，with有很多中重载，我们可以在with中传入Context，Activity、Fragment等等参数。但是在这些重载方法中，实现的逻辑大都差不多，首先是构建一个RequestManagerRetriever对象，然后调用get方法。那么我们先来看看RequestManagerRetriever.get();构建出来的对象的作用是什么。
继续看源码``RequestManagerRetriever.get()``：
```
/** The singleton instance of RequestManagerRetriever. */
private static final RequestManagerRetriever INSTANCE = new RequestManagerRetriever();

public static RequestManagerRetriever get() {
    return INSTANCE;
}

// Visible for testing.
RequestManagerRetriever() {
    handler = new Handler(Looper.getMainLooper(), this /* Callback */);
}
```
通过上面的源码我们可以看到，``RequestManagerRetriever.get()``返回的就是一个静态的RequestManagerRetriever对象，这个对象初始化的时候就是只是创建了一个Handler。那么这个Handler什么时候触发，我们后面在讲。
我们注意到在``with()``方法中，我们最后返回的是`` retriever.get(activity)``或者`` retriever.get(fragment)``，具体实现如下代码所示：
```
public RequestManager get(FragmentActivity activity) {
    if (Util.isOnBackgroundThread()) {//如果是在后台线程
        return get(activity.getApplicationContext());
    } else {
        assertNotDestroyed(activity);
        FragmentManager fm = activity.getSupportFragmentManager();
        return supportFragmentGet(activity, fm);
    }
}

public RequestManager get(Context context) {
    if (context == null) {
        throw new IllegalArgumentException("You cannot start a load on a null Context");
    } else if (Util.isOnMainThread() && !(context instanceof Application)) {
        if (context instanceof FragmentActivity) {
            return get((FragmentActivity) context);
        } else if (context instanceof Activity) {
            return get((Activity) context);
        } else if (context instanceof ContextWrapper) {
            return get(((ContextWrapper) context).getBaseContext());
        }
    }

    return getApplicationManager(context);
}

```
可以发现，如果当前是在后台线程的话，get()方法会返回一个``get(activity.getApplicationContext())``。我们继续来看``getApplicationManager(context)``的实现逻辑：

```
private RequestManager getApplicationManager(Context context) {
    // Either an application context or we're on a background thread.
    if (applicationManager == null) {
        synchronized (this) {
            if (applicationManager == null) {
                // Normally pause/resume is taken care of by the fragment we add to the fragment or activity.
                // However, in this case since the manager attached to the application will not receive lifecycle
                // events, we must force the manager to start resumed using ApplicationLifecycle.
                applicationManager = new RequestManager(context.getApplicationContext(),
                        new ApplicationLifecycle(), new EmptyRequestManagerTreeNode());
            }
        }
    }

    return applicationManager;
}
```
可以看出，getApplicationManager方法通过使用双重锁的单例模式通过``RequestManager``来实例化一个``applicationManager``,并将当作返回值。关于这个``applicationManager``我们稍后在做讨论。
在当前线程不是在主线程时，而且在``get()``方法中传入的不是Application对象时，会根据传入的对象来调用``RequestManagerRetriever``中重载的``get()``方法。
以传入的对象是Activity为例子，那么调用的就是``get((Activity) context)``，下面是该方法的实现逻辑：
```
@TargetApi(Build.VERSION_CODES.HONEYCOMB)
public RequestManager get(Activity activity) {
    if (Util.isOnBackgroundThread() || Build.VERSION.SDK_INT < Build.VERSION_CODES.HONEYCOMB) {
        return get(activity.getApplicationContext());
    } else {
        assertNotDestroyed(activity);//判断activity是否已经销毁了
        android.app.FragmentManager fm = activity.getFragmentManager();
        return fragmentGet(activity, fm);
    }
}

@TargetApi(Build.VERSION_CODES.HONEYCOMB)
RequestManager fragmentGet(Context context, android.app.FragmentManager fm) {
    RequestManagerFragment current = getRequestManagerFragment(fm);
    RequestManager requestManager = current.getRequestManager();
    if (requestManager == null) {
        requestManager = new RequestManager(context, current.getLifecycle(), current.getRequestManagerTreeNode());
        current.setRequestManager(requestManager);
    }
    return requestManager;
}

@TargetApi(Build.VERSION_CODES.JELLY_BEAN_MR1)
RequestManagerFragment getRequestManagerFragment(final android.app.FragmentManager fm) {
    RequestManagerFragment current = (RequestManagerFragment) fm.findFragmentByTag(FRAGMENT_TAG);
    if (current == null) {
        current = pendingRequestManagerFragments.get(fm);
        if (current == null) {
            current = new RequestManagerFragment();
            pendingRequestManagerFragments.put(fm, current);
            fm.beginTransaction().add(current, FRAGMENT_TAG).commitAllowingStateLoss();
            handler.obtainMessage(ID_REMOVE_FRAGMENT_MANAGER, fm).sendToTarget();
            //这里handler发送了一个消息，这里就是handler发送消息的时机
        }
    }
    return current;
}

//这里就是handler回调的时机，主要用于移除容器中的对象。
@Override
public boolean handleMessage(Message message) {
    boolean handled = true;
    Object removed = null;
    Object key = null;
    switch (message.what) {
        case ID_REMOVE_FRAGMENT_MANAGER:
            android.app.FragmentManager fm = (android.app.FragmentManager) message.obj;
            key = fm;
            removed = pendingRequestManagerFragments.remove(fm);
            break;
        case ID_REMOVE_SUPPORT_FRAGMENT_MANAGER:
            FragmentManager supportFm = (FragmentManager) message.obj;
            key = supportFm;
            removed = pendingSupportRequestManagerFragments.remove(supportFm);
            break;
        default:
            handled = false;
    }
    if (handled && removed == null && Log.isLoggable(TAG, Log.WARN)) {
        Log.w(TAG, "Failed to remove expected request manager fragment, manager: " + key);
    }
    return handled;
}

```
总的来说，这里逻辑还是比较简单。首先在``get()``方法中，判断是否在后台线程且判断以下当前的版本。如果不是在后台线程且当前版本低于11，那么就会先判断当前activity是否销毁了，然后调用``ragmentGet(activity, fm)``方法。在``ragmentGet(activity, fm)``方法中，首先会调用``getRequestManagerFragment(fm)``来创建``RequestManagerFragment``对象，在``getRequestManagerFragment``方法中，首先创建一个``RequestManagerFragment``，然后将该对象放入到一个名为``pendingRequestManagerFragments的hashMap``对象中，然后通过``handler``发送一个消息。
在``handleMessage``中，收到这个消息之后，就会将``RequestManagerFragment``从``pendingRequestManagerFragments的hashMap``中移除。

这里有没有想这么一个问题，为什么要存在``pendingRequestManagerFragments``这么一个容器呢？当我们创建一个``RequestManagerFragment``之后，马上将其添加到``pendingRequestManagerFragments``中，然后运行``commitAllowingStateLoss``方法之后，又马上通过``Handler``发送一个消息让其从``pendingRequestManagerFragments``容器中移除，竟然如此，那何不如直接new一个``RequestManagerFragment``，然后直接提交就好了，何必用这个容器呢？以我的理解，这个``gilde``开发者为了增加gilde的可靠性而设计的，梳理第25-36行的逻辑如下：首先通过``findFragmentByTag``来查找``RequestManagerFragment``，若其返回空，继续下面的逻辑，否则直接返回找到的``RequestManagerFragment``对象。在``findFragmentByTag``返回空的情况下，会继续用``get()``方法来从容器中查找``RequestManagerFragment``对象，如果还是空就重新创建一个``RequestManagerFragment``，并将其添加到``pendingRequestManagerFragments``容器中,然后执行``commitAllowingStateLoss``方法，随后通过handler发送消息将``pendingRequestManagerFragments``容器中的该条记录移除。那么为什么要用``pendingRequestManagerFragments``呢？道理很简单，避免快速多次调用``getRequestManagerFragment``时，重复创建``RequestManagerFragment``对象。试想一下，当我们运行完第30行的时候，第二个``getRequestManagerFragment``请求已经到了，这个时候，第二个请求需要的``RequestManagerFragment``，就可以通过``get()``方法获取，而不要去重新创建。当运行完第31行时，第二个``getRequestManagerFragment``就可以直接通过``findFragmentByTag``方法获取。这种情况是可能产生的，设想这样一种情况：当我们在Activity A和B都同时有``Gilde.with()``,我们在进入activity A后秒切ACtivity B就可能会导致上面所说的情况，只是可能性比较低，但毕竟是可能的，所以``gilde``开发团队就用了一个容器来存储当前的``RequestManagerFragment``对象，进一步提高其性能。
关于``commitAllowingStateLoss``方法，这里我们不做详细的讨论，具体可参考[这里](https://jucongyuan.com/2015/02/25/%E4%BB%8E%E6%BA%90%E7%A0%81%E7%9C%8Bcommit%E5%92%8CcommitAllowingStateLoss%E6%96%B9%E6%B3%95%E5%8C%BA%E5%88%AB/)。
然后在``fragmentGet``方法中也会调用``RequestManager``方法。似曾相识是吧，没错，我们在``getApplicationManager``方法中也调用了这个方法！！那么下面来看看这个方法究竟做了写什么吧。
```
public RequestManager(Context context, Lifecycle lifecycle, RequestManagerTreeNode treeNode) {
    this(context, lifecycle, treeNode, new RequestTracker(), new ConnectivityMonitorFactory());
}

RequestManager(Context context, final Lifecycle lifecycle, RequestManagerTreeNode treeNode,
        RequestTracker requestTracker, ConnectivityMonitorFactory factory) {
    this.context = context.getApplicationContext();
    this.lifecycle = lifecycle;
    this.treeNode = treeNode;
    this.requestTracker = requestTracker;
    this.glide = Glide.get(context); //创建一个Glide对象
    this.optionsApplier = new OptionsApplier();

    ConnectivityMonitor connectivityMonitor = factory.build(context,
            new RequestManagerConnectivityListener(requestTracker));

    // If we're the application level request manager, we may be created on a background thread. In that case we
    // cannot risk synchronously pausing or resuming requests, so we hack around the issue by delaying adding
    // ourselves as a lifecycle listener by posting to the main thread. This should be entirely safe.
    if (Util.isOnBackgroundThread()) {
        new Handler(Looper.getMainLooper()).post(new Runnable() {
            @Override
            public void run() {
                lifecycle.addListener(RequestManager.this);//将生命周期进行绑定
            }
        });
    } else {
        lifecycle.addListener(this);//绑定生命周期
    }
    lifecycle.addListener(connectivityMonitor);
}

//单例模式创建Gilde对象
 public static Glide get(Context context) {
        if (glide == null) {
            synchronized (Glide.class) {
                if (glide == null) {
                    Context applicationContext = context.getApplicationContext();
                    List<GlideModule> modules = new ManifestParser(applicationContext).parse();

                    GlideBuilder builder = new GlideBuilder(applicationContext);
                    for (GlideModule module : modules) {
                        module.applyOptions(applicationContext, builder);
                    }
                    glide = builder.createGlide();
                    for (GlideModule module : modules) {
                        module.registerComponents(applicationContext, glide);
                    }
                }
            }
        }

        return glide;
    }

//在这里进行一系列的初始化，    
 Glide createGlide() {
        if (sourceService == null) {
            final int cores = Math.max(1, Runtime.getRuntime().availableProcessors());
            sourceService = new FifoPriorityThreadPoolExecutor(cores);
        }
        if (diskCacheService == null) {
            diskCacheService = new FifoPriorityThreadPoolExecutor(1);
        }

        MemorySizeCalculator calculator = new MemorySizeCalculator(context);
        if (bitmapPool == null) {
            if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.HONEYCOMB) {
                int size = calculator.getBitmapPoolSize();
                bitmapPool = new LruBitmapPool(size);
            } else {
                bitmapPool = new BitmapPoolAdapter();
            }
        }

        if (memoryCache == null) {
            memoryCache = new LruResourceCache(calculator.getMemoryCacheSize());
        }

        if (diskCacheFactory == null) {
            diskCacheFactory = new InternalCacheDiskCacheFactory(context);
        }

        if (engine == null) {
            engine = new Engine(memoryCache, diskCacheFactory, diskCacheService, sourceService);
        }

        if (decodeFormat == null) {
            decodeFormat = DecodeFormat.DEFAULT;
        }

        return new Glide(engine, memoryCache, bitmapPool, context, decodeFormat);
    }    
```
不难看出，在``RequestManager``构造方法中，对生命周期进行了绑定，并绑定了监听。然后通过``Glide.get(context);``创建一个Gilde对象，在``Glide.get(）``方法中，
这里``createGlide``方法中，我们通过``new Gidle``创建了一个Gilde对象,简单介绍以下其中的四个参数,他们都是在createGlide中生成的。
* MemoryCache 内存缓存
* BitmapPool 图片池
* DecodeFormat 图片格式
* Engine 引擎类
我们对其值的获取和计算也进行一个简单的了解：我们首先来看memoryCacher的大小是由``calculator.getMemoryCacheSize()``计算，来看其实现：
```
//首先是其调用MemorySizeCalculator的构造方法，最终会回调到如下构造方法中
 MemorySizeCalculator(Context context, ActivityManager activityManager, ScreenDimensions screenDimensions) {
        this.context = context;
        final int maxSize = getMaxSize(activityManager);

        final int screenSize = screenDimensions.getWidthPixels() * screenDimensions.getHeightPixels()
                * BYTES_PER_ARGB_8888_PIXEL;(宽*高*4)

        int targetPoolSize = screenSize * BITMAP_POOL_TARGET_SCREENS;(宽*高*4*4)
        int targetMemoryCacheSize = screenSize * MEMORY_CACHE_TARGET_SCREENS;(宽*高*4*2)

        //判断是否超过最大值,否则就等比缩小
        if (targetMemoryCacheSize + targetPoolSize <= maxSize) {
            memoryCacheSize = targetMemoryCacheSize;
            bitmapPoolSize = targetPoolSize;
        } else {
            int part = Math.round((float) maxSize / (BITMAP_POOL_TARGET_SCREENS + MEMORY_CACHE_TARGET_SCREENS));
            memoryCacheSize = part * MEMORY_CACHE_TARGET_SCREENS;
            bitmapPoolSize = part * BITMAP_POOL_TARGET_SCREENS;
        }
        ...
    }
    
//该方法主要用于计算最大内存。如果是低配手机就每个进程可用的最大内存乘以0.33,否则就每个进程可用的最大内存乘以0.4
private static int getMaxSize(ActivityManager activityManager) {
        final int memoryClassBytes = activityManager.getMemoryClass() * 1024 * 1024;
        //每个进程可用的最大内存
        final boolean isLowMemoryDevice = isLowMemoryDevice(activityManager);
        return Math.round(memoryClassBytes
                * (isLowMemoryDevice ? LOW_MEMORY_MAX_SIZE_MULTIPLIER : MAX_SIZE_MULTIPLIER));
    }    

    //这里直接返回已经在构造函数中计算好的内容缓存。
   public int getMemoryCacheSize() {
        return memoryCacheSize;
    }    
```
从上面的代码可以分析出来，内容缓存memoryCacheSize的大小一般为屏幕的宽*高*4*2，当然，也有可能等比例缩小。
然后图片池BitmapPool的计算就比较简单了。在``createGlide``中计算BitmapPool的方法为：
```
 if (bitmapPool == null) {
        if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.HONEYCOMB) {
            int size = calculator.getBitmapPoolSize();
            bitmapPool = new LruBitmapPool(size);
        } else {
            bitmapPool = new BitmapPoolAdapter();
        }
}
```
可以看出来，``bitmapPool``在android3.0以后用`` calculator.getBitmapPoolSize()``进行计算，这个值在前面的``MemorySizeCalculator``已经计算出来了，而在MemorySizeCalculator中只是将其返回。
```
  public int getBitmapPoolSize() {
        return bitmapPoolSize;
    }
```
可以分析出来，图片池用的是targetPoolSize(即一般是缓存大小是屏幕的宽*高*4*4)。
接下来是DecodeFormat 图片格式
```
//GlideBuilder中的createGlide方法
decodeFormat = DecodeFormat.DEFAULT;

//public enum DecodeFormat 
 public static final DecodeFormat DEFAULT = PREFER_RGB_565;
```
即默认的图片格式是PREFER_RGB_565；

最后一个Engine引擎类：
```
//GlideBuilder中的createGlide方法
if (engine == null) {
engine = new Engine(memoryCache, diskCacheFactory, diskCacheService, sourceService);
}
```
**engine里面的主要参数：**
* 内存缓存 memoryCache
* 本地缓存 diskCacheFactory
* 处理源资源的线程池 sourceService
* 处理本地缓存的线程池 diskCacheService
第一个参数就是我们前面已经计算出来的``memoryCache``，这里就不在介绍，加下来介绍第二个参数diskCacheFactory：

```
//GlideBuilder中的createGlide方法
  if (diskCacheFactory == null) {
            diskCacheFactory = new InternalCacheDiskCacheFactory(context);
}

//InternalCacheDiskCacheFactory中的方法
public InternalCacheDiskCacheFactory(Context context) {
        this(context, DiskCache.Factory.DEFAULT_DISK_CACHE_DIR, DiskCache.Factory.DEFAULT_DISK_CACHE_SIZE);
    }
    

//DiskCache接口中的内容
 interface Factory {

    //250 MB of cache. 
    int DEFAULT_DISK_CACHE_SIZE = 250 * 1024 * 1024;
    String DEFAULT_DISK_CACHE_DIR = "image_manager_disk_cache";

    //Returns a new disk cache, or {@code null} if no disk cache could be created.
    DiskCache build();
}   
```

从上面的代码我们不难看出：**默认大小为250m,默认目录:image_manager_disk_cache**。

接下来，我们继续看：
sourceService 处理源资源的线程池
```
//GlideBuilder中的createGlide方法
if (sourceService == null) {
        final int cores = Math.max(1, Runtime.getRuntime().availableProcessors());
        sourceService = new FifoPriorityThreadPoolExecutor(cores);
    }
    
//下面的方法都是在FifoPriorityThreadPoolExecutor类中，且该类继承ThreadPoolExecutor
public FifoPriorityThreadPoolExecutor(int poolSize) {
    this(poolSize, UncaughtThrowableStrategy.LOG);
}

public FifoPriorityThreadPoolExecutor(int poolSize, UncaughtThrowableStrategy uncaughtThrowableStrategy) {
    this(poolSize, poolSize, 0, TimeUnit.MILLISECONDS, new DefaultThreadFactory(),
        uncaughtThrowableStrategy);
}

public FifoPriorityThreadPoolExecutor(int corePoolSize, int maximumPoolSize, long keepAlive, TimeUnit timeUnit,
        ThreadFactory threadFactory, UncaughtThrowableStrategy uncaughtThrowableStrategy) {
    super(corePoolSize, maximumPoolSize, keepAlive, timeUnit, new PriorityBlockingQueue<Runnable>(), threadFactory);
    this.uncaughtThrowableStrategy = uncaughtThrowableStrategy;
}    
```
通过上面的源码，我们不难分析出diskCacheService其实一个线程池，里面的核心线程数为可用的处理器的个数。

最后是处理本地缓存的线程池 diskCacheService的实现，代码如下：
```
//GlideBuilder中的createGlide方法
if (diskCacheService == null) {
        diskCacheService = new FifoPriorityThreadPoolExecutor(1);
}
```
核心线程数为1，最多的线程也为1。至于FifoPriorityThreadPoolExecutor方法的实现，可以参考前面的代码。

到这里，我们的with方法的实现就解析的差不多了。最后总结一下``with``方法的实现逻辑：
with方法可以传入不同的参数，根据不同的参数和线程环境来绑定不同的生命周期。如果在子线程且SDK>=11即android3.0以后，则绑定的是Application的生命周期;如传入的是Activity且在主线程，则将gilde和activity的生命周期绑定（通过创建一个无界面的Fragment，让请求和Activity的生命周期同步），然后添加监听。


## load方法
通过的，load方法也有很多的重载。load在RequestManager类中的实现如下：
```
public DrawableTypeRequest<String> load(String string) {
    return (DrawableTypeRequest<String>) fromString().load(string);
}

public DrawableTypeRequest<Uri> load(Uri uri) {
    return (DrawableTypeRequest<Uri>) fromUri().load(uri);
}

public DrawableTypeRequest<File> load(File file) {
    return (DrawableTypeRequest<File>) fromFile().load(file);
}

public DrawableTypeRequest<Integer> load(Integer resourceId) {
    return (DrawableTypeRequest<Integer>) fromResource().load(resourceId);
}

@Deprecated
public DrawableTypeRequest<URL> load(URL url) {
    return (DrawableTypeRequest<URL>) fromUrl().load(url);
}

@Deprecated
public DrawableTypeRequest<byte[]> load(byte[] model, final String id) {
    return (DrawableTypeRequest<byte[]>) load(model).signature(new StringSignature(id));
}

public DrawableTypeRequest<byte[]> load(byte[] model) {
    return (DrawableTypeRequest<byte[]>) fromBytes().load(model);
}

public <T> DrawableTypeRequest<T> load(T model) {
    return (DrawableTypeRequest<T>) loadGeneric(getSafeClass(model)).load(model);
}
```
可以看到，load也有很多的重载方法，具体的作用和参数就不做说明了。通过源码我们可以看到，无论是那个load的重载，最终都会返回一个``DrawableTypeRequest`` 对象(继承自DrawableRequestBuilder类）
我们不可能将所有的load方法分析一篇，这里我就以load(URL url)这个方法为例进行说明，至于其他的load方法，可以自行去理解。
首先，我们在``RequestManager.java``中，我们我们看到load的实现如下：
```
@Deprecated
public DrawableTypeRequest<URL> load(URL url) {
    return (DrawableTypeRequest<URL>) fromUrl().load(url);
}

@Deprecated
public DrawableTypeRequest<URL> fromUrl() {
    return loadGeneric(URL.class);
}

private <T> DrawableTypeRequest<T> loadGeneric(Class<T> modelClass) {
    ModelLoader<T, InputStream> streamModelLoader = Glide.buildStreamModelLoader(modelClass, context);
    ModelLoader<T, ParcelFileDescriptor> fileDescriptorModelLoader =
            Glide.buildFileDescriptorModelLoader(modelClass, context);
    if (modelClass != null && streamModelLoader == null && fileDescriptorModelLoader == null) {
        throw new IllegalArgumentException("Unknown type " + modelClass + ". You must provide a Model of a type for"
                + " which there is a registered ModelLoader, if you are using a custom model, you must first call"
                + " Glide#register with a ModelLoaderFactory for your custom model class");
    }

    return optionsApplier.apply(
            new DrawableTypeRequest<T>(modelClass, streamModelLoader, fileDescriptorModelLoader, context,
                    glide, requestTracker, lifecycle, optionsApplier));
}
```
通过上面的源码，我们发现加载图片UR这一个load方法中的逻辑是非常简单的，只是简单的调用``fromUrl().load(url)``方法（先调用``fromUrl()``,后调用``load(url)``，典型的build模式）。而``fromUrl()``的代码也是非常简单的，只是调用了``loadGeneric(URL.class)``这么一个方法。然后继续来看``loadGeneric(URL.class)``方法，这个方法也比较简单，用``Glide.buildStreamModelLoader``和`` Glide.buildFileDescriptorModelLoader()``来获得``ModeiLoade``对象，这里面的具体实现就不做详细的讨论了，比较复杂，或许等我以后弄清楚了会加上。
在``loadGeneric(URL.class)``方法中，我们返回的是``DrawableTypeRequest``对象，所以最后创建了一个``DrawableTypeRequest``对象，并将相关参数应用到这个``DrawableTypeRequest``对象上，那么我们对``DrawableTypeRequest``这个对象就有必要进行以下分析了。
以下是``DrawableTypeRequest.java``的源码：
```
public class DrawableTypeRequest<ModelType> extends DrawableRequestBuilder<ModelType> implements DownloadOptions {
    private final ModelLoader<ModelType, InputStream> streamModelLoader;
    private final ModelLoader<ModelType, ParcelFileDescriptor> fileDescriptorModelLoader;
    private final RequestManager.OptionsApplier optionsApplier;

    private static <A, Z, R> FixedLoadProvider<A, ImageVideoWrapper, Z, R> buildProvider(Glide glide,
            ModelLoader<A, InputStream> streamModelLoader,
            ModelLoader<A, ParcelFileDescriptor> fileDescriptorModelLoader, Class<Z> resourceClass,
            Class<R> transcodedClass,
            ResourceTranscoder<Z, R> transcoder) {
        if (streamModelLoader == null && fileDescriptorModelLoader == null) {
            return null;
        }

        if (transcoder == null) {
            transcoder = glide.buildTranscoder(resourceClass, transcodedClass);
        }
        DataLoadProvider<ImageVideoWrapper, Z> dataLoadProvider = glide.buildDataProvider(ImageVideoWrapper.class,
                resourceClass);
        ImageVideoModelLoader<A> modelLoader = new ImageVideoModelLoader<A>(streamModelLoader,
                fileDescriptorModelLoader);
        return new FixedLoadProvider<A, ImageVideoWrapper, Z, R>(modelLoader, transcoder, dataLoadProvider);
    }

    DrawableTypeRequest(Class<ModelType> modelClass, ModelLoader<ModelType, InputStream> streamModelLoader,
            ModelLoader<ModelType, ParcelFileDescriptor> fileDescriptorModelLoader, Context context, Glide glide,
            RequestTracker requestTracker, Lifecycle lifecycle, RequestManager.OptionsApplier optionsApplier) {
        super(context, modelClass,
                buildProvider(glide, streamModelLoader, fileDescriptorModelLoader, GifBitmapWrapper.class,
                        GlideDrawable.class, null),
                glide, requestTracker, lifecycle);
        this.streamModelLoader = streamModelLoader;
        this.fileDescriptorModelLoader = fileDescriptorModelLoader;
        this.optionsApplier = optionsApplier;
    }

    /**
     * Attempts to always load the resource as a {@link android.graphics.Bitmap}, even if it could actually be animated.
     *
     * @return A new request builder for loading a {@link android.graphics.Bitmap}
     */
    public BitmapTypeRequest<ModelType> asBitmap() {
        return optionsApplier.apply(new BitmapTypeRequest<ModelType>(this, streamModelLoader,
                fileDescriptorModelLoader, optionsApplier));
    }

    /**
     * Attempts to always load the resource as a {@link com.bumptech.glide.load.resource.gif.GifDrawable}.
     * <p>
     *     If the underlying data is not a GIF, this will fail. As a result, this should only be used if the model
     *     represents an animated GIF and the caller wants to interact with the GIfDrawable directly. Normally using
     *     just an {@link com.bumptech.glide.DrawableTypeRequest} is sufficient because it will determine whether or
     *     not the given data represents an animated GIF and return the appropriate animated or not animated
     *     {@link android.graphics.drawable.Drawable} automatically.
     * </p>
     *
     * @return A new request builder for loading a {@link com.bumptech.glide.load.resource.gif.GifDrawable}.
     */
    public GifTypeRequest<ModelType> asGif() {
        return optionsApplier.apply(new GifTypeRequest<ModelType>(this, streamModelLoader, optionsApplier));
    }

    /**
     * {@inheritDoc}
     */
    public <Y extends Target<File>> Y downloadOnly(Y target) {
        return getDownloadOnlyRequest().downloadOnly(target);
    }

    /**
     * {@inheritDoc}
     */
    public FutureTarget<File> downloadOnly(int width, int height) {
        return getDownloadOnlyRequest().downloadOnly(width, height);
    }

    private GenericTranscodeRequest<ModelType, InputStream, File> getDownloadOnlyRequest() {
        return optionsApplier.apply(new GenericTranscodeRequest<ModelType, InputStream, File>(File.class, this,
                streamModelLoader, InputStream.class, File.class, optionsApplier));
    }
}
```
通过注释，我们大概也可以猜到这个类是用来做什么的。
``DrawableTypeRequest``是一个用于创建加载请求的类，该加载方法可以直接加载动画的GIF或者Bitmap drawable格式的图片。他提供了两个的方法``asBitmap()``和``asGif()``来强制指定加载静态图片和GIF图片。通过观察这两个方法的实现，发现他们的实现其实是通过创建``BitmapTypeRequest``和``GifTypeRequest``来实现的，当然默认的实现还是通过创建``DrawableTypeRequest``实现的。
但是我们发现，在``DrawableTypeRequest``这个类中，并没有``load``这个方法，而在``RequestManager.java``中的``loadGeneric``方法中，最后返回的是``DrawableTypeRequest``对象，这就说明在``DrawableTypeRequest``肯定存在load方法的实现，我们在上面的代码中无法直接看到其方法的存在，那其实现很可能存在其父类中。通过上面的代码可知，``RequestManager``的父类是``DrawableRequestBuilder``。至于其父类``DrawableRequestBuilder``的实现，这里就不全部贴出来了，只贴出``load``方法的实现：
```
   @Override
    public DrawableRequestBuilder<ModelType> load(ModelType model) {
        super.load(model);
        return this;
    }

```
当然，在``DrawableRequestBuilder``中，``load()``只是其中的一个方法而已，在这个类中，还定义了``gilde``常用的一些API方法，例如``error()``,``placeholder()``,``crossFade()``等等，至于其实现，可以参照源码去了解。下面继续来对``load()``方法进一步的深入，可以看到``load``方法调用了父类的``load``方法，接下来我们查看 其父类``GenericRequesBuilder.java``中``load``方法的实现：
```
   public GenericRequestBuilder<ModelType, DataType, ResourceType, TranscodeType> load(ModelType model) {
        this.model = model;
        isModelSet = true;
        return this;
    }
```
可以看到，最终``load()``方法的实现很简单，只是设置了加载的类型，然后就返回了``this``，当然，这个``this``最终就变成了``DrawableTypeRequest``对象。到此,``load``方法基本流程已经差不多了，下面将分析``into()``方法的实现。

## into()
前面的三个方法最终都是为了最后这个into方法做准备，所以搞懂``into``方法，对gilde源码的理解是非常非常重要的，下面我们进行``into``方法的解析。
首先，我们直接调用的``into``方法的实现是在``DrawableRequestBuilder``中实现的，具体实现的代码如下：
```

    /**
     * {@inheritDoc}
     *
     * <p>
     *     Note - If no transformation is set for this load, a default transformation will be applied based on the
     *     value returned from {@link android.widget.ImageView#getScaleType()}. To avoid this default transformation,
     *     use {@link #dontTransform()}.
     * </p>
     *
     * @param view {@inheritDoc}
     * @return {@inheritDoc}
     */
    @Override
    public Target<GlideDrawable> into(ImageView view) {
        return super.into(view);
    }
```
可以看出了，在``DrawableRequestBuilder``中，其实并没有对``into``方法进行实现，只是简单的将其实现抛给了父类``into``方法。前面就分析过，``GenericRequestBuilder``就是``DrawableRequestBuilder``的父类，下面就``into``方法的实现进行解析。
```
public Target<TranscodeType> into(ImageView view) {
        Util.assertMainThread();//判断是否在主线程
        if (view == null) {
            throw new IllegalArgumentException("You must pass in a non null View");
        }

        if (!isTransformationSet && view.getScaleType() != null) {
            switch (view.getScaleType()) {
                case CENTER_CROP:
                    applyCenterCrop();
                    break;
                case FIT_CENTER:
                case FIT_START:
                case FIT_END:
                    applyFitCenter();
                    break;
                //$CASES-OMITTED$
                default:
                    // Do nothing.
            }
        }

        return into(glide.buildImageViewTarget(view, transcodeClass));
    }
```



参考资料：
* <http://www.10tiao.com/html/169/201704/2650822526/1.html>
* <http://blog.csdn.net/yulyu/article/details/60331803>
* <http://www.lightskystreet.com/2015/10/12/glide_source_analysis/>
* <http://blog.csdn.net/guolin_blog/article/details/53939176>
* <http://blog.csdn.net/guolin_blog/article/details/54895665>