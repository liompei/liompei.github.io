title: EventBus3.1完全解析(二)【处理线程】
author: liompei
tags:
  - EventBus
categories:
  - android
date: 2018-02-28 11:09:00
---
EventBus可以为您处理线程：事件可以发布在与发布线程不同的线程中。一个常见的用例是处理UI更改。在Android中，UI更改必须在UI（主）线程中完成。另一方面，网络或任何耗时的任务不得在主线程上运行。EventBus可以帮助您处理这些任务并与UI线程同步（无需深入研究线程转换，使用AsyncTask等）。

EventBus有五种ThreadMode

- ThreadMode: POSTING
- ThreadMode: MAIN
- ThreadMode: MAIN_ORDERED
- ThreadMode: BACKGROUND
- ThreadMode: ASYNC

查看接口`@Subscribe`的源码
```java
    ThreadMode threadMode() default ThreadMode.POSTING;
```
表示事件处理的默认线程为`POSTING`


## ThreadMode: POSTING

和事件发送在同一线程

订阅者将在发布该事件的同一主题中被调用。这是默认设置。事件传递是同步完成的，所有订阅者在发布完成后都会被调用。这个ThreadMode意味着最小的开销，因为它避免了完全的线程切换。因此，对于已知完成的简单任务而言，这是推荐的模式，只需很短的时间而不需要主线程。使用此模式的事件处理程序应该快速返回以避免阻塞发布线程，该线程可能是主线程。

```java
// Called in the same thread (default)
// ThreadMode is optional here
@Subscribe(threadMode = ThreadMode.POSTING)
public void onMessage(MessageEvent event) {
    log(event.message);
}
```


## ThreadMode: MAIN

在主线程(UI线程)

订阅者将在Android的主线程（有时称为UI线程）中被调用。如果发布线程是主线程，则会直接调用事件处理程序方法（与ThreadMode.POSTING中描述的同步）。使用此模式的事件处理程序必须快速返回以避免阻塞主线程。

```java
// Called in Android UI's main thread
@Subscribe(threadMode = ThreadMode.MAIN)
public void onMessage(MessageEvent event) {
    textField.setText(event.message);
}
```


## ThreadMode: MAIN_ORDERED

队列主线程

订阅者将在Android的主线程中被调用。该事件总是排队等待以后发送给订阅者，因此发帖的调用将立即返回。这为事件处理提供了更严格和更一致的顺序（因此名称MAIN_ORDERED）。例如，如果使用主线程模式在事件处理程序中发布另一个事件，则第二个事件处理程序将在第一个事件处理程序在第一个事件处理程序之前完成（因为它是同步调用的 - 将其与方法调用进行比较）使用MAIN_ORDERED，第一个事件处理程序将完成，然后第二个事件处理程序将在稍后的时间点（只要主线程有能力）被调用。
使用此模式的事件处理程序必须快速返回以避免阻塞主线程。

```java
// Called in Android UI's main thread
@Subscribe(threadMode = ThreadMode.MAIN_ORDERED)
public void onMessage(MessageEvent event) {
    textField.setText(event.message);
}
```


## ThreadMode: BACKGROUND

发布线程不是主线程->当前线程<br>
发布线程是主线程->新线程

订阅者将在后台线程中调用。如果发布线程不是主线程，则会在发布线程中直接调用事件处理程序方法。如果发布线程是主线程，则EventBus使用单个后台线程来按顺序发送所有事件。使用此模式的事件处理程序应尽快返回以避免阻塞后台线程。

```java
// Called in the background thread
@Subscribe(threadMode = ThreadMode.BACKGROUND)
public void onMessage(MessageEvent event){
    saveToDisk(event.message);
}
```


## ThreadMode: ASYNC

事件处理全部在单独线程中调用

事件处理程序方法在单独的线程中调用。这总是独立于发布线程和主线程。发布事件永远不会等待使用此模式的事件处理程序方法。事件处理程序方法应该使用这种模式，如果它们的执行可能需要一些时间，例如网络访问。避免同时触发大量长时间运行的异步处理程序方法来限制并发线程的数量。EventBus使用线程池有效地重用已完成异步事件处理程序通知中的线程。

```java
// Called in a separate thread
@Subscribe(threadMode = ThreadMode.ASYNC)
public void onMessage(MessageEvent event){
    backend.send(event.message);
}
```