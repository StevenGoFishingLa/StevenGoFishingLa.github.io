---
layout: post
title:  "Mac Address EditText"
date:   2017-07-30 19:06:05
categories: Android
tags: mac address EditText
---
* content
{:toc}

開發中遇到要輸入MAC Address，卻發現android好像沒有現成的可用

只能多拉幾個EditText，然後合併取值

每次都要拉一大堆EditText，乾脆做一個包起來

直接來看我們想要達到甚麼樣的效果
## 演示

![get_mac_address](/images/2018/07/mac-address-edittext/get-mac-address.gif)



當然只有這樣是不夠的…
輸入完自動跳下一格吧!

![auto_next](/images/2018/07/mac-address-edittext/auto-next.gif)

覺得還是不夠…再加個驗證吧

![mac_check_valid](/images/2018/07/mac-address-edittext/mac-check-valid.gif)


最後在取值時也加入驗證+跳到錯誤欄位

![final_check](/images/2018/07/mac-address-edittext/final-check.gif)
## 實作
OK~釣魚去囉

我們新增一個Class繼承 LinearLayout 在裡面加入6個EditText 與5個TextView(分隔冒號)
```java
import android.content.Context;
import android.support.annotation.Nullable;
import android.text.InputFilter;
import android.text.Spanned;
import android.util.AttributeSet;
import android.view.Gravity;
import android.view.inputmethod.EditorInfo;
import android.widget.EditText;
import android.widget.LinearLayout;
import android.widget.TextView;

public class MacEditText extends LinearLayout {
    private static final int OCTET_NUM = 6;
    private EditText macOctet[] = new EditText[OCTET_NUM];

    public MacEditText(Context context) {
        super(context);
        InitUI();
    }
    public MacEditText(Context context, @Nullable AttributeSet attrs) {
        super(context, attrs);
        InitUI();
    }
    public MacEditText(Context context, @Nullable AttributeSet attrs, int defStyleAttr) {
        super(context, attrs, defStyleAttr);
        InitUI();
    }

    InputFilter filter = new InputFilter() {
        public CharSequence filter(CharSequence source, int start, int end,
                                    Spanned dest, int dstart, int dend) {
            for (int i = start; i < end; i++) {
                if (!Character.isLetterOrDigit(source.charAt(i))) {
                  //allow only numbers and alphabets
                    return "";
                }
            }
            return null;
        }
    };

    private void InitUI() {
        TextView colonTxt[] = new TextView[OCTET_NUM - 1];
        for (int i = 0; i < OCTET_NUM; i++) {
            macOctet[i] = new EditText(getContext());
            macOctet[i].setTag(i);
            addView(macOctet[i]);
            if (i < OCTET_NUM - 1) {
                colonTxt[i] = new TextView(getContext());
                colonTxt[i].setText(":");
                colonTxt[i].setTextSize(24);
                addView(colonTxt[i]);
            }
        }

        for (int i = 0; i < OCTET_NUM; i++) {
            macOctet[i].setGravity(Gravity.CENTER);
            macOctet[i].setFilters(new InputFilter[]{new InputFilter.LengthFilter(2), filter});
            //input max length = 2, allow only numbers and alphabets
            LayoutParams params = (LayoutParams) macOctet[i].getLayoutParams();
            macOctet[i].setSingleLine(true);
            macOctet[i].setImeOptions(EditorInfo.IME_ACTION_NEXT);//set keyboard nex button
            params.weight = 1;
            params.width = 0;
        }
    }
}
```
接著到我們的layout裡面增加就可以看到長怎樣囉
```xml
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:orientation="vertical"
    tools:context=".MainActivity">

    <com.example.steven.macedittext.MacEditText
        android:id="@+id/macEditText"
        android:layout_width="match_parent"
        android:layout_height="wrap_content" />

    <Button
        android:onClick="onClick"
        android:id="@+id/button"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:text="Get Mac Address" />

</LinearLayout>
```
![macedittext](/images/2018/07/mac-address-edittext/macedittext.png)

接著做功能

將每個EditText加上TextChangedListener，每次使用者輸入文字時，檢查是否合法。
```java
macOctet[i].addTextChangedListener(new MyTextWatcher(macOctet[i]) {
      @Override
      public void beforeTextChanged(CharSequence s, int start, int count, int after) {
      }

      @Override
      public void onTextChanged(CharSequence s, int start, int before, int count) {
          EditText editText = getEditTextView();
          String trueMacAddress = "[A-Fa-f0-9]";
          int position = (int) editText.getTag();
          if (s.length() == 1) {
              if (!s.toString().matches(trueMacAddress)) {
                  //Toast.makeText(getContext(), "This is not a valid Mac Address", Toast.LENGTH_SHORT).show();
                  addWarnAnim(editText);
              }
          } else if (s.length() == 2) {
              trueMacAddress = "[A-Fa-f0-9]{2}";
              if (!s.toString().matches(trueMacAddress)) {
                  //Toast.makeText(getContext(), "This is not a valid Mac Address", Toast.LENGTH_SHORT).show();
                  addWarnAnim(editText);
              } else {
                  if (position < OCTET_NUM - 1) {
                      macOctet[position + 1].requestFocus();
                  }
              }
          }
      }

      @Override
      public void afterTextChanged(Editable s) {
      }
  });
```
再加上EditorActionListener，監聽鍵盤下一步按鈕，按下一步時一併檢查
```java
macOctet[i].setOnEditorActionListener(new TextView.OnEditorActionListener() {
    @Override
    public boolean onEditorAction(TextView v, int actionId, KeyEvent event) {
        if (actionId == EditorInfo.IME_ACTION_NEXT) {
            String trueMacAddress = "[A-Fa-f0-9]{2}";
            if (v.getText().toString().matches(trueMacAddress)) {
                return false;
            } else {
                //Toast.makeText(getContext(), "This is not a valid Mac Address", Toast.LENGTH_SHORT).show();
                addWarnAnim((EditText) v);
            }
        }
        return true;
    }
});
```
錯誤提示動畫
```java
private void addWarnAnim(EditText editText) {
    AnimationSet animationSet = new AnimationSet(true);
    animationSet.setRepeatCount(8);
    TranslateAnimation rightAnim = new TranslateAnimation(0, 30, 0, 0);
    rightAnim.setDuration(60);
    TranslateAnimation leftAnim = new TranslateAnimation(30, 0, 0, 0);
    rightAnim.setDuration(60);
    animationSet.addAnimation(rightAnim);
    animationSet.addAnimation(leftAnim);
    editText.startAnimation(animationSet);
}
```

合併取出完整Mac Address
逐欄檢查，如果有不合法的，就跳到該欄位並顯示動畫
```java
public String getMacAddress() {
    StringBuffer sb = new StringBuffer();
    String trueMacAddress = "[A-Fa-f0-9]{2}";
    for (int i = 0; i < OCTET_NUM; i++) {
        if (!macOctet[i].getText().toString().matches(trueMacAddress)) {
            //Toast.makeText(getContext(), "This is not a valid Mac Address", Toast.LENGTH_SHORT).show();
            addWarnAnim(macOctet[i]);
            macOctet[i].requestFocus();
            return null;
        }
        sb.append(macOctet[i].getText());
        if (i < OCTET_NUM - 1) {
            sb.append(":");
        }
    }
    return sb.toString();
}
```
最後取出時，如果為null，表示該Mac Address 不合法

```java
String macAdd = macEditText.getMacAddress();
if(macAdd==null){
    Toast.makeText(this, "This is not a valid Mac Address", Toast.LENGTH_SHORT).show();
}else{
    Toast.makeText(this, macAdd, Toast.LENGTH_SHORT).show();
}
```

[完整範例專案](https://github.com/StevenGoFishingLa/MacEditText)

後來想到好像可以自行設定鍵盤內容，mac address只允許0~9 a-F

之後覺得有必要再來改囉!


參考至:

[MAC地址合法性检测
](https://blog.csdn.net/luzhenrong45/article/details/52979750)

[组合控件--实现Ip地址的输入与校验](http://www.voidcn.com/article/p-xtjfnzbl-boa.html)
