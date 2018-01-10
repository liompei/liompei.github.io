title: 【ROOT】让Android应用申请root权限并运行shell命令
author: liompei
date: 2018-01-10 15:12:29
tags:
---
root权限即Android手机的最高权限

### 对root的理解

对于ROOT本身来说不会降低系统安全性，除非ROOT管理器本身又有漏洞。所以通过官方解锁的方式，使用安全的RECOVERY和安全的ROOT管理器，与官方的安全性一致。

### 开发前提

- 一部已经root过的android手机
- IDE开发环境

### 获取权限

新建类`RootUtils`,给它一个单例

```java
private volatile static RootUtils rootUtils;
    private static OutputStream outputStream;

    public static RootUtils getInstance() {
        if (rootUtils == null) {
            synchronized (RootUtils.class) {
                if (rootUtils == null) {
                    rootUtils = new RootUtils();
                }
            }
        }
        return rootUtils;
    }
```

在私有方法中申请权限

```java
    private RootUtils() {
        if (outputStream == null) {
            try {
                outputStream = Runtime.getRuntime().exec("su").getOutputStream();
            }catch (IOException e){
                e.printStackTrace();
                Zx.d("初始化outputStream失败");
            }
        }
    }
```

在MainActivity或需要获取root权限的地方,加上代码

```
        //获取root权限
        RootUtils.getInstance();
```

如此就可以申请root权限了

![upload successful](\images\pasted-26.png)

### 执行命令

```java
    public void exec(String shell){
        try {
            outputStream.write((shell+ "\n").getBytes());  //输入命令,敲下换行符
            outputStream.flush();  //输出
            Zx.d("输出成功");
        } catch (IOException e) {
            e.printStackTrace();
            Zx.e("输出失败");
        }
    }
```

在MainActivity中使用,这里我们模拟使用微信跳一跳的功能

```java
    RootUtils.getInstance().exec("input swipe 200 200 300 300 500");
```

`input swipe 200 200 300 300 500`表示,滑动屏幕,从坐标(200,200)滑动到(300,300),滑动耗时500ms


### 完整代码

```java
/**
 * Created by Liompei
 * time : 2018/1/4 15:18
 * 1137694912@qq.com
 * https://github.com/liompei
 * remark:root工具类
 */

public class RootUtils {

    private volatile static RootUtils rootUtils;
    private static OutputStream outputStream;

    public static RootUtils getInstance() {
        if (rootUtils == null) {
            synchronized (RootUtils.class) {
                if (rootUtils == null) {
                    rootUtils = new RootUtils();
                }
            }
        }
        return rootUtils;
    }
    
    private RootUtils() {
        if (outputStream == null) {
            try {
                outputStream = Runtime.getRuntime().exec("su").getOutputStream();
            }catch (IOException e){
                e.printStackTrace();
                Zx.d("初始化outputStream失败");
            }
        }
    }

    public void exec(String shell){
        try {
            outputStream.write((shell+ "\n").getBytes());  //输入命令,敲下换行符
            outputStream.flush();  //输出
            Zx.d("输出成功");
        } catch (IOException e) {
            e.printStackTrace();
            Zx.e("输出失败");
        }
    }
    
}
```








