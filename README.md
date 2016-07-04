
- 当你在Activity中使用内部类的时候，需要时刻考虑您是否可以控制该内部类的生命周期，如果不可以，则最好定义为静态内部类
- TextView的任意文本变为链接：
```
      //First make sure TextView not set "autoLink" property
      //Method 1
      SpannableString ss=new SpannableString("Sign up");
      String url="http://www.baidu.com";  //Prefix must be "http://"
      ss.setSpan(new URLSpan(url), 0, ss.length(), Spanned.SPAN_EXCLUSIVE_EXCLUSIVE);
      textView.setText(ss);
      textView.setMovementMethod(LinkMovementMethod.getInstance());
        
      //Method 2
      textView.setText(Html.fromHtml("<a href=\"http://www.baidu.com\">Your text here</a>"));
      textView.setMovementMethod(LinkMovementMethod.getInstance());
```

- moveToBack(true)：把含有当前activity的task转移到后台。如果是false则当前activity为root activity才生效。
- 一般来说，`android:layout_****`属性应该定义在布局文件里，而其他属性`android:****`应该放在style的XML文件里。这个规则的思想是保持布局和内容属性（位置、margin、尺寸）在layout文件里，保持显示细节（颜色、padding、字体）在style文件里。例外情况是：`android:id`和LinearLayout的`android:iorientation`显然应该放在layout文件里；`android:text`应该在layout文件因为它定义了内容；有时候可以把layout_width和layout——height放在style文件里来做成通用样式，但是默认是要放在layout里面。
- 要用styles。几乎每个项目都得合适地使用styles，因为对于view来说有重复的外观是很常见的。至少你得给app的大部分文本内容定义一个通用样式：

```
<style name="ContentText">
    <item name="android:textSize">@dimen/font_normal</item>
    <item name="android:textColor">@color/basic_black</item>
</style>

//Applied to TextViews:
<TextView
    android:layout_width="wrap_content"
    android:layout_height="wrap_content"
    android:text="@string/price"
    style="@style/ContentText"
    />
```

- 你可以把一个大的style文件分割成几个文件。`styles.xml`的文件名没啥神奇的，重要的是文件里的`<style>`标签。因此你可以有style_home.xml, styles_item_details.xml之类的。不像build系统下的其他文件夹名，values下面的文件名可以很随意。
- AS编译R文件丢失，常见原因是: 1. xml有错误 2. res下面的资源文件格式有问题(如，jpeg的图片用的却是png的后缀)
- ///


