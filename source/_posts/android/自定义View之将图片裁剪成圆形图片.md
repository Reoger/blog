---
title: 自定义View之将图片裁剪成圆形图片
date: 2018-08-02 16:44:55
categories: android
tags: [android,CircleImageView]
---
# 自定义View之将图片裁剪成圆形图片

## 使用PorterDuffXfermode方式来实现
在使用这中方式之前，我们有必要弄清楚``PorterBuddx``究竟是什么。
简单来说，他是定义的一种图像融合的方式，具体的模式有如下几种，效果如图：
![官方的说明图](https://upload-images.jianshu.io/upload_images/2041548-d964105abf4be5d9.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/312)

在次，不多做介绍，想深入了解``PorterDuffXfermode``可以参考[各个击破搞明白PorterDuff.Mode](https://www.jianshu.com/p/d11892bbe055)
本次使用mode是``SRCIN``,由图效果可知，我们需要在原图上绘制一个圆，原图与我们绘制得圆重叠得区域就会显示出来，原理很简单，下面开始代码实现：
```
public class CircleImageView1 extends ImageView {

    private Paint mPaint;
    private Shape mShape;
    private float mRadius;

    private float[] outerRadii = new float[8];;

    public CircleImageView1(Context context) {
        this(context,null);
    }

    public CircleImageView1(Context context, @Nullable AttributeSet attrs) {
        this(context,attrs,0);
    }

    public CircleImageView1(Context context, @Nullable AttributeSet attrs, int defStyleAttr) {
        super(context, attrs, defStyleAttr);
        initView();
    }

    private void initView() {
        setLayerType(LAYER_TYPE_HARDWARE,null);
        mPaint = new Paint(Paint.ANTI_ALIAS_FLAG);
        mPaint.setColor(Color.BLACK);
        //注意这里，将画笔的PorterDuffXfermode模式设置为了DST_IN模式！！！
        mPaint.setXfermode(new PorterDuffXfermode(PorterDuff.Mode.DST_IN));
        
    }

    @Override
    protected void onMeasure(int widthMeasureSpec, int heightMeasureSpec) {
        super.onMeasure(widthMeasureSpec, heightMeasureSpec);
        int width = getMeasuredWidth();
        int height = getMeasuredHeight();
        int min = Math.min(width,height);
        mRadius = min/2;
        setMeasuredDimension(min,min);
        //因为是圆，所以要设置为长宽相等
    }

    @Override
    protected void onLayout(boolean changed, int left, int top, int right, int bottom) {
        super.onLayout(changed, left, top, right, bottom);
        if (changed){
            if (mShape == null){
                Arrays.fill(outerRadii,mRadius);
                mShape = new RoundRectShape(outerRadii,null,null);
                //
            }
            mShape.resize(getWidth(),getHeight());
        }

    }

    @Override
    protected void onDraw(Canvas canvas) {
        int saveCount = canvas.getSaveCount();
        canvas.save();
        super.onDraw(canvas);
        if (mShape != null){
            mShape.draw(canvas,mPaint);
        }
        canvas.restoreToCount(saveCount);
    }

// 这里直接使用Glide库来加载图片，也可以在xml文件或者java代码中设置图片
    public void build(String url){
        Glide.with(getContext()).load(url).into(this);
    }
}
```
在布局中的引用：
```
<?xml version="1.0" encoding="utf-8"?>
<ScrollView xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:background="#333"
    tools:context="com.example.cm.circleview.MainActivity">

    <LinearLayout
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:gravity="center_horizontal"
        android:orientation="vertical">

        <com.example.cm.circleview.view.CircleImageView1
            android:layout_marginTop="10dp"
            android:layout_marginBottom="10dp"
            android:id="@+id/image_id1"
            android:layout_width="200dp"
            android:layout_height="200dp" />
    </LinearLayout>

</ScrollView>
```
通过java代码设置图片来源，代码如下:
```
public class MainActivity extends AppCompatActivity {

    public static final String URL = "https://timgsa.baidu.com/timg?image&quality=80&size=b9999_10000&sec=1533271076118&di=" +
            "4ef2b0b50abb957177874a76d828126a&imgtype=0&src=http%3A%2F%2Fwww.91danji.com%2Fattachments%2F201503%2F24%2F11%2F3q8s74tc8.jpeg";

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);

        CircleImageView1 circleImageView1 = findViewById(R.id.image_id1);
        circleImageView1.build(URL);
    }
}
```
因为项目使用到了``Glide``,所以需要在module的``build.grade``文件中添加必要的依赖，
```
implementation 'com.github.bumptech.glide:glide:3.8.0'
```
因为图片来自网路，需在``androidMainFest``中添加联网权限。
显示效果如图：
![效果图](http://ovec6nnof.bkt.clouddn.com/dump_4314198875772344884.png)

此次的实现都是最简单的实现，没有添加额外的逻辑，代码比较少，便于理解。
后续会在这个的基础上添加稍微复杂点的图像处理。

### 使用小结
通过该方式使得图片展现成圆形，需要开启硬件加速，否则图片的渲染效果达不到理想的效果，具体原因可以参照：[硬件加速 setlayertype](https://blog.csdn.net/hqdoremi/article/details/8307496)。
我们可以通过``shape``或者``drawable``来绘制圆，使用然后通过图片的叠加显示来显示我们需要的原型图片的效果。这种方式就是通过绘制一个圆盖在原来的图片上，通过设置两张图片叠加显示效果来实现显示圆角图片

### 优点
该方式不需要操作原来的``drawable``,使用起来风险比较小，也比较方便。

### 缺点
需要启动硬件加速，否则可能会出现意想不到的结果。

## 使用BitmapShader重新绘制实现圆形图片
整体思路： 通过对原来图片进行处理，将原图片裁剪成圆形然后绘制展示。
贴上``CircleImageView2``的代码。
```
public class CircleImageView2 extends ImageView {

    private Paint mPaint;
    private int mRadius;
    private float mScale;
    private Matrix mMatrix;

    private BitmapShader mBitmapShader;

    public static final String TAG = "CircleImageView2";

    public CircleImageView2(Context context) {
        this(context, null);
    }

    public CircleImageView2(Context context, @Nullable AttributeSet attrs) {
        this(context, attrs, 0);
    }

    public CircleImageView2(Context context, @Nullable AttributeSet attrs, int defStyleAttr) {
        super(context, attrs, defStyleAttr);
        initView(attrs);
    }

    private void initView(AttributeSet attrs) {

        mPaint = new Paint(Paint.ANTI_ALIAS_FLAG);
        mPaint.setAntiAlias(true);
        mMatrix = new Matrix();
    }


    @Override
    protected void onMeasure(int widthMeasureSpec, int heightMeasureSpec) {
        super.onMeasure(widthMeasureSpec, heightMeasureSpec);
        int min = Math.min(getMeasuredHeight(), getMeasuredWidth());
        mRadius = min / 2;
        setMeasuredDimension(min, min);
        //强制设置成宽度和高度一致
    }

    @Override
    protected void onDraw(Canvas canvas) {
        if (mBitmapShader == null) {
            Bitmap bitmap = drawableToBitmap(getDrawable());
            //CLAMP表示，当所画图形的尺寸大于Bitmap的尺寸的时候，会用Bitmap四边的颜色填充剩余空间。
            //REPEAT表示，当我们绘制的图形尺寸大于Bitmap尺寸时，会用Bitmap重复平铺整个绘制的区域
            //MIRROR与REPEAT类似，当绘制的图形尺寸大于Bitmap尺寸时，MIRROR也会用Bitmap重复平铺整个绘图区域，与REPEAT不同的是，两个相邻的Bitmap互为镜像。
            mBitmapShader = new BitmapShader(bitmap, Shader.TileMode.CLAMP, Shader.TileMode.CLAMP);
            //计算缩放比例
            mScale = (mRadius * 2.0f) / Math.min(bitmap.getHeight(), bitmap.getWidth());
            mMatrix.setScale(mScale, mScale);
        }
        //设置图片缩放
        mBitmapShader.setLocalMatrix(mMatrix);
        mPaint.setShader(mBitmapShader);
        canvas.drawCircle(mRadius, mRadius, mRadius, mPaint);
    }

    // 将圆图片转化成bitmap
    private Bitmap drawableToBitmap(Drawable drawable) {
        if (drawable instanceof BitmapDrawable) {
            return ((BitmapDrawable) drawable).getBitmap();
        }
        int width = drawable.getIntrinsicWidth();
        int height = drawable.getIntrinsicHeight();
        //ARGB_8888 表示颜色色由4个8位组成即32位
        Bitmap bitmap = Bitmap.createBitmap(width, height, Bitmap.Config.ARGB_8888);
        Canvas canvas = new Canvas(bitmap);
        drawable.setBounds(0, 0, width, height);
        drawable.draw(canvas);
        return bitmap;
    }

    public void build(String url) {
        Glide.with(getContext())
                .load(url)
                .into(this);
    }
}
```
**实现逻辑：**
首先将获得的``drawable``转化成``bitmap``,然后通过[BitmapShader](https://www.cnblogs.com/tianzhijiexian/p/4298660.html)处理``bitmap``,并将其设置给画笔，当使用此画笔绘制时，该``bitmap``就会展示出来，如此我们只需要使用画布绘制一个圆形即可。

### 优点
可以实现对图片复杂的处理，可扩展性较强。
### 缺点
实现起来较为复杂，且有对

# 通过clipPath裁剪的方式
这种方式最简单暴力，直接在原来的图片上进行裁剪就ok了，如果不需要对图片进行处理的画，推荐用这种方式，因为，简单，粗暴！
直接上代码：
```
public class CircleImageView3 extends ImageView {

    private float mRadius;
    private Path mPath;

    public CircleImageView3(Context context) {
        this(context,null);
    }

    public CircleImageView3(Context context, @Nullable AttributeSet attrs) {
        this(context, attrs,0);
    }

    public CircleImageView3(Context context, @Nullable AttributeSet attrs, int defStyleAttr) {
        super(context, attrs, defStyleAttr);
        initView();
    }

    private void initView() {
        mPath = new Path();
    }

    @Override
    protected void onMeasure(int widthMeasureSpec, int heightMeasureSpec) {
        super.onMeasure(widthMeasureSpec, heightMeasureSpec);
        int min = Math.min(getMeasuredHeight(),getMaxWidth());
        mRadius = min/2;
    }

    @Override
    protected void onDraw(Canvas canvas) {
        mPath.addCircle(mRadius, mRadius, mRadius,Path.Direction.CW);
        canvas.clipPath(mPath);
        super.onDraw(canvas);
    }

    public void build(String url) {
        Glide.with(getContext())
                .load(url)
                .into(this);
    }
}
```
没什么好说的，关键就一句`` canvas.clipPath(mPath);``，需要关注一下[path](https://blog.csdn.net/xiangzhihong8/article/details/78278931)的用法。

### 优点
简单粗暴，不涉及``drwable`` 相关的操作，效率较高，实现也简单。
### 缺点
因为是对图片的直接剪裁，不能实现对图片相关的处理，也没有抗锯齿效果，如果图片不是正方形图片可能会影响最终的效果。


源程序下载地址:<https://github.com/Reoger/CurtomViewTest>

# 总结
上述三种方式都能实现圆形图片的需求，但是各有优缺点。如果只是需要展示圆形图片而不需要对图片进行任何处理，可以选用第三种方式，需要对图片进行比较复杂的处理的画，则推荐第一种或者第二种方式。
本篇博客到此结束，下一篇将会使用上述的三种方式实现圆形外边框的效果。

# 参考链接
* [Android Xfermode 实战 实现圆形、圆角图片](https://blog.csdn.net/lmj623565791/article/details/42094215)
* [Android 完美实现图片圆角和圆形（对实现进行分析](https://blog.csdn.net/lmj623565791/article/details/24555655)
* [Android圆形图片不求人，自定义View实现（BitmapShader使用）](https://blog.csdn.net/a369414641/article/details/53198671)
* [关于圆角ImageView的几种实现方式](https://www.jianshu.com/p/626dbd93207d)
* [Android中Canvas绘图之Shader使用图文详解](https://blog.csdn.net/iispring/article/details/50500106)
* [ShapedImageView](https://github.com/gavinliu/ShapedImageView)
* [CircleImageView](https://github.com/hdodenhof/CircleImageView)
* [自定义控件三部曲之绘图篇（十八）——BitmapShader与望远镜效果](https://blog.csdn.net/harvic880925/article/details/52039081)
* [自定义控件其实很简单5/12](https://blog.csdn.net/aigestudio/article/details/41960507)
* [Android开发之Path详解](https://blog.csdn.net/xiangzhihong8/article/details/78278931)