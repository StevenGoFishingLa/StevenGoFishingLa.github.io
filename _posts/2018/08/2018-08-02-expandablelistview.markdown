---
layout: "post"
title: "ExpandableListView 基本用法"
date: "2018-08-02 10:12"
categories: "Android"
tags: "android ExpandableListView listview 摺疊"
---
* content
{:toc}

今天來分享下ExpandableListView一些應用

ExpandableListView簡單說起來就像是一個ListView包著一個Listview?

先從簡單的做起吧~
## 基本
![basic](/images/2018/08/expandablelistview/basic.gif)



先介紹layout檔需要準備哪些

先在主頁面加入ExpandableListView

activity_main.xml
```java
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    tools:context=".MainActivity">
    <ExpandableListView
        android:id="@+id/expand_list"
        android:layout_width="match_parent"
        android:layout_height="match_parent" />
</LinearLayout>
```

再來我們需要準備父層(group)與子層(child)的layout
最簡單的就是放各一個TextView在裡面
## layout files
```java
expand_list_group.xml
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent">
    <TextView
        android:id="@+id/txt_expand_group"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:gravity="center"
        android:text="Group"
        android:textSize="24sp" />
</LinearLayout>
```
expand_list_child.xml
```java
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent">
    <TextView
        android:id="@+id/txt_expand_child"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:gravity="center"
        android:text="Child"
        android:textSize="16sp" />
</LinearLayout>
```
## Data
接著我們準資料
```java
private List<String> groupList;
private List<List<String>> childList;
void initData() {
    groupList = new ArrayList<>();
    groupList.add("魚");
    groupList.add("蝦");
    groupList.add("蟹");

    childList = new ArrayList<>();
    List<String> child_list_fish = new ArrayList<>();
    child_list_fish.add("黑鯛");
    child_list_fish.add("鱸魚");
    childList.add(child_list_fish);

    List<String> child_list_shrimp = new ArrayList<>();
    child_list_shrimp.add("白蝦");
    child_list_shrimp.add("沙蝦");
    childList.add(child_list_shrimp);

    List<String> child_list_crab = new ArrayList<>();
    child_list_crab.add("青蚶");
    child_list_crab.add("紅腳");
    child_list_crab.add("薄仔");
    childList.add(child_list_crab);
}
```
簡單來說就是準備兩個String 陣列

一個放父資料，一個放子資料

用這樣應該會好理解很多，但用List比較好處理
```java
String[] group = {"魚", "蝦", "蟹"};
String[][] child = {
        {"黑鯛", "鱸魚"},
        {"白蝦", "沙蝦"},
        {"青蚶", "紅腳", "薄仔"}
                    };

```
## Adapter
再來就是設計adapter囉，這邊我們繼承BaseExpandableListAdapter來處理

```java
public class myBaseExpandableListAdapter extends BaseExpandableListAdapter {
    private Context context;
    private List<String> groupList;
    private List<List<String>> childList;

    public myBaseExpandableListAdapter(Context context, List<String> groupList, List<List<String>> childList) {//將資料傳入
        this.context = context;
        this.groupList = groupList;
        this.childList = childList;
    }

    @Override
    public int getGroupCount() {//記得改~不燃Adapter抓不到資料哦
        return groupList.size();
    }

    @Override
    public Object getGroup(int groupPosition) {//這裡也是
        return groupList.get(groupPosition);
    }

    @Override
    public long getGroupId(int groupPosition) {//這裡我不太確定用途，有請高手補充
        return groupPosition;
    }

    private class GroupViewHolder {//ViewHolder 可以加快顯示速度，減少View刷新時一直創建
        TextView groupText;
    }

    @Override
    public View getGroupView(int groupPosition, boolean isExpanded, View convertView, ViewGroup parent) {
      GroupViewHolder groupViewHolder;
        if (convertView == null) {//view不存在，第一次創建
            convertView = LayoutInflater.from(context).inflate(R.layout.expand_list_group, null);
            groupViewHolder = new GroupViewHolder();
            groupViewHolder.groupText = convertView.findViewById(R.id.txt_expand_group);
            convertView.setTag(groupViewHolder);//group第一次創建時，新增此group 的tagID
        } else {//使用以創建的view
            groupViewHolder = (GroupViewHolder) convertView.getTag();
        }
        groupViewHolder.groupText.setText(groupList.get(groupPosition));//讀取資料並顯示
        return convertView;
    }
    //chlid view部分大致上都一樣，要注意的是要取list的第二層內容
    @Override
    public int getChildrenCount(int groupPosition) {
        return childList.get(groupPosition).size();
    }

    @Override
    public Object getChild(int groupPosition, int childPosition) {
        return childList.get(groupPosition).get(childPosition);
    }

    @Override
    public long getChildId(int groupPosition, int childPosition) {
        return childPosition;
    }

    private class ChildViewHolder {
        TextView childText;
    }

    @Override
    public View getChildView(int groupPosition, int childPosition, boolean isLastChild, View convertView, ViewGroup parent) {
      ChildViewHolder childViewHolder;
        if (convertView == null) {
            convertView = LayoutInflater.from(context).inflate(R.layout.expand_list_child, null);
            childViewHolder = new ChildViewHolder();
            childViewHolder.childText = convertView.findViewById(R.id.txt_expand_child);
            convertView.setTag(childViewHolder);
        } else {
            childViewHolder = (ChildViewHolder) convertView.getTag();
        }
        childViewHolder.childText.setText(childList.get(groupPosition).get(childPosition));
        return convertView;
    }

    @Override
    public boolean isChildSelectable(int groupPosition, int childPosition) {
        return false;
    }

    @Override
    public boolean hasStableIds() {
        return false;
    }
}
```
### ViewHolder
這是一個很實用的技巧，可以用在很多地方，可以大幅提升app效能，建議可以把它學起來

這邊提供幾個幾個解說不錯的網址~

[ListView之BaseAdapter的基本使用以及ViewHolder模式](http://www.jcodecraeer.com/a/anzhuokaifa/androidkaifa/2014/1021/1815.html)


## 套用
以上準備工作完成就可以套用囉!
```java
private ExpandableListView expandableListView;
private myBaseExpandableListAdapter adapter;
void initView() {
    expandableListView = findViewById(R.id.expand_list);
}
void initAdapter() {
    adapter = new myBaseExpandableListAdapter(this,groupList,childList);
    expandableListView.setAdapter(adapter);
}
@Override
protected void onCreate(Bundle savedInstanceState) {
    super.onCreate(savedInstanceState);
    setContentView(R.layout.activity_main);
    initView();
    initData();
    initAdapter();
}
```
完成~

下一篇來看不同的用法!
