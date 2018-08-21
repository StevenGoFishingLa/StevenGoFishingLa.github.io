---
layout: "post"
title: "Java String split"
date: "2018-08-01 13:55"
categories: "Java"
tags: "java sring split dot regex"
---
* content
{:toc}

## 問題
摁剛剛在切割字串時發現
怎麼沒作用啦?

```java
void test(){
  String input = "abc.png";
  String str[] = input.split(".");
  for (String output :str){
      Log.d("output",""+output);
  }
}
```



查一下查發現，原來是'.'在正規樣式Regex代表任何一個字元

也就是說我上面的code把所有字元都當作分隔字元被拿掉啦!

## 解決方法
主要有兩種
### 第一種
加入跳脫符號
```java
String str[] = input.split("\\.");
```

### 第二種
```java
String str[] = input.split("[.]");
```
包起來囉

另外說一下第二種方式
```java
String str[] = input.split("[.,]");
```
這樣表示遇到'.'與','都當作切割字元
### 這些字元也是
還有這些符號也都需要用到上面的方法才能正確地切割
```java
"|"
"."
"*"
"_"
"+"
```
