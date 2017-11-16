title: 【 json解析】之JSONObject和JSONArray
author: liompei
date: 2017-11-16 14:34:39
tags:
---
JSON的全称是JavaScript Object Notation,即JS 对象标记,它是一种轻量级的数据交换格式
### 简介
在 JS 语言中，一切都是对象。因此，任何支持的类型都可以通过 JSON 来表示
JSON的特点有:
- 对象表示为键值对
- 数据由逗号分隔
- 花括号保存对象
- 方括号保存数组

举个例子?</br>
{"say":"hello world"}
其中 `say`是键,`helloo world`是值</br>
键和值都用双信号包裹,使用冒号分割,键在前,值在后</br>
下面通过一些实例讲解如何使用自带`org.json.JSONObject`解析json对象

**JSON在线解析及格式化验证的网站是 [json.cn](http://json.cn)**


### 案例:

#### 解析一个简单的数据

```json
{
    "key1":"hello",
    "key2":20,
    "key3":true
}
```
解析
```java
private void parseJson(String jsonString){
        try {
             //把字符串转成 JSONObject 对象
            JSONObject jsonObject = new JSONObject(jsonString);
            //拿到参数
            String key1 = jsonObject.getString("key1");
            int key2 = jsonObject.getInt("key2");
            boolean key3 = jsonObject.getBoolean("key3");
            System.out.println("key1的值: "+key1);
            System.out.println("key2的值: "+key2);
            System.out.println("key3的值: "+key3);
        } catch (JSONException e) {
            e.printStackTrace();
        }
    }
```
#### 解析一个登录返回数据

有一个简单的登录请求,返回的数据格式为
```json
{
    "code":"200",
    "message":"登录成功",
    "data":{
        "name":"liompei",
        "sex":"男",
        "age":"18"
    }
}
```
code 是状态码,200说明返回成功(当然状态码可以自己定义,本文只讨论json解析)
message是消息,dota内包含了登录后返回的参数
```java
    private void parseJson(String jsonString) {
        try {
            //把字符串转成 JSONObject 对象
            JSONObject jsonObject = new JSONObject(jsonString);
            //拿到参数
            int code = jsonObject.getInt("code");
            String message = jsonObject.getString("message");
            String data = jsonObject.getString("data");
            System.out.println("code: " + code + ",message: " + message + ",data: " + data);
        } catch (JSONException e) {
            e.printStackTrace();
        }
    }
```
解析后我们需要把数据放到一个对象中
MyUser.class
```java
public class MyUser {
    private String name;
    private String sex;
    private int age;

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
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
}
```
保存到对象
```java
    private void parseJson(String jsonString) {
        try {
            //把字符串转成 JSONObject 对象
            JSONObject jsonObject = new JSONObject(jsonString);
            //拿到参数
            int code = jsonObject.getInt("code");
            String message = jsonObject.getString("message");
            JSONObject userObject = jsonObject.getJSONObject("data");
            String name = userObject.getString("name");
            int age = userObject.getInt("age");
            String sex = userObject.getString("sex");
            MyUser myUser = new MyUser();
            myUser.setName(name);
            myUser.setAge(age);
            myUser.setSex(sex);
            System.out.println("code: " + code + ",message: " + message + ",data: " + myUser.getName() + myUser.getAge() + myUser.getSex());
        } catch (JSONException e) {
            e.printStackTrace();
        }
    }
}
```
#### 解析一个数组
##### 简单的数组解析
```json
["第一段文字","第二段文字","第三段文字"]
```
解析
```java
    private void parseJson(String jsonString) {
        try {
            //把字符串转成 JSONArray 对象
            JSONArray jsonArray = new JSONArray(jsonString);
            if (jsonArray.length() > 0) {
                for (int i = 0; i < jsonArray.length(); i++) {
                    String str = jsonArray.getString(i);
                }
            }
        } catch (JSONException e) {
            e.printStackTrace();
        }
    }
```

##### 包含JSONObject的数组
```json
[
    {
        "id":"xciodsfh",
        "title":"这是第一条"
    },
    {
        "id":"dsdxx",
        "title":"这是第二条"
    },
    {
        "id":"cxzz",
        "title":"这是第三条"
    }
]
```
解析
```json
    private void parseJson(String jsonString) {
        try {
            //把字符串转成 JSONArray 对象
            JSONArray jsonArray = new JSONArray(jsonString);
            if (jsonArray.length() > 0) {
                for (int i = 0; i < jsonArray.length(); i++) {
                    //拿到数组内JSONObject
                    JSONObject jsonObject = jsonArray.getJSONObject(i);
                    String id=jsonObject.getString("id");
                    String title=jsonObject.getString("title");
                }
            }
        } catch (JSONException e) {
            e.printStackTrace();
        }
    }
```















