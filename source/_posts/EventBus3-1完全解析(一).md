title: EventBus3.1完全解析(一)【初步使用】
author: liompei
tags:
  - EventBus
categories:
  - android
date: 2018-02-27 17:11:00
---
EventBus是Android和Java的发布/订阅事件总线。

https://github.com/greenrobot/EventBus

使用EventBus的优势是:


+ 简化了组件之间的通信
+ 将事件发送者和接收者分开
+ 活动，片段和后台线程表现良好
+ 避免了复杂且容易出错的依赖性和生命周期问题
+ 使您的代码更简单
+ 速度很快
+ 很小（约50k）
+ 通过100,000,000+安装的应用程序在实践中证明了这一点
+ 具有高级功能，如传送线程，用户优先级等。


##### 添加依赖

```gradle
implementation 'org.greenrobot:eventbus:3.1.1'
```

##### 定义事件

新建一个类

```java
public class MessageEvent {

    public final String message;

    public MessageEvent(String message) {
        this.message = message;
    }
}
```

##### 处理事件

实现处理方法,当事件被发布时调用,使用`@Subscribe`注释定义

```java
// This method will be called when a MessageEvent is posted (in the UI thread for Toast)
@Subscribe(threadMode = ThreadMode.MAIN)
public void onMessageEvent(MessageEvent event) {
    Toast.makeText(getActivity(), event.message, Toast.LENGTH_SHORT).show();
}
 
// This method will be called when a SomeOtherEvent is posted
@Subscribe
public void handleSomethingElse(SomeOtherEvent event) {
    doSomethingWith(event);
}
```


用户需要注册和注销事件总线,只有注册后,才可以收到活动,多数情况下,我会在Activity的`onCreate`方法中注册,在`onDestroy`中注销

```java
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        //注册
        EventBus.getDefault().register(this);
    }
    
    @Override
    protected void onDestroy() {
        super.onDestroy();
        //注销
        EventBus.getDefault().unregister(this);
    }
```


##### 发布活动

可以从任何部分发布事件.所有当前注册的匹配事件类型的订阅者都会收到它

```java
EventBus.getDefault().post(new MessageEvent("Hello everyone!"));
```

----

EventBus最简单的使用方法















