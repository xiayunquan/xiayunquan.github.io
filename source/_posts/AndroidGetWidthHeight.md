---
title: Android获取控件宽高的几种方式
date: 2021-09-24 17:29:14
categories: Android
tags: View
---

### Android获取控件宽高的几种方式

获取控件的宽高直接使用view的getWidth() 和 getHeight()方法获取。
但是直接在Activity的onCreate() 或 onResume()中获取的宽高为0。
原因是Activity的启动流程和布局文件的加载流程是2个异步的过程，在onCreate或onResume的时候控件还没有绘制完成，因此直接通过getWidth() 和 getHeight()获取的宽、高为0，下面给出几种方式来实现view宽高的获取。

##### 定义获取宽高方法

```java
/**
 * 以textView为例获取控件宽、高
 */
private void getTextWidthAndHeight() {
	int width = textView.getWidth();
	int height = textView.getHeight();
	Log.i(TAG,"width:" + width + ", height:" + height);
}
```

##### 第一种方式：在需要时获取，如控件点击时再获取

```java
button.setOnClickListener(new View.OnClickListener() {
     @Override
     public void onClick(View v) {
          getTextWidthAndHeight();     
     }
});
```

##### 第二种方式：重写onWindowFocusChanged()方法

```java
@Override
public void onWindowFocusChanged(boolean hasFocus) {
	super.onWindowFocusChanged(hasFocus);
	if (hasFocus) {
		getTextWidthHeight();
	}
}
```

需要注意的是，这个方法可能会执行多次，比如锁屏，切到后台等重新进入时都会执行该方法。



##### 第三种方式：添加OnPreDrawListener事件监听

```java
getWindow().getDecorView().getViewTreeObserver().addOnPreDrawListener(new ViewTreeObserver.OnPreDrawListener() {
    @Override
    public boolean onPreDraw() {
         getTextWidthHeight();
         getWindow().getDecorView().getViewTreeObserver().removeOnPreDrawListener(this);
         return false;
    }
});
```

##### 第四种方式：添加OnGlobalLayoutListener事件监听

```javascript
getWindow().getDecorView().getViewTreeObserver().addOnGlobalLayoutListener(new ViewTreeObserver.OnGlobalLayoutListener() {
      @Override
      public void onGlobalLayout() {
           getTextWidthHeight();
           getWindow().getDecorView().getViewTreeObserver().removeOnGlobalLayoutListener(this);
      }
});
```

##### 第五种方式：post Runnable方式
```java
 textView.post(new Runnable() {
 	@Override
	public void run() {
 		getTextWidthHeight();
 	}
 });
```

### 示例
```java
public class GetViewHeightActivity extends AppCompatActivity {

    private Button testView;
    private TextView tvHeightWidthInfo;
    private String info = "";

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_get_view_height);
        testView = findViewById(R.id.view_test);
        tvHeightWidthInfo = findViewById(R.id.tv_height_width_info);

        getHeightWidth("Default");
        onPreView();
        onGlobalLayout();
        postRunnable();

        //点击按钮的时候再获取
        testView.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
                getHeightWidth("onClick");
            }
        });
    }

    /**
     * 获取view宽高
     */
    private void getHeightWidth(String tag) {
        int height = testView.getHeight();
        int width = testView.getWidth();
        info += tag + " width:" + width + ", height:" + height + "\n";
        tvHeightWidthInfo.setText(info);
    }

    /**
     *  重写onWindowFocusChanged()方法
     * @param hasFocus 当前页面是否有焦点
     */
    @Override
    public void onWindowFocusChanged(boolean hasFocus) {
        super.onWindowFocusChanged(hasFocus);
        if (hasFocus) {
            getHeightWidth("onWindowFocusChanged");
        }
    }

    /**
     * OnPreDrawListener事件监听
     */
    private void onPreView() {
        getWindow().getDecorView().getViewTreeObserver().addOnPreDrawListener(new ViewTreeObserver.OnPreDrawListener() {
            @Override
            public boolean onPreDraw() {
                getHeightWidth("OnPreView");
                getWindow().getDecorView().getViewTreeObserver().removeOnPreDrawListener(this);
                return false;
            }
        });
    }

    /**
     * OnGlobalLayoutListener事件监听
     * 最低支持Api 16
     */
    @TargetApi(Build.VERSION_CODES.JELLY_BEAN)
    private void onGlobalLayout() {
        getWindow().getDecorView().getViewTreeObserver().addOnGlobalLayoutListener(new ViewTreeObserver.OnGlobalLayoutListener() {
            @Override
            public void onGlobalLayout() {
                getHeightWidth("OnGlobalLayout");
                getWindow().getDecorView().getViewTreeObserver().removeOnGlobalLayoutListener(this);
            }
        });
    }

    /**
     * post Runnable方式
     */
    private void postRunnable() {
        tvHeightWidthInfo.post(new Runnable() {
            @Override
            public void run() {
                getHeightWidth("PostRunnable");
            }
        });
    }
}

```

##### 布局文件R.layout.activity_get_view_height
```xml
<?xml version="1.0" encoding="utf-8"?>
<androidx.constraintlayout.widget.ConstraintLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:padding="12dp"
    tools:context=".GetViewHeightActivity">

    <Button
        android:id="@+id/view_test"
        android:layout_width="match_parent"
        android:layout_height="85dp"
        android:text="test view"
        app:layout_constraintTop_toTopOf="parent" />

    <TextView
        android:id="@+id/tv_height_width_info"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:layout_marginTop="10dp"
        android:textSize="16sp"
        app:layout_constraintTop_toBottomOf="@+id/view_test" />

</androidx.constraintlayout.widget.ConstraintLayout>
```

##### 结果截图
![示例](https://img-blog.csdnimg.cn/2019101503265465.gif)