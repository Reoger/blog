---
title: PopupWindow 使用实例
date: 2018-07-20 22:44:55
categories: android
tags: android, PopupWindows
---
# PopupWindow 使用实例
``popupWindows``是一个弹出框的界面载体，类似于``AlertDialog``，但它比``AlertDialog``更加灵活，能更加精确的指定其所在的位置。
下面记录一下自定义popupWindows的使用。

# 自定会``CustomPopup``，继承自``PopupWindow``
具体代码如下：
```

public class CustomPopup extends PopupWindow {


    // PopupWindow中控件点击事件回调接口
    private IPopuWindowListener mOnClickListener;
    //PopupWindow布局文件中的Button
    private Button alarm_pop_btn;

    /**
     * @param s
     * @description 构造方法
     * @author ldm
     * @time 2016/9/30 9:14
     */
    public CustomPopup(Context mContext, int width, int height, IPopuWindowListener listener) {
        super(mContext);

        this.mOnClickListener = listener;
        //获取布局文件
        View mContentView = LayoutInflater.from(mContext).inflate(R.layout.popupwindow_layout, null);
        //设置布局
        setContentView(mContentView);
        // 设置弹窗的宽度和高度
//        setWidth(width);
//        setHeight(height);
        setWidth(WindowManager.LayoutParams.MATCH_PARENT);
        setHeight(WindowManager.LayoutParams.WRAP_CONTENT);
        //设置能否获取到焦点
        setFocusable(false);
        //设置PopupWindow进入和退出时的动画效果
        setAnimationStyle(R.style.MyPopupWindow);
        setTouchable(true); // 默认是true，设置为false，所有touch事件无响应，而被PopupWindow覆盖的Activity部分会响应点击
        // 设置弹窗外可点击,此时点击PopupWindow外的范围，Popupwindow不会消失
        setOutsideTouchable(false);
        //外部是否可以点击，设置Drawable原因可以参考：http://blog.csdn.net/harvic880925/article/details/49278705
        setBackgroundDrawable(new BitmapDrawable());
        // 设置弹窗的布局界面
        initUI();
    }

    /**
     * 初始化弹窗列表
     */
    private void initUI() {
        //获取到按钮
        alarm_pop_btn = (Button) getContentView().findViewById(R.id.alarm_pop_btn);
        //设置按钮点击事件
        alarm_pop_btn.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View view) {
                if (null != mOnClickListener) {
                    mOnClickListener.dispose();
                }
            }
        });
    }

    /**
     * 显示弹窗列表界面
     */
    public void show(View view) {
        int[] location = new int[2];
        view.getLocationOnScreen(location);
        //Gravity.BOTTOM设置在view下方，还可以根据location来设置PopupWindowj显示的位置
        int x = location[0];
        int y = location[1];
        showAtLocation(view, Gravity.BOTTOM, x, y);
    }

    /**
     * @param null
     * @author ldm
     * @description 点击事件回调处理接口
     * @time 2016/7/29 15:30
     */
    public interface IPopuWindowListener {
        void dispose();
    }

}

```


# xml资源文件
布局资源文件：``popupwindow_layout.xml``
```
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="wrap_content"
    android:orientation="vertical"
    android:background="#ffffff"
    android:padding="20dp" >

    <Button
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:clickable="true"
        android:gravity="center"
        android:textColor="@android:color/holo_orange_dark"
        android:id="@+id/alarm_pop_btn"
        android:text="确定" />

    <TextView
        android:layout_marginTop="20dp"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:layout_marginBottom="10dp"
        android:clickable="true"
        android:gravity="center"
        android:text="取消" />

</LinearLayout>
```
类型资源文件``styles``
```
  <style name="MyPopupWindow">

        <item name="android:windowEnterAnimation">@anim/pop_in</item>
        <item name="android:windowExitAnimation">@anim/pop_out</item>
    </style>

```
动画资源文件``pop_in``
```
<?xml version="1.0" encoding="utf-8"?>
<set xmlns:android="http://schemas.android.com/apk/res/android">
    <scale
        android:duration="200"
        android:fromXScale="0"
        android:fromYScale="0"
        android:pivotX="50%"
        android:pivotY="50%"
        android:toXScale="0.8"
        android:toYScale="0.5" />
    <alpha
        android:duration="200"
        android:fromAlpha="0.0"
        android:toAlpha="1.0" />
</set>
```
动画资源文件和``pop_out``
```
<?xml version="1.0" encoding="utf-8"?>
<set xmlns:android="http://schemas.android.com/apk/res/android">
    
    <scale
        android:fromXScale="0.8"
        android:fromYScale="0.5"
        android:pivotX="50%"
        android:pivotY="50%"
        android:toXScale="0"
        android:toYScale="0"
        android:duration="200"/>
    
    <alpha
        android:duration="200"
        android:fromAlpha="1.0"
        android:toAlpha="0.0"/>

</set>
```

# 在activity中进行调用
```

public class MainActivity extends AppCompatActivity implements CustomPopup.IPopuWindowListener {

    private CustomPopup alarmPopup;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
    }



    public void testPopupWindow2(View view){
        //初始化PopupWindow这里通过数据200来设置PopupWindow高度
        alarmPopup=new CustomPopup(MainActivity.this, ViewGroup.LayoutParams.MATCH_PARENT, 200, this);//这里的this是指当前Activity实现了PopupWindow中IPopuWindowListener接口
        //弹出PopupWindow
        alarmPopup.show(view);
    }

    @Override
    public void dispose() {
        if (alarmPopup.isShowing()) {
            alarmPopup.dismiss();//关闭PopupWindow
        }
    }
}

```

---
end

# 参考链接
* <https://blog.csdn.net/harvic880925/article/details/49272285>
* <https://blog.csdn.net/true100/article/details/52785496>