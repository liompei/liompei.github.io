title: Android常用Theme属性分析
author: liompei
tags:
  - Theme
categories:
  - android
date: 2018-02-26 14:24:00
---

主题决定了App的展示效果，我们可以为整个Application指定主题，也可以为Activity单独指定主题，甚至可以为各个控件如Button、TextView指定主题。本文对Android Theme的引用方式，版本分类进行列举，并对兼容包主题进行了归类、总结。期望达到对Theme有一个总体的清晰的认识。


打开style,一般都会有一个默认的Theme
```xml
    <!-- Base application theme. -->
    <style name="AppTheme" parent="Theme.AppCompat.Light.DarkActionBar">
        <!-- Customize your theme here. -->
        <item name="colorPrimary">@color/colorPrimary</item>
        <item name="colorPrimaryDark">@color/colorPrimaryDark</item>
        <item name="colorAccent">@color/colorAccent</item>
    </style>
```

parent指定继承的父类,没有设置的参数将直接使用父类的参数,例如`Theme.AppCompat.Light.DarkActionBar`

Aplication Theme常用设置如下

```xml
    <!-- Base application theme. -->
    <style name="AppTheme" parent="Theme.AppCompat.Light.DarkActionBar">
        <!-- Customize your theme here. -->
        <item name="colorPrimary">@color/md_green</item>
        <item name="colorPrimaryDark">@color/md_green</item>
        <item name="colorAccent">@color/colorAccent</item>

        <!--no actionbar and title-->
        <item name="windowActionBar">false</item>
        <item name="windowNoTitle">true</item>
        <!--other-->
        <item name="android:textColor">@color/text_color</item>
        <!--<item name="android:textColorHint">@color/text_color_hint</item>-->
        <item name="android:windowBackground">@color/window_background</item>
        <item name="android:textSize">14sp</item>
    </style>

    <style name="FullScreenTheme" parent="AppTheme">
        <item name="android:windowFullscreen">true</item>
    </style>

    <style name="SplashTheme" parent="FullScreenTheme">
        <item name="android:windowFullscreen">true</item>
        <item name="android:windowBackground">@color/transparent</item>
    </style>
```

#### 设置toolbar的theme
layout_toolbar.xml 

```xml
<?xml version="1.0" encoding="utf-8"?>
<android.support.v7.widget.Toolbar
    xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    android:id="@+id/toolbar"
    android:layout_width="match_parent"
    android:layout_height="48dp"
    android:background="@color/colorPrimary"
    android:theme="@style/ToolbarMenu"
    app:titleTextAppearance="@style/ToolbarTitleText">

    <TextView
        android:id="@+id/toolbar_title"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_gravity="center"
        android:gravity="center"
        android:text=""
        android:textColor="@color/white"
        android:textSize="16sp"/>


</android.support.v7.widget.Toolbar>
```

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
        <item name="android:textSize">16sp</item>
        <item name="android:textColor">@color/white</item>
        <item name="android:textStyle">normal</item>
    </style>

    <!--menu样式-->
    <!--<style name="ToolbarMenuButtonOverflow" parent="android:style/Widget.Holo.ActionButton.Overflow">-->
    <!--<item name="android:src">@drawable/ic_add_white_24dp</item>-->
    <!--</style>-->

    <!--toolbar文字大小,颜色-->
    <style name="BottomTabText" parent="TextAppearance.Widget.AppCompat.Toolbar.Title">
        <item name="android:textSize">12sp</item>
        <item name="android:textColor">@color/white</item>
        <item name="android:textStyle">normal</item>
    </style>
```

#### 常用的Material Design色彩
```xml
    <color name="text_color">#212121</color>
    <color name="secondary_text">#757575</color>
    <!--<color name="text_color_hint">#FFFFFF</color>-->
    <color name="window_background">#F1F2F3</color>
    <color name="divider">#BDBDBD</color>
    <color name="item_touch">#eaeaea</color>

    <color name="md_red">#F44336</color>
    <color name="md_pink">#E91E63</color>
    <color name="md_purple">#9C27B0</color>
    <color name="md_deep_purple">#673AB7</color>
    <color name="md_indigo">#3F51B5</color>
    <color name="md_blue">#2196F3</color>
    <color name="md_light_blue">#03A9F4</color>
    <color name="md_cyan">#00BCD4</color>
    <color name="md_teal">#009688</color>
    <color name="md_green">#4CAF50</color>
    <color name="md_light_green">#8BC34A</color>
    <color name="md_lime">#CDDC39</color>
    <color name="md_yellow">#FFEB3B</color>
    <color name="md_amber">#FFC107</color>
    <color name="md_orange">#FF9800</color>
    <color name="md_deep_orange">#FF5722</color>
    <color name="md_brown">#795548</color>
    <color name="md_grey">#9E9E9E</color>
    <color name="md_blue_grey">#607D8B</color>
    <color name="white">#FFFFFF</color>
    <color name="black">#000000</color>
    <color name="transparent">#00000000</color>
```

