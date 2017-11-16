title: 【 json解析】之Gson
author: liompei
tags:
  - json
  - gson
categories:
  - Android
date: 2017-11-16 16:32:00
---
Gson是谷歌出品的优秀json解析框架
Gson是一个Java库，可用于将Java对象转换为JSON表示。它也可以用来将JSON字符串转换为等效的Java对象。Gson可以使用任意的Java对象，包括你没有源代码的预先存在的对象。
<!--more-->

### 概述

https://github.com/google/gson

#### Gson目标
- 提供简单的toJson()和fromJson()方法的Java对象转换为JSON，反之亦然
- 允许将现有的不可修改的对象转换为JSON或从JSON转换
- 广泛的Java泛型支持
- 允许对象的自定义表示
- 支持任意复杂的对象（具有深度继承层次结构和广泛的泛型使用）

#### 在Android中使用Gson
打开需要使用gson解析的module
```
dependencies {
    compile 'com.google.code.gson:gson:2.8.2'
}
```
//最新版本请在 https://github.com/google/gson 查看

官方的用户指南在此,https://github.com/google/gson/blob/master/UserGuide.md 英语不好当我没说

### Gson实战
#### Gson解析一个对象
```json
{
    "username":"liompei",
    "sex":"男",
    "age":18,
    "level":100,
    "isVip":true
}
```
进行解析<br>
新建实体类MyUser.class
```java
/**
 * Created by Liompei
 * time : 2017/11/16 15:42
 * 1137694912@qq.com
 * https://github.com/liompei
 * remark:
 */

public class MyUser {
    
    private String username;
    private String sex;
    private int age;
    private int level;
    private boolean isVip;

    public String getUsername() {
        return username;
    }

    public void setUsername(String username) {
        this.username = username;
    }

    public String getSex() {
        return sex;
    }

    public void setSex(String sex) {
        this.sex = sex;
    }

    public int getAge() {
        return age;
    }

    public void setAge(int age) {
        this.age = age;
    }

    public int getLevel() {
        return level;
    }

    public void setLevel(int level) {
        this.level = level;
    }

    public boolean isIsVip() {
        return isVip;
    }

    public void setIsVip(boolean isVip) {
        this.isVip = isVip;
    }
}
```
解析代码
```java
    private void parseJson(String jsonString) {
        Gson gson=new Gson();
        MyUser myUser=gson.fromJson(jsonString,MyUser.class);
        System.out.println(myUser.getUsername());
    }
```
简直太爽了有木有,一行代码搞定
#### Gson解析一个数组
比如这里有一堆的用户
```json
[
    {
        "username":"liompei1",
        "sex":"男",
        "age":18,
        "level":100,
        "isVip":true
    },
    {
        "username":"liompei2",
        "sex":"男",
        "age":19,
        "level":100,
        "isVip":true
    },
    {
        "username":"liompei3",
        "sex":"男",
        "age":20,
        "level":100,
        "isVip":true
    },
    {
        "username":"liompei4",
        "sex":"男",
        "age":18,
        "level":100,
        "isVip":true
    }
]
```
解析
```java
    private void parseJson(String jsonString) {
        Type type = new TypeToken<LinkedList<MyUser>>() {
        }.getType();  //TypeToken是一个抽象类 , Type是java.lang.reflect.Type
        Gson gson = new Gson();
        //json数据存在type中,注:jsonString必须为集合
        LinkedList<MyUser> users = gson.fromJson(jsonString, type);
        for (Iterator<MyUser> iterator = users.iterator(); iterator.hasNext(); ) {
            MyUser myUser = iterator.next();
            System.out.println(myUser.getUsername());
        }
    }
```
注: 这里最后的LinkList是用迭代器Iterator拿到每一条数据
### Gson进阶学习

#### @SerializedName注解的使用
@SerializedName 注解是属性重命名的方法,例如在MyUser.class中加上
```java
    @SerializedName("username")
    private String username;
```
如此就可以修改属性名,把下面`private String username`中的`username`修改成任意你喜欢的名字<br>
这种情况可以用在后台命名混乱又不愿意改的情况下

想起来再补充

以上




