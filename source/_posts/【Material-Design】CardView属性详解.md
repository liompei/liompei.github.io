title: 【Material Design】CardView属性详解
author: liompei
date: 2017-11-22 09:26:06
tags:
---
CardView 卡片式布局,继承自FrameLayout,用户可以自定义圆角弧度和Z轴高度,阴影大小等

展示:一个简单的cardView

![cardView](\images\pasted-6.png)

caedView属性详解:

- app:cardBackgroundColor  &nbsp;&nbsp;&nbsp;&nbsp;背景颜色
- app:cardCornerRadius  &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;圆角弧度
- app:cardElevation  &nbsp;&nbsp;&nbsp;&nbsp;阴影大小
- app:cardMaxElevation  &nbsp;&nbsp;&nbsp;&nbsp;阴影的最大高度
- app:cardPreventCornerOverlap  &nbsp;&nbsp;&nbsp;&nbsp;为内容添加内边距防止与CardView角点重叠(为兼容5.0之前版本)
- app:cardUseCompatPadding  &nbsp;&nbsp;&nbsp;&nbsp;填充内容设置内边距(为5.0之后版本添加相同的填充值)
- app:contentPadding  &nbsp;&nbsp;&nbsp;&nbsp;设置内边距
- app:contentPaddingBottom
- app:contentPaddingLeft
- app:contentPaddingRight
- app:contentPaddingTop

使用
引入cardView包

Gradle
```
    implementation 'com.android.support:cardview-v7:27.0.1'
```
注: `implementation`为Android Studio新的依赖方式,代替了`compile`<br>区别请看https://zhuanlan.zhihu.com/p/27475883

```xml
    <android.support.v7.widget.CardView
        android:layout_width="220dp"
        android:layout_height="120dp"
        android:layout_centerInParent="true"
        app:cardBackgroundColor="@color/colorPrimary"
        app:cardCornerRadius="5dp"
        app:cardElevation="2dp"
        app:cardPreventCornerOverlap="true"
        app:cardUseCompatPadding="true">
        <TextView
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:layout_gravity="center"
            android:text="这是一个cardView"
            android:textColor="@color/white"/>
    </android.support.v7.widget.CardView>
```
很简单却能实现圆角的控件,如果想去除阴影可以设置`app:cardElevation="0dp"`