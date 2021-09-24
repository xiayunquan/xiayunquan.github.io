---
title: Android控件-TabLayout使用介绍
date: 2021-09-24 17:29:33
categories: Android
tags: View
---

## 简述
TabLayout是Android support中的一个控件android.support.design.widget.TabLayout，Google在升级了AndroidX之后，将TabLayout迁移到material包下面去了com.google.android.material.tabs.TabLayout，原来的support下面的TabLayout从API 29开始就不再维护了。所以如果项目已经升级了AndroidX，建议直接使用后者。TabLayout一般结合ViewPager+Fragment的使用实现滑动的标签选择器。
比如新闻标签切换
![在这里插入图片描述](https://img-blog.csdnimg.cn/2020050612022865.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2NzZG54aWE=,size_16,color_FFFFFF,t_70#pic_center)
看下TabLayout的继承关系，如下图
support包下面的
![继承关系](https://img-blog.csdnimg.cn/20200506114959629.png#pic_center)
material包下面的
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200506154511501.png#pic_center)

TabLayout继承的HorizontalScrollView，所以支持左右滑动，下面写的简单的例子看看效果。
## 简单示例
**activity_tab.xml**

```xml
<?xml version="1.0" encoding="utf-8"?>
<androidx.constraintlayout.widget.ConstraintLayout 
	xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    xmlns:app="http://schemas.android.com/apk/res-auto">

    <com.google.android.material.tabs.TabLayout
        android:id="@+id/tab_layout"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        app:layout_constraintTop_toTopOf="parent"
        app:layout_constraintStart_toStartOf="parent"
        app:tabTextColor="@color/colorPrimary"
        app:tabSelectedTextColor="@color/colorPrimaryDark"
        />
    <androidx.viewpager.widget.ViewPager
        android:id="@+id/view_pager"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        app:layout_constraintTop_toBottomOf="@+id/tab_layout"
        />

</androidx.constraintlayout.widget.ConstraintLayout>
```
ConstraintLayout 布局里面就一个TabLayout和一个ViewPager，tabSelectedTextColor和tabTextColor属性分别设置标签选中和未选中状态的文字颜色，其他属性后面介绍。
**TabFragment.java**

```java
public class TabFragment extends Fragment {
    public static TabFragment newInstance(String label) {
        Bundle args = new Bundle();
        args.putString("label", label);
        TabFragment fragment = new TabFragment();
        fragment.setArguments(args);
        return fragment;
    }
    
    @Nullable
    @Override
    public View onCreateView(@NonNull LayoutInflater inflater, @Nullable ViewGroup container, @Nullable Bundle savedInstanceState) {
        return inflater.inflate(R.layout.fragment_tab, container, false);
    }

    @Override
    public void onStart() {
        super.onStart();
        String label = getArguments().getString("label");
        TextView text = getView().findViewById(R.id.tv_bg);
        text.setText(label);
        text.setBackgroundColor(Color.rgb((int)(Math.random() * 255), (int)(Math.random() * 255), (int)(Math.random() * 255)));
    }
}
```
TabFragment中就一个TextView，通过给每一个Fragment中的TextView设置不同的text和背景颜色来区分当前是哪一个Fragment。Fragment的布局文件如下：
**fragment_tab.xml**
```xml
<?xml version="1.0" encoding="utf-8"?>
<TextView xmlns:android="http://schemas.android.com/apk/res/android"
    android:id="@+id/tv_bg"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:gravity="center_horizontal"
    android:paddingTop="100dp"
    />
```

**TabActivity.java**

```java
public class TabActivity extends AppCompatActivity {

    private String[] tabs = {"tab1", "tab2", "tab3"};
    private List<TabFragment> tabFragmentList = new ArrayList<>();

    @Override
    protected void onCreate(@Nullable Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_tab);
        TabLayout tabLayout = findViewById(R.id.tab_layout);
        ViewPager viewPager = findViewById(R.id.view_pager);
        //添加tab
        for (int i = 0; i < tabs.length; i++) {
            tabLayout.addTab(tabLayout.newTab().setText(tabs[i]));
            tabFragmentList.add(TabFragment.newInstance(tabs[i]));
        }
        
        viewPager.setAdapter(new FragmentPagerAdapter(getSupportFragmentManager(), FragmentPagerAdapter.BEHAVIOR_RESUME_ONLY_CURRENT_FRAGMENT) {
            @NonNull
            @Override
            public Fragment getItem(int position) {
                return tabFragmentList.get(position);
            }

            @Override
            public int getCount() {
                return tabFragmentList.size();
            }

            @Nullable
            @Override
            public CharSequence getPageTitle(int position) {
                return tabs[position];
            }
        });
        
        //设置TabLayout和ViewPager联动
        tabLayout.setupWithViewPager(viewPager,false);
    }
}
```
简单的3步：

  1. 为TabLayout添加tab
  2. 给ViewPager设置adapter
  3. 设置TabLayout和ViewPager联动

看看运行结果：
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200506171118192.gif#pic_center)
好了，基本的功能已经实现了。这里有个需要注意的点：
就是在给ViewPager设置Adapter的时候，一定要重写getPageTitle(int position)方法，不然TabLayout中的标签是看不到的，即使在addTab时newTab().setText(tabs[i])也没用。原因很简单，是在tabLayout.setupWithViewPager的时候，TabLayout中先将所有tabs remove了，然后取的PagerAdapter中的getPageTitle返回值添加的tab。看下源码

```java
@ViewPager.DecorView
public class TabLayout extends HorizontalScrollView {
	...
	private void setupWithViewPager(
	      @Nullable final ViewPager viewPager, boolean autoRefresh, boolean implicitSetup) {
		if (viewPager != null) {
			this.viewPager = viewPager;
			...
			final PagerAdapter adapter = viewPager.getAdapter();
	      	if (adapter != null) {
	        	// Now we'll populate ourselves from the pager adapter, adding an observer 	if
	        	// autoRefresh is enabled
	        	setPagerAdapter(adapter, autoRefresh);
	      	}
	      	...
	    } else {
	      // We've been given a null ViewPager so we need to clear out the internal state,
	      // listeners and observers
	      this.viewPager = null;
	      setPagerAdapter(null, false);
	    }
	}
	
	void setPagerAdapter(@Nullable final PagerAdapter adapter, final boolean addObserver) {
		...
	    // Finally make sure we reflect the new adapter
	    populateFromPagerAdapter();
	  }
	
	void populateFromPagerAdapter() {
	    removeAllTabs();
	
	    if (pagerAdapter != null) {
	      final int adapterCount = pagerAdapter.getCount();
	      for (int i = 0; i < adapterCount; i++) {
	        addTab(newTab().setText(pagerAdapter.getPageTitle(i)), false);
	      }
	      ...
	    }
	}
	...
}
```
可以看到populateFromPagerAdapter方法中执行了removeAllTabs()方法，然后取pagerAdapter.getPageTitle(i)方法返回值重新添加tab，所以记得重写getPageTitle方法。
除了在代码里面动态的添加tab，还可以直接在xml中进行添加TabItem。

```xml
<com.google.android.material.tabs.TabLayout
    android:id="@+id/tab_layout"
    android:layout_width="match_parent"
    android:layout_height="40dp"
    app:layout_constraintTop_toTopOf="parent"
    app:layout_constraintStart_toStartOf="parent"
    app:tabTextColor="@color/grey"
    app:tabSelectedTextColor="@color/colorAccent"
    >
    <com.google.android.material.tabs.TabItem
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="tab1"/>
    <com.google.android.material.tabs.TabItem
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="tab2"/>
</com.google.android.material.tabs.TabLayout>
```
![tab item](https://img-blog.csdnimg.cn/20200508100706903.gif#pic_center)

## TabLayout属性介绍
TabLayout有很多属性可以供我们使用，下面简单介绍几个。

### tabIndicatorFullWidth
上面的运行结果可以看到指示器的整个宽度是充满屏幕的，有时项目需要指示器线条的宽度和文字得宽度一致，那么就可以设置tabIndicatorFullWidth属性为false，默认为true
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200506191431655.gif#pic_center)
### tabRippleColor
默认点击每一个tab的时候，会出现渐变的背景色
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200506192949908.gif#pic_center)
如果想要去掉这个点击时的背景，可以通过设置tabRippleColor属性值为一个透明的背景色就可以了

```xml
<com.google.android.material.tabs.TabLayout
        ...
        app:tabRippleColor="@android:color/transparent"/>
```
### tabTextAppearance
有时候如果设计师需要我们实现，选中的tab文字字体加粗并放大字号，但是TabLayout并没有直接设置字体大小样式的属性，这时候就可以通过设置自定义属性tabTextAppearance来实现，其值是一个自定义style。

```xml
<style name="TabLayoutTheme">
        <item name="android:textSize">16sp</item>
        <item name="android:textStyle">bold</item>
</style>

<com.google.android.material.tabs.TabLayout
        ...
        app:tabTextAppearance="@style/TabLayoutTheme"
        >
```
![appearance](https://img-blog.csdnimg.cn/20200506195139656.gif#pic_center)
可以看到所有的tab字体都变了，不是我们想要的效果。TabLayout可以设置OnTabSelectedListener监听事件，可以通过选中状态的改变来动态的设置tab样式。下面看看具体的实现逻辑
创建一个tab_text_layout.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<TextView xmlns:android="http://schemas.android.com/apk/res/android"
    android:id="@android:id/text1"
    android:layout_width="wrap_content"
    android:layout_height="match_parent"
    android:gravity="center"
    />
```
然后在styles.xml中新建选中和未选中的style

```xml
<style name="TabLayoutTextSelected">
        <item name="android:textSize">16sp</item>
        <item name="android:textStyle">bold</item>
        <item name="android:textColor">@color/colorAccent</item>
</style>

 <style name="TabLayoutTextUnSelected">
        <item name="android:textSize">14sp</item>
        <item name="android:textStyle">normal</item>
        <item name="android:textColor">@color/grey</item>
</style>
```
最后给TabLayout添加监听事件

```java
tabLayout.addOnTabSelectedListener(new TabLayout.OnTabSelectedListener() {
	@Override
    public void onTabSelected(TabLayout.Tab tab) {
       View customView = tab.getCustomView();
         if (customView == null) {
             tab.setCustomView(R.layout.tab_text_layout);
         }
         TextView textView = tab.getCustomView().findViewById(android.R.id.text1);
         textView.setTextAppearance(TabActivity.this, R.style.TabLayoutTextSelected);
     }

     @Override
     public void onTabUnselected(TabLayout.Tab tab) {
         View customView = tab.getCustomView();
         if (customView == null) {
             tab.setCustomView(R.layout.tab_text_layout);
         }
         TextView textView = tab.getCustomView().findViewById(android.R.id.text1);
         textView.setTextAppearance(TabActivity.this, R.style.TabLayoutTextUnSelected);
     }

     @Override
     public void onTabReselected(TabLayout.Tab tab) {
     }
});
```
获取tab中的CustomView，为空则设置成我们自己创建的tab_text_layout，然后找到textView的id，最后给textView设置TextAppearance属性。
这里需要注意的是textView的id必须是`android.R.id.text1`，因为从源码中可以看到CustomView获取的是android.R.id.text1。

```java
public class TabLayout extends HorizontalScrollView {
	...
	public final class TabView extends LinearLayout {
		...
		final void update() {
			final Tab tab = this.tab;
	      	final View custom = tab != null ? tab.getCustomView() : null;
	      	...
	        customTextView = custom.findViewById(android.R.id.text1);
	        ...
		}
	}
}
```
一切修改完毕，重新运行看看效果：
![fail](https://img-blog.csdnimg.cn/20200507144309934.gif#pic_center)
好像有效果了，但是还有点小问题，第一次进去的时候未选中的tab还是使用的默认样式，我们自定义的样式并没有生效，原因是未选中的tab并没有执行到OnTabSelectedListener中的onTabUnselected方法。解决办法是：只需要给TabLayout控件的tabTextAppearance属性设置一个初始的TabLayoutTextUnSelected样式即可。

```xml
<com.google.android.material.tabs.TabLayout
	...
    app:tabTextAppearance="@style/TabLayoutTextUnSelected" />
```
然后再看看效果：
![success](https://img-blog.csdnimg.cn/20200507145026963.gif#pic_center)
### tabMode
tabMode属性用于设置tab是否可以横向滚动，可选的值有fixed(默认)、auto、scrollable。
为了看到更加明显的效果对比，这里又新增几个tab，当设置默认的fixed时，所有的tab都会挤到屏幕上显示
![fixed](https://img-blog.csdnimg.cn/20200507151138193.gif#pic_center)
当设置scrollable时
![scrollable](https://img-blog.csdnimg.cn/2020050715122054.gif#pic_center)
一般tab较少时使用fixed，tab较多时使用scrollable，视项目而定。

### tabIndicatorColor
这是属性设置指示器线条的颜色，没什么好讲的

### tabIndicatorHeight
这个属性设置指示器的高度，如果我们不需要显示指示器，则可以通过设置tabIndicatorHeight等于0来实现
![height is 0](https://img-blog.csdnimg.cn/202005071537246.gif#pic_center)
### tabIndicatorGravity
这个属性可以设置指示器的显示位置，可选值有bottom(默认)、center、top、stretch。
**bottom**
![bottom](https://img-blog.csdnimg.cn/20200508095158160.gif#pic_center)

**center**
![center](https://img-blog.csdnimg.cn/20200508095211987.gif#pic_center)

**top**
![top](https://img-blog.csdnimg.cn/202005080952264.gif#pic_center)

**stretch**
![stretch](https://img-blog.csdnimg.cn/20200508095239287.gif#pic_center)
还有一些其他的属性，可以去[官网](https://developer.android.google.cn/reference/com/google/android/material/tabs/TabLayout?hl=en)进行了解。

## 给TabLayout设置间隔drawable
TabLayout默认每个Tab之间是没有间隔的，实际项目中可能需要给每个Tab之间设置一个小竖线什么的，那么可以通过下面的方法来实现。

```java
LinearLayout linearLayout = (LinearLayout) tabLayout.getChildAt(0);
linearLayout.setShowDividers(LinearLayout.SHOW_DIVIDER_MIDDLE);
linearLayout.setDividerDrawable(ContextCompat.getDrawable(this,
                R.drawable.layout_divider_vertical));
```
自定义一个drawable文件，这里自定义一个线条
layout_divider_vertical.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<shape xmlns:android="http://schemas.android.com/apk/res/android">
    <solid android:color="#ff0000"/>
    <size android:width="1dp"/>
</shape>
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200508170421302.gif#pic_center)可以看到线条的高度是充满了整个TabLayout的，如果我们需要让线条的高度短一点，则可以设置DividerPadding来实现。

```java
LinearLayout linearLayout = (LinearLayout) tabLayout.getChildAt(0);
linearLayout.setDividerPadding(10);
```
运行结果如下：
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200508170643694.gif#pic_center)好了，TabLayout的基本使用就这些了。文章中如有错误的地方，还望指出！