title: 项目Base类搭建
author: liompei
tags:
  - BaseActivity
  - BaseFragment
  - Toolbar
categories:
  - Android
date: 2017-11-12 19:10:00
---
### 前言
一个项目最基础的当然就是Application,BaseActivity,BaseFragment的搭建了,下面是我的写法,当做参考未尝不可,如果有更优的解决方案请多指教
<!-- more -->
### Application
---
App.class
``` java
public class App extends Application {

    //***全局***
    public static App instance;

    public static App getInstance() {
        return instance;
    }

    private List<Activity> activityList = new ArrayList<>();

    // 添加Activity到容器中
    public void addActivity(Activity activity) {
        activityList.add(activity);
    }

    public void deleteActivity(Activity activity) {
        activityList.remove(activity);
    }

    public void finishAllActivity() {
        for (Activity activity : activityList) {
            activity.finish();
        }
        activityList.clear();
    }

    // 退出
    public void exit() {
        for (Activity activity : activityList) {
            activity.finish();
        }
        activityList.clear();
        System.exit(0);
    }
    //***end***

    @Override
    public void onCreate() {
        super.onCreate();
        instance = this;
        //do something
    }
}
```
其中,<font color=blue>activityList</font>将每次打开的activity添加到list中,方便进行统一处理(如退出),当然在BaseActivity中进行配置后才会生效,下面看看BaseActivity
### BaseActivity
---
```java
public abstract class BaseActivity extends AppCompatActivity {

    protected BaseActivity mBaseActivity;

    @Override
    protected void onCreate(@Nullable Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        App.getInstance().addActivity(this);
        mBaseActivity=this;
        setContentView(getLayoutId());
        initViews(savedInstanceState);
        initData();
        onEvent();
    }

    public abstract int getLayoutId();

    public abstract void initViews(Bundle savedInstanceState);

    public abstract void initData();

    public abstract void onEvent();

    protected Toolbar getToolbar(CharSequence title, boolean showHomeAsUp) {
        Toolbar toolbar = findViewById(R.id.toolbar);
        toolbar.setTitle(title);
        setSupportActionBar(toolbar);
        getSupportActionBar().setDisplayHomeAsUpEnabled(showHomeAsUp);
        return toolbar;
    }

    @Override
    public boolean onOptionsItemSelected(MenuItem item) {
        switch (item.getItemId()) {
            case android.R.id.home:
                onBackPressed();
                break;
        }
        return super.onOptionsItemSelected(item);
    }

    @Override
    protected void onDestroy() {
        App.getInstance().deleteActivity(this);
        super.onDestroy();
    }

    @Override
    public void finish() {
        super.finish();
    }

    protected void toast(final String s) {
        runOnUiThread(new Runnable() {
            @Override
            public void run() {
                Toast.makeText(App.getInstance(), s, Toast.LENGTH_SHORT).show();
            }
        });
    }
}
```
在Activity onCreate时调用:  <font color=blue>App.getInstance().addActivity(this);</font>
onDestroy时移除Activity: <font color=blue>App.getInstance().deleteActivity(this);</font>
- getLayoutId()&nbsp;&nbsp;传入Activity的布局文件
- initViews(Bundle savedInstanceState)&nbsp;&nbsp;初始化view控件,如findViewById
- initData()&nbsp;&nbsp;初始化数据
- onEvent()&nbsp;&nbsp;进行事件
- toast()&nbsp;&nbsp;用于toast输出
- getToolbar(CharSequence title, boolean showHomeAsUp)  创建toolBar并返回一个toolbar对象

传入的值分别为:

toolbar的title,是否显示返回按钮,返回按钮由<font color=blue>getSupportActionBar().setDisplayHomeAsUpEnabled(showHomeAsUp);</font>设定,并在<font color=blue>onOptionsItemSelected</font>中添加了点击事件
#### Toolbar
使用Toolbar请先删除ActionBar,在styles.xml的AppTheme中配置
```xml
    <!-- Base application theme. -->
    <style name="AppTheme" parent="Theme.AppCompat.Light.DarkActionBar">
        <!-- Customize your theme here. -->
        <item name="colorPrimary">@color/colorPrimary</item>
        <item name="colorPrimaryDark">@color/colorPrimaryDark</item>
        <item name="colorAccent">@color/colorAccent</item>

        <!--no actionbar and title-->
        <item name="windowActionBar">false</item>
        <item name="windowNoTitle">true</item>
    </style>
```
getToolbar需要在layout中创建一个包含toolbar的布局文件,例如
abc_toolbar.xml
```xml
<android.support.v7.widget.Toolbar
    xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    android:id="@+id/toolbar"
    android:layout_width="match_parent"
    android:layout_height="wrap_content"
    android:background="@color/colorPrimary"
    android:theme="@style/ToolbarMenu"
    app:titleTextAppearance="@style/ToolbarTitleText"/>
```
- background:&nbsp;&nbsp;toolbar的背景色
- titleTextAppearance:&nbsp;&nbsp;toolbar字体颜色,大小,(加粗,斜体等)
- theme:&nbsp;&nbsp;toolbar上menu的样式
style如下
```xml
    <style name="ToolbarMenu" parent="ThemeOverlay.AppCompat.Dark.ActionBar">

        <!--设置Menu菜单的背景色-->
        <item name="android:colorBackground">@color/white</item>
        <!--设置Menu菜单字体颜色-->
        <item name="android:textColor">@color/black</item>
        <!--字体大小-->
        <item name="android:textSize">16sp</item>
        <!--设置Menu窗口不覆盖Toolbar视图-->
        <item name="actionOverflowMenuStyle">@style/OverflowMenuStyle</item>
        <!--Menu样式-->
        <!--<item name="android:actionOverflowButtonStyle">@style/ToolbarMenuButtonOverflow</item>-->
    </style>

    <!--把该属性改为false即可使menu位置位于toolbar之下-->
    <style name="OverflowMenuStyle" parent="Widget.AppCompat.Light.PopupMenu.Overflow">
        <item name="overlapAnchor">false</item>
    </style>

    <!--toolbar文字大小,颜色-->
    <style name="ToolbarTitleText" parent="TextAppearance.Widget.AppCompat.Toolbar.Title">
        <item name="android:textSize">20sp</item>
        <item name="android:textColor">@color/white</item>
        <item name="android:textStyle">normal</item>
    </style>

    <!--menu样式-->
    <!--<style name="ToolbarMenuButtonOverflow" parent="android:style/Widget.Holo.ActionButton.Overflow">-->
    <!--<item name="android:src">@drawable/ic_add_white_24dp</item>-->
    <!--</style>-->
```
### BaseFragment
```
public abstract class BaseFragment extends RxFragment {

    private View parentView;
    private boolean isCreated;
    protected boolean isVisible;
    protected boolean isLazyed;

    @Override
    public void onCreate(@Nullable Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setHasOptionsMenu(true);
    }

    @Nullable
    @Override
    public View onCreateView(LayoutInflater inflater, @Nullable ViewGroup container, @Nullable Bundle savedInstanceState) {
        parentView = inflater.inflate(getLayoutResId(), container, false);
        initView(savedInstanceState);
        initData();
        return parentView;
    }

    @Override
    public void onViewCreated(View view, @Nullable Bundle savedInstanceState) {
        super.onViewCreated(view, savedInstanceState);
        onEvent();
        isCreated=true;
        onVisible();
    }

    @Override
    public void setUserVisibleHint(boolean isVisibleToUser) {
        super.setUserVisibleHint(isVisibleToUser);
        if (getUserVisibleHint()) {
            isVisible = true;
            onVisible();
        } else {
            isVisible = false;
            onInvisible();
        }
    }

    protected void onVisible() {
        if (isVisible&&!isLazyed&&isCreated){
            isLazyed = true;
            lazyLoad();
        }
    }

    protected void lazyLoad() {
    }

    protected void onInvisible() {
    }

    @Override
    public void onDestroy() {
        super.onDestroy();
    }

    protected BaseActivity getBaseActivity() {
        return (BaseActivity) getActivity();
    }

    public abstract
    @LayoutRes
    int getLayoutResId();

    protected abstract void initView(@Nullable Bundle savedInstanceState);

    protected abstract void initData();

    protected abstract void onEvent();

    protected <T extends View> T findViewById(int resId) {
        return parentView.findViewById(resId);
    }

    protected Toolbar getToolbar(CharSequence title, boolean showHomeAsUp) {
        Toolbar toolbar = parentView.findViewById(R.id.toolbar);
        toolbar.setTitle(title);
        getBaseActivity().setSupportActionBar(toolbar);
        getBaseActivity().getSupportActionBar().setDisplayHomeAsUpEnabled(showHomeAsUp);
        return toolbar;
    }

    @Override
    public boolean onOptionsItemSelected(MenuItem item) {
        switch (item.getItemId()) {
            case android.R.id.home:
                getBaseActivity().onBackPressed();
                break;
        }
        return super.onOptionsItemSelected(item);
    }

    protected void toast(final String s) {
        getBaseActivity().toast(s);
    }
}
```
和BaseActivity同理,其中lazyLoad();是fragment的懒加载</br>
setHasOptionsMenu(true);解决了在fragment中创建Toolbar不显示的问题
### 补充
#### findViewById
在Android Studio 3.0中
```
buildToolsVersion '27.0.1'
```
findViewById方法返回对象由原来的View变成了T(即`<T extends View>`)
在代码中使用findViewById时就不需要强转成View的子对象了
如果你的buildToolsVersion过低,可以在BaseActivity中加入下面的代码提高写作速度
```java
    protected <T extends View> T findView(int resId) {
        return (T) (findViewById(resId));
    }
```
最好是早日更新AS,打开SDK Manager那个下载的图标,勾选需要更新的内容后后点击Apply,最后在build.gradle中设置即可