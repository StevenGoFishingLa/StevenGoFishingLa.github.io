---
layout: "post"
title: "ExpandableListview inside Scrollview"
date: "2018-08-06 15:01"
categories: "Android"
tags: "android ExpandableListView ScrollView"
---
* content
{:toc}

之前遇到兩個問題~分享一下

1.ScrollView嵌套ExpandableListView滑動異常

2.進入頁面無法置頂



## 無法整個頁面滑動
ScrollView 包 ExpandableListview 滾動上遇到問題，如圖

![scrollview-problem](/images/2018/08/expandablelistview-inside-scrollview/scrollview-problem.gif)

我明明TextView也有包起來呀!
```xml
<?xml version="1.0" encoding="utf-8"?>
<ScrollView xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:fillViewport="true"
    tools:context=".MainActivity">

    <LinearLayout
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:orientation="vertical">

        <TextView
            android:id="@+id/txt_title_hooks"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:layout_marginBottom="8dp"
            android:layout_marginLeft="16dp"
            android:layout_marginStart="16dp"
            android:layout_marginTop="8dp"
            android:text="魚鉤"
            android:textSize="24sp" />

        <ExpandableListView
            android:id="@+id/expand_list"
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:divider="@null"
            android:groupIndicator="@null" />
    </LinearLayout>
</ScrollView>
```
### 解決方法
查了一下解決方案，自訂ExpandListView，重寫onMeasure
```java
public class CustomExpandableListView extends ExpandableListView {

    public CustomExpandableListView(Context context) {
        super(context);
        // TODO Auto-generated constructor stub
    }

    public CustomExpandableListView(Context context, AttributeSet attrs) {
        super(context, attrs);
        // TODO Auto-generated constructor stub
    }

    public CustomExpandableListView(Context context, AttributeSet attrs,
                                    int defStyle) {
        super(context, attrs, defStyle);
        // TODO Auto-generated constructor stub
    }
    @Override
    protected void onMeasure(int widthMeasureSpec, int heightMeasureSpec) {
        // TODO Auto-generated method stub
        int expandSpec = MeasureSpec.makeMeasureSpec(Integer.MAX_VALUE >> 2,
                MeasureSpec.AT_MOST);
        super.onMeasure(widthMeasureSpec, expandSpec);
    }
}
```
接著直接使用就可以囉!
```xml
<com.example.steven.expandablelistview.CustomExpandableListView
    android:id="@+id/expand_list"
    android:layout_width="match_parent"
    android:layout_height="wrap_content"
    android:divider="@null"
    android:groupIndicator="@null" />
```

詳細原理請看~~
[详解嵌套ListView、ScrollView布局显示不全的问题](https://blog.csdn.net/hanhailong726188/article/details/46136569)

## 進入頁面，不在最頂部
![not-top](/images/2018/08/expandablelistview-inside-scrollview/not-top.gif)
### 解決方法
在配置完adapter後，取消該ExpandableListView焦點
```java
expandableListView.setFocusable(false);
```
![auto-top](/images/2018/08/expandablelistview-inside-scrollview/auto-top.gif)
