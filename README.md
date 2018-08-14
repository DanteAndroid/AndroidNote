- ### [Kotlin Note](/KotlinNote.md)
- ### [Git Note](/GitNote.md)

- 最强AS写代码神器 [Exynap](http://exynap.com/)
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
- [app框架搭建笔记](http://www.jianshu.com/p/3b3932d3afab)
- **Transition转场动画**
      0. EnterTransition顾名思义进入activity时的动画，在进入的activity的onCreate中设置，ReenterTransition即从之前打开的activity返回来时的动画（第二次回到这个activity）。这两个动画都是进入动画，即现在打开这个activity时的动画; ReturnTransition返回动画，即窗口关闭时（如在此activity时点击back时）呈现的动画。ExitTransition退出动画，在此activity中跳转到新的activity时的动画，需要跟returnTransition做区别。这两个动画都是离开动画，即现在离开这个activity的动画；setAllowEnterTransitionOverlap是否允许第一次进入这个activity时的动画(EnterTransition)覆盖。setAllowReturnTransitionOverlap是否允许返回这个activity时的动画(ReenterTransition)覆盖，如果是，那么返回这个activity的动画立刻执行。
      1.Return和Reenter Transitions分别是Enter和Exit的反向动画（如果没有特别设置的话）
      1. xml中设置：在activity的主题中添加<item name="android:windowXXXTransition">@transition/CustomTransitionXML</item>其中自自定义的transition文件在res/transition下面，根元素是transitionSet
      2. 代码中设置： getWindow().setXXXTransition(transition);这里的transition对象可以直接new。有Slide, Explode, Fade三种默认的动画。
      3. Transition默认会应用到View树的所有view上，但是用`addTarget()`可以单独为某些view应用动画。      


- animator写动画效果时，遇到`java.lang.IllegalStateException: Already started!`错误，是因为没有setListen()。因为之前已经使用了`animate()`方法并且setListener了，所以这个错误是由于用的是之前的listener对象造成的。

- fragment自定义动画时，方法应该写在replace(or add)前面，否则不会生效，例如：
```
                getSupportFragmentManager().beginTransaction()
                        .setCustomAnimations(android.R.anim.slide_in_left(进入动画), android.R.anim.slide_out_right,
                                android.R.anim.slide_in_left, android.R.anim.slide_out_right(退出动画))
                        .replace(R.id.fragment_container, new RRFragment())
                        //当前transition加入回退栈，即，点击返回可以回到replace之前的状态
                        .addToBackStack("")
                        .commit();
```

- 项目的包应该优先按照功能（模块）划分，而非类型
- 程序跑起来不意味着会按预期一样运行。把调试当做是补充知识点的机会；总有一天你会发现，比起写代码你会花更多时间在调试bug上面。
- 调试时，子线程中的断点应该在debugger初始化之后（运行后）再选择，否则可能不会被触发。
- `logm` - Log current method name and its arguments ; `logr` - Log result of the current method ; `noInstance` - private empty constructor to prohibit instance creation
- 调试技巧：按下home，然后在terminal中输入adb shell am kill com.your.packagename即可模拟杀掉后台
- Try to understand and follow TDD (Test Driven Development) ; Follow the DRY principle DRY = Do not Repeat Yourself
- [Package by Feature, not layers](https://medium.com/the-engineering-team/package-by-features-not-layers-2d076df1964d)
- [依赖冲突的解决方案](http://crushingcode.co/the-curious-case-of-dependency-conflicts/)，[参考]（https://docs.gradle.org/current/dsl/org.gradle.api.artifacts.ResolutionStrategy.html）
- [避免内存泄露](http://blog.nimbledroid.com/2016/09/06/stop-memory-leaks.html)
- [资源命名格式](http://jeroenmols.com/blog/2016/03/07/resourcenaming/)
- [用Handler替代Timer](http://www.mopri.de/2010/timertask-bad-do-it-the-android-way-use-a-handler/)
- 在新的task中打开activity：M1. launchMode="singleTask" + android:taskAffinity="new.package.name" 在启动一个singleTask的Activity实例时，如果系统中已经存在这样一个实例，就会将这个实例调度到任务栈的栈顶，并清除它当前所在任务中位于它上面的所有的activity。 M2. `intent.setFlags(Intent.FLAG_ACTIVITY_NEW_TASK);` + android:taskAffinity="new.package.name" 
-   以singleInstance模式启动的Activity具有全局唯一性，即整个系统中只会存在一个这样的实例
    以singleInstance模式启动的Activity具有独占性，即它会独自占用一个任务，被他开启的任何activity都会运行在其他任务中（官方文档上的描述为，      singleInstance模式的Activity不允许其他Activity和它共存在一个任务中）
    被singleInstance模式的Activity开启的其他activity，能够开启一个新任务，但不一定开启新的任务，也可能在已有的一个任务中开启

- Deep Link的配置(添加到启动Activity的清单文件下)，可以用adb测试是否正常启动`adb shell am start -W -a android.intent.action.VIEW  -d "dante://link" com.your.package`:

```
            <!-- deep link 不能通过直接在浏览器输入网址测试 -->
            <!-- 而是直接在网页源码中加入 <a href="mj://link"></a>  -->
            <intent-filter>
                <action android:name="android.intent.action.VIEW"/>
                <category android:name="android.intent.category.DEFAULT"/>
                <category android:name="android.intent.category.BROWSABLE"/>
                <data android:host="link"
                      android:scheme="dante"/>
            </intent-filter>
```

接受数据（启动activity中）：

```
Uri uri = getIntent().getData();
if (uri != null) {
     String channel = uri.getQueryParameter("channel");
     String data = uri.getQueryParameter("data");

     Log.i(TAG, "test: receive Uri, channel:" + channel + " data: " + data);
}
```

- EditText 的 imeOptions 设置 action 的时候需要跟 inputType合用才有效：
```
               android:imeOptions="actionNext"
                android:inputType="text" 
                android:maxLines="1"
```
- fragment 事务的动画必须写在add/replace前面，否则无效：
```
   getActivity().getSupportFragmentManager().beginTransaction()
                .setCustomAnimations(android.R.anim.slide_in_left, android.R.anim.slide_out_right
                        , android.R.anim.slide_in_left, android.R.anim.slide_out_right)
                .hide(this)
                .add(R.id.container, f)
                .addToBackStack("")
                .commit();
```

- 关于从fragment到 Activity中的fragment，如果在Activity中设置transition，则不会生效（会是默认的fade transition）哪怕你在fragment中调用activity.startPostponeEnterTransition也不行。但是sharedelement是可以正常transition的。如果需要transition，建议直接在fragment中加（记得在onCreate里setTransition而不是onCreateView）【注：目前看起来结论是这样，后期可能会修复，也可能是我自己写法有问题】

- jenkins自动化配置：
——————————————————————————

一.安装jenkins----使用命令行

安装jenkins
$ brew install jenkins

启动jenkins
$ jenkins

卸载jenkins
$ brew uninstall jenkins

如果brew无效，安装homebrew
$ ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)"


二. 打开 http://localhost:8080/

三. JDK配置
$ /usr/libexec/java_home
e.g.	/Library/Java/JavaVirtualMachines/jdk1.8.0_45.jdk/Contents/Home

四. SDK配置
设置——系统设置——Environment variables √
ANDROID_HOME
/Users/yons/Documents/android-sdk-macosx

五. Gradle配置
设置——全局工具配置——Gradle
Name：Gradle，Install automatically √

五. 项目配置
Pass all job parameters as Project properties √

——————————————————————————

- 使AlertDialog的文本可点击：
```
            AlertDialog dialog = new AlertDialog.Builder(this)
                    .setTitle("关于应用")
                    .setMessage("You message...  <a href="http://your.link">点这里</a> blah blah")
                    .setPositiveButton("检查更新", (dialog1, which) -> AppUtil.goMarket(MainActivity.this))
                    .show();
            ((TextView) dialog.findViewById(android.R.id.message)).setMovementMethod(LinkMovementMethod.getInstance());
```

- [如何写一个注解处理器（APT）](http://hannesdorfmann.com/annotation-processing/annotationprocessing101)
- 在 Activity 中显示 Dialog / DialogFragment，不会走 onPause。仅当你的Activity 不在栈（stack）顶时，才会走 onPause。DialogFragment 生命周期是与 Activity 绑定的
