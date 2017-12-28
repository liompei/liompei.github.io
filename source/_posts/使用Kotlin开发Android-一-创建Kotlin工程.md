title: 使用Kotlin开发Android(一)创建Kotlin工程
author: liompei
tags:
  - Kotlin
categories:
  - android
date: 2017-12-28 09:46:00
---
Kotlin如今已经成为Android官方支持开发语言,接触了Kotlin看了几天之后发现,其语言简介,代码易读,成为开发Android开发的主流框架之一也是时间问题.开发Kotlin之前,可以在相关网站上多多学习.
<!--more-->
### 学习资料
[Kotlin中文语言站](https://www.kotlincn.net/docs/reference/android-overview.html)

看完之后就差不多了,如果不理解可以继续往下看,或者参考github上面的众多项目

### 创建应用
打开AS(3.0+),创建应用,注意:勾选`Include Kotlin Support`
其他步骤和创建一个普通Android应用一样

首先打开module的build.gradle,看看Kotlin相关的引用
```gradle
apply plugin: 'kotlin-android'

apply plugin: 'kotlin-android-extensions'

dependencies {
    implementation fileTree(dir: 'libs', include: ['*.jar'])
    implementation"org.jetbrains.kotlin:kotlin-stdlib-jre7:$kotlin_version"
}
```
再看看项目的build.gradle
```gradle
buildscript {
    ext.kotlin_version = '1.2.0'
    repositories {
        google()
        jcenter()
    }
    dependencies {
        classpath 'com.android.tools.build:gradle:3.0.1'
        classpath "org.jetbrains.kotlin:kotlin-gradle-plugin:$kotlin_version"

        // NOTE: Do not place your application dependencies here; they belong
        // in the individual module build.gradle files
    }
}
```
这里指定了引用kotlin的版本

如果这里需要添加java8的支持,按快捷键  Control+Alt+Shift+S

![upload successful](\images\pasted-15.png)

点击确定后会重新构建,等待完毕

### 分析代码

MainActivity.kt

```kotlin
class MainActivity : AppCompatActivity() {

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_main)
    }
}

```

对比原java代码

```java
public class MainActivity extends AppCompatActivity {

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main2);
    }
}
```

对比可见:

- class中`:`代替了extends 表示继承
- 方法中`savedInstanceState: Bundle?`的`:`表示类型,`savedInstanceState`的类型为Bundle,`?`表示可以为空

### 布局文件的引用

打开main_activity.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<android.support.constraint.ConstraintLayout
    xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    tools:context="com.liompei.kotlindemo.MainActivity">

    <TextView
        android:id="@+id/tv_test"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="Hello World!"
        app:layout_constraintBottom_toBottomOf="parent"
        app:layout_constraintLeft_toLeftOf="parent"
        app:layout_constraintRight_toRightOf="parent"
        app:layout_constraintTop_toTopOf="parent"/>

</android.support.constraint.ConstraintLayout>
```
这里我们TextView的id是tv_text

打开MainActivity.kt

直接添加

```kotlin
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_main)
        tv_test.text = "测试文字"
    }
```
屌爆了,原来的是

```java
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        TextView textView=findViewById(R.id.tv_test);
        textView.setText("测试文字");
    }
```
在Kotlin代码中省略了`findViewById`,省略了引号

### 点击事件

首先添加一个Button

```xml
    <Button
        android:id="@+id/btn_test"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:text="点击事件"/>
```
在activity中添加一个toast方法

```kotlin
    private fun toast(string: String) {
        Toast.makeText(this, string, Toast.LENGTH_SHORT).show()
    }
```

点击事件,打印`tv_test`的文字
```kotlin
 btn_test.setOnClickListener { toast(tv_test.text as String) }
```
或
```kotlin
 btn_test.setOnClickListener { toast("${tv_test.text}") }
```
可根据as的自动提示一键生成

//当需要参数View时,可使用lambda表达式

```kotlin
 btn_test.setOnClickListener { view -> onClick(view) }
```
或
```kotlin
 btn_test.setOnClickListener { onClick(it) }
```
#### 什么是it

单个参数的隐式名称

另一个有用的约定是，如果函数字面值只有一个参数， 那么它的声明可以省略（连同 ->），其名称是 it。

### when 代替 while

```kotlin
    private fun onClick(view: View) {
        when (view) {
            btn_test -> toast("aaaaaaaaaaaaaa")
            tv_test -> toast("sssssssss")
            else ->toast("其他")
        }
    }

```


### 总结


- 在类名中,`:`表示继承,而在方法的`()`中`:`表示定义类型,`()`后的`:`表示返回值类型

例如
```kotlin
    private fun toast(string: String): String {
        Toast.makeText(this, string, Toast.LENGTH_SHORT).show()
        return ""
    }
```

- Kotlin中的null安全,添加`?`后方可为null
- it:单个参数的隐式名称

基本上,读一遍[Kotlin中文语言站](https://www.kotlincn.net/docs/reference/android-overview.html)上的代码,就可以对Kotlin有基本的了解,并看懂别人的Kotlin代码,自己写写也不是难题

以上








































