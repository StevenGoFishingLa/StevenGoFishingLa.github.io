---
layout: "post"
title: "Ip Address EditText"
date: "2018-07-30 19:09"
categories: Android
tags: ip address EditText
---

* content
{:toc}

上次做了Mac address的EditText
這次來做Ip的吧

## 演示
![get ip address](/images/2018/07/ip-address-edittext/get-ip-address.gif)



自動換欄

![auto next](/images/2018/07/ip-address-edittext/auto-next.gif)

驗證

![check ip valid](/images/2018/07/ip-address-edittext/check-ip-valid.gif)

最終驗證

![final check](/images/2018/07/ip-address-edittext/final-check.gif)

## 實作
就把MacEditText拿來改一改囉

不過這次要將輸入長度限制為3，並指允許輸入數字
```java
public class IpEditText extends LinearLayout {
    private static final int OCTET_NUM = 4;

    private EditText ipOctet[] = new EditText[OCTET_NUM];

    public IpEditText(Context context) {
        super(context);
        InitUI();
        checkInput();
    }

    public IpEditText(Context context, @Nullable AttributeSet attrs) {
        super(context, attrs);
        InitUI();
        checkInput();
    }

    public IpEditText(Context context, @Nullable AttributeSet attrs, int defStyleAttr) {
        super(context, attrs, defStyleAttr);
        InitUI();
        checkInput();
    }

    private void InitUI() {
        TextView colonTxt[] = new TextView[OCTET_NUM - 1];
        for (int i = 0; i < OCTET_NUM; i++) {
            ipOctet[i] = new EditText(getContext());
            ipOctet[i].setTag(i);
            addView(ipOctet[i]);
            if (i < OCTET_NUM - 1) {
                colonTxt[i] = new TextView(getContext());
                colonTxt[i].setText(".");
                colonTxt[i].setTextSize(24);
                addView(colonTxt[i]);
            }
        }

        for (int i = 0; i < OCTET_NUM; i++) {
            ipOctet[i].setGravity(Gravity.CENTER);
            ipOctet[i].setFilters(new InputFilter[]{new InputFilter.LengthFilter(3)});
            ipOctet[i].setInputType(InputType.TYPE_CLASS_NUMBER);
            LayoutParams params = (LayoutParams) ipOctet[i].getLayoutParams();
            ipOctet[i].setSingleLine(true);
            ipOctet[i].setImeOptions(EditorInfo.IME_ACTION_NEXT);//set keyboard nex button
            params.weight = 1;
            params.width = 0;
        }
    }
}
```
輸入時檢查Ip開頭是否大於0，其他是否在0~255區間內

並且監聽鍵盤下一步按鈕，一樣加入驗證機制
```java
private void checkInput() {
    for (int i = 0; i < OCTET_NUM; i++) {
        ipOctet[i].addTextChangedListener(new MyTextWatcher(ipOctet[i]) {
            @Override
            public void beforeTextChanged(CharSequence s, int start, int count, int after) {
            }

            @Override
            public void onTextChanged(CharSequence s, int start, int before, int count) {
                EditText editText = getEditTextView();
                int ipByte;
                if (editText.getText().length() > 0) {
                    ipByte = Integer.valueOf(editText.getText().toString());
                } else {
                    ipByte = -1;
                }
                if ((editText == ipOctet[0] && ipByte < 1) || ipByte > 255) {
                    addWarnAnim(editText);
                }
                int position = (int) editText.getTag();
                if (s.length() == 3) {
                    if (position < OCTET_NUM - 1) {
                        ipOctet[position + 1].requestFocus();
                    }
                }
            }

            @Override
            public void afterTextChanged(Editable s) {
            }
        });
        ipOctet[i].setOnEditorActionListener(new TextView.OnEditorActionListener() {
            @Override
            public boolean onEditorAction(TextView editText, int actionId, KeyEvent event) {
                int ipByte;
                if (editText.getText().length() > 0) {
                    ipByte = Integer.valueOf(editText.getText().toString());
                } else {
                    ipByte = -1;
                }
                if (actionId == EditorInfo.IME_ACTION_NEXT) {
                    if ((editText == ipOctet[0] && ipByte < 1) || ipByte > 255 || ipByte < 0) {
                        addWarnAnim((EditText) editText);
                        return true;
                    }
                }
                return false;
            }
        });
    }
}
```
取值也一樣，檢查開頭，與其他欄位是否在範圍內
```java
public String getIpAddress() {
    StringBuffer sb = new StringBuffer();
    int ipByte;
    for (int i = 0; i < OCTET_NUM; i++) {
        if (ipOctet[i].getText().length() > 0) {
            ipByte = Integer.valueOf(ipOctet[i].getText().toString());
        } else {
            ipByte = -1;
        }
        if ((ipOctet[i] == ipOctet[0] && ipByte < 1) || ipByte > 255 || ipByte < 0) {
            addWarnAnim(ipOctet[i]);
            ipOctet[i].requestFocus();
            return null;
        }
        sb.append(ipOctet[i].getText());
        if (i < OCTET_NUM - 1) {
            sb.append(".");
        }
    }
    return sb.toString();
}
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
[完整範例專案](https://github.com/StevenGoFishingLa/IpEditText)


參考至:

[组合控件--实现Ip地址的输入与校验](http://www.voidcn.com/article/p-xtjfnzbl-boa.html)
