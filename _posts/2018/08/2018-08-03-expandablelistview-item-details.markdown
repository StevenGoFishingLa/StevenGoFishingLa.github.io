---
layout: "post"
title: "Expandable ListView item details"
date: "2018-08-03 10:48"
categories: "Android"
tags: "android ExpandableListView listview details 摺疊 展開 詳細"
---
* content
{:toc}

先看一下我們今天要做甚麼!
基本款+美化款

![expand_only_one](/images/2018/08/expandablelistview-item-details/expand-only-one.gif)
![enhance](/images/2018/08/expandablelistview-item-details/enhance.gif)

有時候我們想要讓ListView有展開的效果

用Listview來做的話就是將group view 與child view包在一個view裡面

點一下(展開)，顯示child view

再點一下(摺疊)，隱藏child view



## 實作
釣魚去啦!

除了用ListView實現外這，邊提供用ExpandableListView的做法

個人覺得用ExpandableListView做起來code比較乾淨XD

OK..一步一步來
我們先將ExpandListView的箭頭拿掉，後面用自己想要的樣式取代他
```xml
<ExpandableListView
    android:groupIndicator="@null"<!--取消默認箭頭-->
    android:id="@+id/expand_list"
    android:layout_width="match_parent"
    android:layout_height="match_parent" />
```
將group與child改成自己想要的樣式
code我就不貼囉，有興趣再去git挖

group
![group_layout](/images/2018/08/expandablelistview-item-details/group-layout.png)

child

![child_layout](/images/2018/08/expandablelistview-item-details/child-layout.png)

## Data
接著我們處裡資料
將資料做成一個Object包起來

因為一個group 只有一個對應的child(該group的詳細資訊)

所以我們這邊不用再而外建立Child的資料，直接傳一個自訂的Object list去裡面撈就好
```java
public class HookDetails {
    String name;
    String price;
    String size;
    String color;
    String feature;
    String details;
    Drawable picture;
}
```
```java
private List<HookDetails> hooksList;
void initData() {
    hooksList = new ArrayList<>();
    HookDetails product_1 = new HookDetails();
    product_1.name ="激太チヌ(NSB)";
    product_1.price ="99元";
    product_1.size ="0.3 0.5 0.8 1 2 3 4 5 6 7 8 9 10 11 12";
    product_1.color ="NS Black(黑鉻色)";
    product_1.feature="*三角牙鉤尖";
    product_1.details ="<激太>是日文漢字,是.....魚鉤直徑加粗的意思。中魚後,可有效防止魚鉤被拉直或斷裂的情況。";
    product_1.picture = getResources().getDrawable(R.drawable.hook_1);
    hooksList.add(product_1);

    HookDetails product_2 = new HookDetails();
    product_2.name ="紫電チヌ";
    product_2.price ="50元";
    product_2.size ="0.5 0.8 1 2 3 4 5 6 7 8";
    product_2.color ="淡紫色(Light Purple)";
    product_2.feature="双重電鍍，大幅增加防鏽蝕功能。";
    product_2.details ="特殊新素材(HTC合金)，鉤身強度一級棒。";
    product_2.picture = getResources().getDrawable(R.drawable.hook_2);
    hooksList.add(product_2);
}
```
接著看我們自訂的adapter
```java
public class myBaseExpandableListAdapter extends BaseExpandableListAdapter {
    private Context context;
    private List<HookDetails> hooksList;
    private GroupViewHolder groupViewHolder;
    private Drawable moreDraw;
    private Drawable lessDraw;
    public myBaseExpandableListAdapter(Context context, List<HookDetails> hooks) {//傳入我們List的資料
        this.context = context;
        this.hooksList = hooks;
        moreDraw = context.getResources().getDrawable(R.drawable.ic_group_more);//展開箭頭
        lessDraw = context.getResources().getDrawable(R.drawable.ic_group_less);//摺疊箭頭
    }

    @Override
    public int getGroupCount() {
        return hooksList.size();
    }

    @Override
    public Object getGroup(int groupPosition) {
        return hooksList.get(groupPosition);
    }

    @Override
    public long getGroupId(int groupPosition) {
        return groupPosition;
    }

    private class GroupViewHolder {
        TextView nameTxt;
        TextView priceTxt;
        ImageView moreDetailsImg;
    }

    @Override
    public View getGroupView(int groupPosition, boolean isExpanded, View convertView, ViewGroup parent) {
        if (convertView == null) {
            convertView = LayoutInflater.from(context).inflate(R.layout.expand_list_group, null);
            groupViewHolder = new GroupViewHolder();
            groupViewHolder.nameTxt = convertView.findViewById(R.id.txt_name);
            groupViewHolder.priceTxt = convertView.findViewById(R.id.txt_price);
            groupViewHolder.moreDetailsImg = convertView.findViewById(R.id.img_more_details);
            convertView.setTag(groupViewHolder);
        } else {
            groupViewHolder = (GroupViewHolder) convertView.getTag();
        }
        groupViewHolder.nameTxt.setText(hooksList.get(groupPosition).name);
        groupViewHolder.priceTxt.setText(hooksList.get(groupPosition).price);

        if(isExpanded){//展開摺疊時，變更箭頭圖示
            groupViewHolder.moreDetailsImg.setImageDrawable(lessDraw);
        }else {
            groupViewHolder.moreDetailsImg.setImageDrawable(moreDraw);
        }
        return convertView;
    }

    @Override
    public int getChildrenCount(int groupPosition) {
        return 1;//一個group只有一個詳細資訊，所以設定為1
    }

    @Override
    public Object getChild(int groupPosition, int childPosition) {
        return hooksList.get(groupPosition);
    }

    @Override
    public long getChildId(int groupPosition, int childPosition) {
        return 0;
    }

    private class ChildViewHolder {
        TextView sizeTxt;
        TextView colorTxt;
        TextView featureTxt;
        TextView detailsTxt;
        ImageView pictureImg;
    }

    @Override
    public View getChildView(int groupPosition, int childPosition, boolean isLastChild, View convertView, ViewGroup parent) {
        ChildViewHolder childViewHolder;
        if (convertView == null) {
            convertView = LayoutInflater.from(context).inflate(R.layout.expand_list_child, null);
            childViewHolder = new ChildViewHolder();
            childViewHolder.sizeTxt = convertView.findViewById(R.id.txt_size);
            childViewHolder.colorTxt = convertView.findViewById(R.id.txt_color);
            childViewHolder.featureTxt = convertView.findViewById(R.id.txt_feature);
            childViewHolder.detailsTxt = convertView.findViewById(R.id.txt_details);
            childViewHolder.pictureImg = convertView.findViewById(R.id.img_picture);
            convertView.setTag(childViewHolder);
        } else {
            childViewHolder = (ChildViewHolder) convertView.getTag();
        }
        childViewHolder.sizeTxt.setText(hooksList.get(groupPosition).size);
        childViewHolder.colorTxt.setText(hooksList.get(groupPosition).color);
        childViewHolder.featureTxt.setText(hooksList.get(groupPosition).feature);
        childViewHolder.detailsTxt.setText(hooksList.get(groupPosition).details);
        childViewHolder.pictureImg.setImageDrawable(hooksList.get(groupPosition).picture);
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
如果上一篇有看懂的話這篇應該很容易就可以上手

就是把原本的兩個list data縮減成一個

這樣就完成拉!

![basic](/images/2018/08/expandablelistview-item-details/basic.gif)

下面是一些延伸
## 一次只能展開一個
```java
expandableListView.setOnGroupExpandListener(new ExpandableListView.OnGroupExpandListener() {
    @Override
    public void onGroupExpand(int groupPosition) {
        // TODO Auto-generated method stub
        for (int i = 0; i < adapter.getGroupCount(); i++) {
            if (groupPosition != i) {
                expandableListView.collapseGroup(i);
            }
        }
    }
});

```
![expand_only_one](/images/2018/08/expandablelistview-item-details/expand-only-one.gif)


## 美化
再來家個外框，把它包起來，視覺效果好一點
(上面那個鉤子沒有去背，後來換了一個張XD)

![enhance](/images/2018/08/expandablelistview-item-details/enhance.gif)

我們的目標是將每個item都用框包起來，展開時變更框色、填充色

這樣看起來更有(獨立)選中的效果

主要做法就是將我們的group與child加入border background

先從未展開時開始處裡
在drawable新增Drawable resource file

border_group_close.xml
```xml
<?xml version="1.0" encoding="utf-8"?>
<shape xmlns:android="http://schemas.android.com/apk/res/android" android:shape="rectangle" >
    <corners android:radius="5dp" />
    <solid android:color="#FFFFFF" />
    <stroke android:width="1dip" android:color="@android:color/holo_blue_light"/>
</shape>
```
再來就是設定background

我這邊多用一層LinearLayout包起來，因為如果不多包一層layout_margin會沒有作用~

這裡有說明

[listView item 增加间距 以及item根部局 margin 失效原因](https://blog.csdn.net/android_freshman/article/details/52077323)

[ListView的Item根布局layout_margin无效分析](https://blog.csdn.net/ysq_chris/article/details/80775212)
```xml
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="wrap_content"
    xmlns:app="http://schemas.android.com/apk/res-auto">

    <LinearLayout
        android:id="@+id/ll_group"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:layout_marginEnd="8dp"
        android:layout_marginLeft="8dp"
        android:layout_marginRight="8dp"
        android:layout_marginStart="8dp"
        android:layout_marginTop="8dp"
        android:background="@drawable/border_group_close">

        <ImageView
            android:id="@+id/imageView"
            android:layout_width="48dp"
            android:layout_height="48dp"
            android:layout_marginBottom="8dp"
            android:layout_marginLeft="8dp"
            android:layout_marginTop="8dp"
            app:srcCompat="@drawable/fishing" />

            ...
            ...
    </LinearLayout>

</LinearLayout>
```

先將background設成我們畫的眶

再來除了左右各加了layout_margin，頂部也要加，這邊主要是用於讓每個item有間距

可能有人會問，為何不直接設定ExpandableListView
```xml
android:dividerHeight="8dp"
```
就好了?
因為會變這樣...

![dividerHeight](/images/2018/08/expandablelistview-item-details/dividerheight.jpg)


Child的背景框，頂部線拿掉，下面加入圓角

border_detail.xml
```xml
<?xml version="1.0" encoding="utf-8"?>
<layer-list xmlns:android="http://schemas.android.com/apk/res/android">
    <item
        android:top="-2dp">
        <shape>
            <solid android:color="#E2E2E2" />
            <corners
                android:topLeftRadius="0dp"
                android:topRightRadius="0dp"
                android:bottomRightRadius="5dp"
                android:bottomLeftRadius="5dp" />
            <stroke
                android:width="1dp"
                android:color="@android:color/background_dark" />
        </shape>
    </item>
</layer-list>
```
之後再設定child的background就好囉
```xml
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    android:layout_width="match_parent"
    android:layout_height="match_parent">

    <android.support.constraint.ConstraintLayout
        android:background="@drawable/border_detail"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:layout_marginEnd="8dp"
        android:layout_marginLeft="8dp"
        android:layout_marginRight="8dp"
        android:layout_marginStart="8dp">

        <ImageView
            android:id="@+id/img_picture"
            android:layout_width="128dp"
            android:layout_height="192dp"
            android:layout_marginBottom="8dp"
            android:layout_marginLeft="8dp"
            android:layout_marginStart="8dp"
            android:layout_marginTop="8dp"
            app:layout_constraintBottom_toBottomOf="parent"
            app:layout_constraintStart_toStartOf="parent"
            app:layout_constraintTop_toTopOf="parent"/>

        ...
        ...
    </android.support.constraint.ConstraintLayout>

</LinearLayout>
```

## Group 與 Child 間隙
就算不用dividerHeight也還是會發現Group 與 Child 還是會存在一點點間隙
後來我找到這個方法，就可很徹底黏在一起囉
```xml
android:divider="@null"
```
## 展開 更換背景
group未展開時的背景處裡完了，再來要接著處裡展開時，group的背景樣式

gorup要將原本的眶最下底線拿掉，並改變背景顏色

border_group_expand.xml
```xml
<?xml version="1.0" encoding="utf-8"?>
<layer-list xmlns:android="http://schemas.android.com/apk/res/android">
    <item
        android:bottom="-2dp">
        <shape>
            <solid android:color="#E2E2E2" />
            <corners
                android:topLeftRadius="5dp"
                android:topRightRadius="5dp"
                android:bottomRightRadius="0dp"
                android:bottomLeftRadius="0dp" />
            <stroke
                android:width="1dp"
                android:color="@android:color/background_dark" />
        </shape>
    </item>
</layer-list>
```
我們要在group展開時，將group的background更改為上面內容
所以回到我們的myBaseExpandableListAdapter的getGroupView裡
加入
```java
if(isExpanded){
    groupViewHolder.moreDetailsImg.setImageDrawable(lessDraw);
    groupViewHolder.groupLl.setBackgroundResource(R.drawable.border_group_expand);//*

}else {
    groupViewHolder.moreDetailsImg.setImageDrawable(moreDraw);
    groupViewHolder.groupLl.setBackgroundResource(R.drawable.border_group_close);//*
}
```
這樣就大功告成囉!
