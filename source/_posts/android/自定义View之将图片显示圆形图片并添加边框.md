---
title: 自定义View之将图片显示圆形图片并添加边框
date: 2018-08-04 16:44:55
categories: android
tags: android
---
# 效果预览
![效果预览](http://ovec6nnof.bkt.clouddn.com/image1.png)

本次的图片效果是基于上一篇博客的三种方法进行扩展实现图片的外边框效果。上一篇博客的[传送门](https://blog.csdn.net/Reoger/article/details/81394579)。
下面开始进行到整体，针对上次实现圆形图片的三个方式添加一个外边框效果。

# 使用PorterDuffXfermode方式来实现
我们使用这个方式实现圆形图片的原理是通过图片的设置图片的叠加显示效果。还是这张图：
![官方的说明图](https://upload-images.jianshu.io/upload_images/2041548-d964105abf4be5d9.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/312)

## 思路
实现圆形图片，我们只需要绘制一个圆，并将模式设置为``SRC_IN``即可。但是我们需要在外围添加一个边框的话，就不会那么简单了。我们需要绘制三次图片，通过合理的设置他们的``Xfermode``模式来达到我们的要求。具体步骤为：
1. 将原图片绘制出来(如果有需要，进行必要的比例缩放处理，使其是一个正方形)
2. 绘制一个等大的正方形，颜色为边框要显示的颜色。(这一步的目的是为了将外围的边框也绘制出来)
3. 绘制一个半径为二分之一边长减去边框长度的圆，模式为``DST_OUT``
4. 绘制一个半径为二分之一边长的圆，模式为``DST_IN``
通过上面四步，基本就实现了我们想要的外边框的效果。
用图表示为：
![](http://ovec6nnof.bkt.clouddn.com/%E6%BC%94%E7%A4%BA%E6%96%87%E7%A8%BF1.png)
具体细节可以参考代码实现。
## 代码
自定义view``CircleImageView1``的代码如下。
```
public class CircleImageView1 extends ImageView {

    public static final int DEFAULT_BORDER_WIDTH = 0;

    private Paint mPaint;
    private Paint mBorderPaint;
    private Shape mShape;
    private Shape mBorderShape;
    private float mRadius;
    private float mBorderWidth;
    private int mBorderColor;
    private PorterDuffXfermode mBorderDuffMode;
    private Bitmap mBorderBitmap;

    private float[] outerRadii = new float[8];

    public CircleImageView1(Context context) {
        this(context, null);
    }

    public CircleImageView1(Context context, @Nullable AttributeSet attrs) {
        this(context, attrs, 0);
    }

    public CircleImageView1(Context context, @Nullable AttributeSet attrs, int defStyleAttr) {
        super(context, attrs, defStyleAttr);
        initView(attrs);
    }

    private void initView(AttributeSet attrs) {
        if (attrs != null) {
            TypedArray a = getContext().obtainStyledAttributes(attrs, R.styleable.CircleImageView1);
            mBorderWidth = a.getDimensionPixelSize(R.styleable.CircleImageView1_circle_border_width1, DEFAULT_BORDER_WIDTH);
            mBorderColor = a.getInt(R.styleable.CircleImageView1_circle_border_color1, Color.BLACK);
            a.recycle();
        }
        setLayerType(LAYER_TYPE_HARDWARE, null);
        mPaint = new Paint(Paint.ANTI_ALIAS_FLAG);
        mPaint.setColor(Color.BLACK);
        mPaint.setXfermode(new PorterDuffXfermode(PorterDuff.Mode.DST_IN));

        if (mBorderWidth != DEFAULT_BORDER_WIDTH) {
            mBorderPaint = new Paint(Paint.ANTI_ALIAS_FLAG);
            mBorderPaint.setColor(mBorderColor);
            mBorderPaint.setFilterBitmap(true);
            mBorderDuffMode = new PorterDuffXfermode(PorterDuff.Mode.DST_OUT);
        }
    }

    @Override
    protected void onMeasure(int widthMeasureSpec, int heightMeasureSpec) {
        super.onMeasure(widthMeasureSpec, heightMeasureSpec);
        int width = getMeasuredWidth();
        int height = getMeasuredHeight();
        int min = Math.min(width, height);
        mRadius = min / 2;
        setMeasuredDimension(min, min);
    }

    @Override
    protected void onLayout(boolean changed, int left, int top, int right, int bottom) {
        super.onLayout(changed, left, top, right, bottom);
        if (changed) {
            if (mShape == null) {
                Arrays.fill(outerRadii, mRadius);
                mShape = new RoundRectShape(outerRadii, null, null);
                if (mBorderWidth != DEFAULT_BORDER_WIDTH && mBorderShape == null) {
                    mBorderShape = new RoundRectShape(outerRadii, null, null);
                }

            }
            mShape.resize(getWidth(), getHeight());
            if (mBorderWidth != DEFAULT_BORDER_WIDTH && mBorderShape != null) {
                mBorderShape.resize(getWidth() - mBorderWidth * 2, getHeight() - mBorderWidth * 2);
                makeStrokeBitmap();
            }
        }
    }

    @Override
    protected void onDraw(Canvas canvas) {
        int saveCount = canvas.getSaveCount();
        canvas.save();
        super.onDraw(canvas);
        if (mBorderWidth != DEFAULT_BORDER_WIDTH && mBorderShape != null && mBorderBitmap != null) {
            int i = canvas.saveLayer(0, 0, getMeasuredWidth(), getMeasuredHeight(), null, Canvas.ALL_SAVE_FLAG);
            //1.绘制外边框颜色的正方形
            mBorderPaint.setXfermode(null);
            canvas.drawBitmap(mBorderBitmap, 0, 0, mBorderPaint);
            //2.适当画布的中心位置
            canvas.translate(mBorderWidth, mBorderWidth);
            //3.绘制减去边框长度的图片，模式为DST_OUT
            mBorderPaint.setXfermode(mBorderDuffMode);
            mBorderShape.draw(canvas, mBorderPaint);
            canvas.restoreToCount(i);
        }

        if (mShape != null) {
            mShape.draw(canvas, mPaint);
        }
        canvas.restoreToCount(saveCount);
    }

    private void makeStrokeBitmap() {
        if (mBorderWidth <= 0) return;

        int w = getMeasuredWidth();
        int h = getMeasuredHeight();

        if (w == 0 || h == 0) {
            return;
        }

        releaseBorderBitmap();

        mBorderBitmap = Bitmap.createBitmap(w, h, Bitmap.Config.ARGB_8888);
        Canvas c = new Canvas(mBorderBitmap);
        Paint p = new Paint(Paint.ANTI_ALIAS_FLAG);
        p.setColor(mBorderColor);
        c.drawRect(new RectF(0, 0, w, h), p);
    }

    private void releaseBorderBitmap() {
        if (mBorderBitmap != null) {
            mBorderBitmap.recycle();
            mBorderBitmap = null;
        }
    }


    public void build(String url) {
        Glide.with(getContext()).load(url).into(this);
    }
}
```
在代码中使用到了自定义的属性，所以需要在``value/attrs``文件中添加支持，代码如下：
```
<?xml version="1.0" encoding="utf-8"?>
<resources>
    <declare-styleable name="CircleImageView1">
        <attr name="circle_border_width1" format="dimension" />
        <attr name="circle_border_color1" format="color" />
        <attr name="circle_border_overlay1" format="boolean" />
        <attr name="circle_fill_color1" format="color" />
        <attr name="circle_circle_background_color1" format="color" />
    </declare-styleable>
</resources>
```
使用也很简单，在xml中使用的代码如下：
```
 <com.example.cm.circleview.view2.CircleImageView1
            android:id="@+id/image2_id1"
            app:circle_border_width1="5dp"
            app:circle_border_color1="#fff"
            android:layout_width="200dp"
            android:layout_height="200dp"
            android:layout_marginTop="10dp"
            android:layout_marginBottom="10dp" />
```
通过java代码设置图片来源，代码如下：
```
public class MainActivity extends AppCompatActivity {

    public static final String URL = "https://timgsa.baidu.com/timg?image&quality=80&size=b9999_10000&sec=1533911636&di=04ec95ec44a1623d52b20ff08727ac78&imgtype=jpg&er=1&src=http%3A%2F%2Fimg4.duitang.com%2Fuploads%2Fitem%2F201312%2F05%2F20131205172342_P3nwv.jpeg";

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);

        CircleImageView1 circleImageView21 = findViewById(R.id.image2_id1);
        circleImageView21.build(URL);


    }
}
```

## 优点
能够实现对图片不同边框形状的支持。
## 缺点
实现较复杂，性能代价也比较高，且不支持设置外边框透明度。

# 使用BitmapShader重新绘制实现圆形图片
## 思路
和上一种方式比起来，第二种方式就显得简单很多了。主要思路就是利用``bitmapShader``将原图绘制出来，然后通过`` canvas.drawCircle``绘制我们想要的边框即可。
## 代码
这里只贴出自定义view``CircleImageView2``的代码，完整代码可以参考我后面给出的代码链接。
```

public class CircleImageView2 extends ImageView {

    public static final int DEFAULT_BORDER_WIDTH = 0;
    private Paint mBorderPaint;
    private Paint mPaint;
    private int mRadius;
    private Matrix mMatrix;
    private int mBorderWidth;
    private int mBorderColor;
    private BitmapShader mBitmapShader;


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
        if (attrs != null) {
            TypedArray a = getContext().obtainStyledAttributes(attrs, R.styleable.CircleImageView2);
            mBorderWidth = a.getDimensionPixelSize(R.styleable.CircleImageView2_circle_border_width2, DEFAULT_BORDER_WIDTH);
            mBorderColor = a.getInt(R.styleable.CircleImageView2_circle_border_color2, Color.BLACK);
            a.recycle();
        }
        super.setScaleType(ScaleType.CENTER_CROP);
        mPaint = new Paint(Paint.ANTI_ALIAS_FLAG);
        mPaint.setAntiAlias(true);
        mMatrix = new Matrix();

        if (mBorderWidth != DEFAULT_BORDER_WIDTH) {
            mBorderPaint = new Paint();
            mBorderPaint.setStyle(Paint.Style.STROKE);
            mBorderPaint.setAntiAlias(true);
            mBorderPaint.setColor(mBorderColor);
            mBorderPaint.setStrokeWidth(mBorderWidth);

        }

    }


    @Override
    protected void onMeasure(int widthMeasureSpec, int heightMeasureSpec) {
        super.onMeasure(widthMeasureSpec, heightMeasureSpec);
        int min = Math.min(getMeasuredHeight(), getMeasuredWidth());
        mRadius = min / 2;
        setMeasuredDimension(min, min);
    }

    @Override
    protected void onLayout(boolean changed, int left, int top, int right, int bottom) {
        super.onLayout(changed, left, top, right, bottom);

    }

    @Override
    protected void onDraw(Canvas canvas) {
        if (mBitmapShader == null) {
            Bitmap bitmap = drawableToBitmap(getDrawable());
            //CLAMP表示，当所画图形的尺寸大于Bitmap的尺寸的时候，会用Bitmap四边的颜色填充剩余空间。
            //REPEAT表示，当我们绘制的图形尺寸大于Bitmap尺寸时，会用Bitmap重复平铺整个绘制的区域
            //MIRROR与REPEAT类似，当绘制的图形尺寸大于Bitmap尺寸时，MIRROR也会用Bitmap重复平铺整个绘图区域，与REPEAT不同的是，两个相邻的Bitmap互为镜像。
            if (bitmap == null) {
                super.onDraw(canvas);
                return;
            }

            mBitmapShader = new BitmapShader(bitmap, Shader.TileMode.CLAMP, Shader.TileMode.CLAMP);
            //这里可以进行图片相关的处理，注释的代码是设置比例缩放
//            float mScale = (mRadius * 2.0f) / Math.min(bitmap.getHeight(), bitmap.getWidth());
//            mMatrix.setScale(mScale, mScale);
//            mBitmapShader.setLocalMatrix(mMatrix);
        }

        mPaint.setShader(mBitmapShader);
        canvas.drawCircle(mRadius, mRadius, mRadius, mPaint);
        if (mBorderWidth != DEFAULT_BORDER_WIDTH) {
            canvas.drawCircle(mRadius, mRadius, mRadius-mBorderWidth/2, mBorderPaint);
        }
    }

    private Bitmap drawableToBitmap(Drawable drawable) {
        if (drawable == null)
            return null;
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
与用这种方式实现没有边框的圆形图片相比，并添加复杂的逻辑，只是再次基础上画了一个圆，我们将画圆的画笔``setStrokeWidth``设置我们要实现的边框即可。
## 优点
实现比较简单，支持复杂的图片处理。
## 缺点
需要两次处理``drawable``，性能还是有一点的开销。

# 通过clipPath裁剪的方式
## 思路
我们在裁剪好的图片上画一个圆边框即可。与第二种方式实现圆类似。
## 代码
```
public class CircleImageView3 extends ImageView {

    public static final int DEFAULT_BORDER_WIDTH = 0;
    private float mRadius;
    private int mBorderWidth;
    private int mBorderColor;
    private Paint mBorderPaint;
    private Path mPath;

    public CircleImageView3(Context context) {
        this(context,null);
    }

    public CircleImageView3(Context context, @Nullable AttributeSet attrs) {
        this(context, attrs,0);
    }

    public CircleImageView3(Context context, @Nullable AttributeSet attrs, int defStyleAttr) {
        super(context, attrs, defStyleAttr);
        initView(attrs);
    }

    private void initView(@Nullable AttributeSet attrs) {
        if (attrs!=null){
            TypedArray a = getContext().obtainStyledAttributes(attrs, R.styleable.CircleImageView3);
            mBorderWidth = a.getDimensionPixelSize(R.styleable.CircleImageView3_circle_border_width3, DEFAULT_BORDER_WIDTH);
            mBorderColor = a.getInt(R.styleable.CircleImageView3_circle_border_color3, Color.BLACK);
            a.recycle();
        }
        if (mBorderWidth != DEFAULT_BORDER_WIDTH){
            mBorderPaint = new Paint();
            mBorderPaint.setStyle(Paint.Style.STROKE);
            mBorderPaint.setAntiAlias(true);
            mBorderPaint.setColor(mBorderColor);
            mBorderPaint.setStrokeWidth(mBorderWidth);
        }
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
        canvas.drawCircle(mRadius, mRadius, mRadius-mBorderWidth/2, mBorderPaint);
    }

    public void build(String url) {
        Glide.with(getContext())
                .load(url)
                .into(this);
    }
}

```
## 优点
实现简单，不需要额外操作``drawble``，简单粗暴。
## 缺点
不支持复杂的图片处理。

源代码链接:<https://github.com/Reoger/CurtomViewTest>
上一篇链接:<https://blog.csdn.net/Reoger/article/details/81394579>
