title: 【Xposed】Android-Hook初探
author: liompei
tags:
  - hook
categories:
  - android
date: 2018-01-02 17:06:00
---
### 什么是Xposed?

Android牛X开发者[rovo89](https://github.com/rovo89)开发的框架

Xposed框架是一款可以在不修改APK的情况下影响程序运行(修改系统)的框架服务，基于它可以制作出许多功能强大的模块，且在功能不冲突的情况下同时运作。Xposed理论上能够hook到系统任意一个Java进程，由于是从底层hook，所以需要root权限，并且每次更新都要重新启动

### Xposed能实现那些功能?

手机系统信息修改,例如改变状态栏,imei,推送通知,去广告.微信QQ防撤回,自动领红包,修改app功能...


### 开发前提

一部已安装Xposed框架的手机

![upload successful](\images\pasted-16.png)


### 开发Xposed项目

#### 1.像往常一样,新建一个app项目
#### 2.导入jar包

这里我使用 Gradle 方式导入
```gradle
    provided 'de.robv.android.xposed:api:82'
```
注意:是`provided`

当然你也可以下载jar包导入,但必须是`provided`方式

它的仓库地址 https://bintray.com/rovo89/de.robv.android.xposed/api 我比较喜欢用最新版

#### 3.打开Manifast.xml
添加代码在Application中

```xml
        <!-- 标记xposed插件 start-->
        <meta-data
            android:name="xposedmodule"
            android:value="true"/>
        <!-- 模块描述 -->
        <meta-data
            android:name="xposeddescription"
            android:value="测试Xposed"/>
        <!-- 最低版本号 -->
        <meta-data
            android:name="xposedminversion"
            android:value="54"/>
        <!-- 标记xposed插件 end-->
```


- 新建类HelloHook.class
- 在app右键,新建资源文件夹,如下

![upload successful](\images\pasted-17.png)
在资源文件夹下新建 `xposed_init` 文件  注意:名字不能变

然后写入HelloHook路径:.例如我的:


![upload successful](\images\pasted-18.png)
 
我们再打开`HelloHook`,让它继承自`IXposedHookLoadPackage`

```java
public class HelloHook implements IXposedHookLoadPackage{
    @Override
    public void handleLoadPackage(XC_LoadPackage.LoadPackageParam lpparam) throws Throwable {

    }
}
```
此时,一个Hook应用就新建成功了,为了方便我们查看,可以加入打印

```java
public class HelloHook implements IXposedHookLoadPackage {
    @Override
    public void handleLoadPackage(XC_LoadPackage.LoadPackageParam lpparam) throws Throwable {
        XposedBridge.log("===包名===" + lpparam.packageName);
        Log.d("HelloHook", "===包名===" + lpparam.packageName);
    }
}
```
这里的`XposedBridge.log`是xposed的打印,可以在xposed框架的日志中查看,log打印是我加的打印,方便在as中查看(注意logcat要选中No Filters)

#### 一个重要的坑

 需要对软件进行正式签名,否则不可用
 
所以就打个正式包吧,有了签名文件,我们设置一下安装即是正式打包apk即可,如下


快捷键Control+Alt+Shift+S,选中module

添加一个Signing


![upload successful](\images\pasted-19.png)

将Build Types中的debug和release 的Signing Config选中新建的config


![upload successful](\images\pasted-20.png)


此时项目会重新构建,等待构建完成

在这里可以切换当前编译版本

![upload successful](\images\pasted-21.png)


接下来直接Run到手机里即可~~~如果安装成功,就可以收到xposed提示勾选的通知,(也可以在软件中打开查看)

打开Xposed框架,勾选

#### 重启手机


- 重启之后,看一下Xposed中的日志或者as打印是否成功


![upload successful](\images\pasted-22.png)

这样说明我们成功了

### 实现第三方Hook

打开项目的activity_main.xml,给默认的`textView`添加一个id
```xml
<?xml version="1.0" encoding="utf-8"?>
<android.support.constraint.ConstraintLayout
    xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    tools:context="com.liompei.xposeddemo.MainActivity">

    <TextView
        android:id="@+id/tv_test"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text=""
        app:layout_constraintBottom_toBottomOf="parent"
        app:layout_constraintLeft_toLeftOf="parent"
        app:layout_constraintRight_toRightOf="parent"
        app:layout_constraintTop_toTopOf="parent"/>

</android.support.constraint.ConstraintLayout>
```
我们的目标,就是修改这个TextView的值

这里我们将此项目当做第三方项目,通过hook修改这里的值

在MainActivity中,给此TextView设值
```java
public class MainActivity extends AppCompatActivity {

    private TextView tvTest;

    @Override
    protected void onCreate(@Nullable Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        tvTest = findViewById(R.id.tv_test);
        tvTest.setText("准备修改的值");
    }
}
```
此时TextView的值已经是`准备修改的值`了,run一下看看

![upload successful](\images\pasted-23.png)

在HelloHook.java中写入代码
```java
public class HelloHook implements IXposedHookLoadPackage {
    @Override
    public void handleLoadPackage(XC_LoadPackage.LoadPackageParam lpparam) throws Throwable {
//        XposedBridge.log("===包名===" + lpparam.packageName);
//        Log.d("HelloHook", "===包名===" + lpparam.packageName);
        if (lpparam.packageName.equals("com.liompei.xposeddemo")) {
            XposedHelpers.findAndHookMethod("com.liompei.xposeddemo.MainActivity", lpparam.classLoader, "onCreate", Bundle.class, new XC_MethodHook() {
                @Override
                protected void beforeHookedMethod(MethodHookParam param) throws Throwable {
                    Log.d("HelloHook", "beforeHookedMethod");
                }

                @Override
                protected void afterHookedMethod(MethodHookParam param) throws Throwable {
                    Log.d("HelloHook", "afterHookedMethod");
                    Class aClass = param.thisObject.getClass();
                    Field[] fields = aClass.getDeclaredFields();

                    for (int i = 0; i < fields.length; i++) {
                        Field field = fields[i];
                        Log.d("HelloHook", "参数名: " + field.getName() + ",类型: " + field.getType());
                    }
                }
            });
        }
    }
}
```

这里假设我们不知道此软件的TextView参数,包名为`com.liompei.xposeddemo`,当前类为`MainActivity`,
这里我们循环打印一下onCreate中的参数,Terminal中执行`adb reboot`重启手机

打开XposedDemo,可以接收到打印


![upload successful](\images\pasted-24.png)

此时我们知道了参数名为tvTest,类型是TextView,利用反射就可以进行操作了

```java

public class HelloHook implements IXposedHookLoadPackage {
    @Override
    public void handleLoadPackage(XC_LoadPackage.LoadPackageParam lpparam) throws Throwable {
//        XposedBridge.log("===包名===" + lpparam.packageName);
//        Log.d("HelloHook", "===包名===" + lpparam.packageName);
        if (lpparam.packageName.equals("com.liompei.xposeddemo")) {
            XposedHelpers.findAndHookMethod("com.liompei.xposeddemo.MainActivity", lpparam.classLoader, "onCreate", Bundle.class, new XC_MethodHook() {
                @Override
                protected void beforeHookedMethod(MethodHookParam param) throws Throwable {
                    Log.d("HelloHook", "beforeHookedMethod");
                }

                @Override
                protected void afterHookedMethod(MethodHookParam param) throws Throwable {
                    Log.d("HelloHook", "afterHookedMethod");
                    Class aClass = param.thisObject.getClass();
                    Field[] fields = aClass.getDeclaredFields();

                    for (int i = 0; i < fields.length; i++) {
                        Field field = fields[i];
                        Log.d("HelloHook", "参数名: " + field.getName() + ",类型: " + field.getType());
                    }

                    Field field = aClass.getDeclaredField("tvTest");
                    field.setAccessible(true);
                    TextView textView = (TextView) field.get(param.thisObject);
                    textView.setText("成功修改参数");

                }
            });
        }
    }
}
```

adb reboot

重启后打开应用


![upload successful](\images\pasted-25.png)

修改成功!


### 总结

步骤:
- 1.新建项目
- 2.引入xposed
- 3.manifast.xml添加xposed相关引用
- 4.新建class文件,作为hook类
- 5.assets下新建`xposed_init`,加载hook类
- 6.设置编译为正式签名
- 7.编写hook类,实现相关hook操作
- 8.安装到手机,并在xposed中打钩,重启手机

### 留下的坑

hook的一些接口

- IXposedHookLoadPackage
- IXposedHookInitPackageResources
- IXposedHookZygoteInit
- XposedHelpers

- 如何分析第三方应用

- 遇到微信这种参数名和方法都有混淆的如何改参

- 资源钩子的使用

- 抓包

- 反编译


`有空填坑....`


