Also checkout:
- ### [Kotlin Note](/KotlinNote.md)
- ### [Git Note](/GitNote.md)
- ### [多渠道](/多渠道版本发布.md)

# Android Note

- [性能优化](https://www.jianshu.com/p/7d31399b98c7) & [布局优化](https://www.jianshu.com/p/4e665e96b590)
### TextView的任意文本变为链接
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
- 要用styles。几乎每个项目都得合适地使用styles，因为对于view来说有重复的外观是很常见的。至少你得给app的大部分文本内容定义一个通用样式：
- 你可以把一个大的style文件分割成几个文件。`styles.xml`的文件名没啥神奇的，重要的是文件里的`<style>`标签。因此你可以有style_home.xml, styles_item_details.xml之类的。不像build系统下的其他文件夹名，values下面的文件名可以很随意。
- AS编译R文件丢失，常见原因是: 1. xml有错误 2. res下面的资源文件格式有问题(如，jpeg的图片用的却是png的后缀)
### Transition转场动画
      0. EnterTransition顾名思义进入activity时的动画，在进入的activity的onCreate中设置，ReenterTransition即从之前打开的activity返回来时的动画（第二次回到这个activity）。这两个动画都是进入动画，即现在打开这个activity时的动画; ReturnTransition返回动画，即窗口关闭时（如在此activity时点击back时）呈现的动画。ExitTransition退出动画，在此activity中跳转到新的activity时的动画，需要跟returnTransition做区别。这两个动画都是离开动画，即现在离开这个activity的动画；setAllowEnterTransitionOverlap是否允许第一次进入这个activity时的动画(EnterTransition)覆盖。setAllowReturnTransitionOverlap是否允许返回这个activity时的动画(ReenterTransition)覆盖，如果是，那么返回这个activity的动画立刻执行。
      1.Return和Reenter Transitions分别是Enter和Exit的反向动画（如果没有特别设置的话）
      1. xml中设置：在activity的主题中添加<item name="android:windowXXXTransition">@transition/CustomTransitionXML</item>其中自自定义的transition文件在res/transition下面，根元素是transitionSet
      2. 代码中设置： getWindow().setXXXTransition(transition);这里的transition对象可以直接new。有Slide, Explode, Fade三种默认的动画。
      3. Transition默认会应用到View树的所有view上，但是用`addTarget()`可以单独为某些view应用动画。      

- animator写动画效果时，遇到`java.lang.IllegalStateException: Already started!`错误，是因为没有setListen()。因为之前已经使用了`animate()`方法并且setListener了，所以这个错误是由于用的是之前的listener对象造成的。
- 调试技巧：按下home，然后在terminal中输入adb shell am kill com.your.packagename即可模拟杀掉后台
- [用Handler替代Timer](http://www.mopri.de/2010/timertask-bad-do-it-the-android-way-use-a-handler/)
- 在新的task中打开activity：M1. launchMode="singleTask" + android:taskAffinity="new.package.name" 在启动一个singleTask的Activity实例时，如果系统中已经存在这样一个实例，就会将这个实例调度到任务栈的栈顶，并清除它当前所在任务中位于它上面的所有的activity。 M2. `intent.setFlags(Intent.FLAG_ACTIVITY_NEW_TASK);` + android:taskAffinity="new.package.name"
-   以singleInstance模式启动的Activity具有全局唯一性，即整个系统中只会存在一个这样的实例
    以singleInstance模式启动的Activity具有独占性，即它会独自占用一个任务，被他开启的任何activity都会运行在其他任务中（官方文档上的描述为，      singleInstance模式的Activity不允许其他Activity和它共存在一个任务中）
    被singleInstance模式的Activity开启的其他activity，能够开启一个新任务，但不一定开启新的任务，也可能在已有的一个任务中开启

### Deep Link的配置
(添加到启动Activity的清单文件下)，可以用adb测试是否正常启动`adb shell am start -W -a android.intent.action.VIEW  -d "dante://link" com.your.package`:

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

### jenkins自动化配置：

```
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
```

### 双缓冲机制
 
问题的由来
CPU访问内存的速度要远远快于访问屏幕的速度。如果需要绘制大量复杂的图像时，每次都一个个从内存中读取图形然后绘制到屏幕就会造成多次地访问屏幕，从而导致效率很低。这就跟CPU和内存之间还需要有三级缓存一样，需要提高效率。

第一层缓冲
在绘制图像时不用上述一个一个绘制的方案，而采用先在内存中将所有的图像都绘制到一个Bitmap对象上，然后一次性将内存中的Bitmap绘制到屏幕，从而提高绘制的效率。Android中View的onDraw()方法已经实现了这一层缓冲。onDraw()方法中不是绘制一点显示一点，而是都绘制完后一次性显示到屏幕。

第二层缓冲
onDraw()方法的Canvas对象是和屏幕关联的，而onDraw()方法是运行在UI线程中的，如果要绘制的图像过于复杂，则有可能导致应用程序卡顿，甚至ANR。因此我们可以先创建一个临时的Canvas对象，将图像都绘制到这个临时的Canvas对象中，绘制完成之后再将这个临时Canvas对象中的内容(也就是一个Bitmap)，通过drawBitmap()方法绘制到onDraw()方法中的canvas对象中。这样的话就相当于是一个Bitmap的拷贝过程，比直接绘制效率要高，可以减少对UI线程的阻塞。

### Gradle currently uased 问题

find ~/.gradle -type f -name "*.lock" | while read f; do rm $f; done

find /Users/l/Documents/RRTrunk/.gradle -type f -name "*.lock" | while read f; do rm $f; done

### 刷新DNS

sudo killall -HUP mDNSResponder

### AS搜索时排除生成类

find usage点击设置图标，点击三个点新增scope，输入规则：`!file:*intermediates*/&&!file:*generated*/&&!lib:*..*`

## ADB

`adb tcpip 5555` --> `adb connect 192.168.*.*(设置-关于手机里面的ip)`

`adb logcat -c` 清空日志

`adb logcat -t 500` > dante.log 输出最近500行日志并保存到项目目录下的 dante.log 文件

### SurfaceView 

- **WMS中单独创建窗口**
- 可以在子线程刷新画面
- 双缓冲机制（更新视图时用了两张Canvas）
- 不可进行旋转缩放等动画
- 适合2D游戏和视频播放器开发

### TextureView

- **作为View hierachy中的普通View**
- 单独的渲染线程
- 必须在硬件加速的窗口中
- 适合视频播放器或者相机开发
- 内存占用比SurfaceView高

### 事件分发

 - 对于事件分发：（dispatchTouchEvent）
 **如果想事件不向下传递，自己消费掉**  ：将当前的dispatchTouchEvent返回true；
**如果想事件不向下传递，返回给上层**  ：将当前的dispatchTouchEvent返回false；
 - 对于事件拦截：（onInterceptTouchEvent）
 **如果想拦截事件，给自己的onTouchEvent方法消费** ：将onInterceptTouchEvent返回true
**如果不拦截事件，默认向下（子view）传递** ：将onInterceptTouchEvent返回false或者返回默认值
 - 对于事件消费：（onTouchEvent）
 **如果不想消费，返回给上层** ：将onTouchEvent返回默认或者返回false；
**如果想消费，不再返回** ：将onTouchEvent返回true；
 - 当`dispatchTouchEvent（）`事件分发时，只有前一个事件（如ACTION_DOWN）返回true，才会收到后一个事件（ACTION_MOVE和ACTION_UP）
 - onInterceptTouchEvent 并不能消费事件，它相当于是一个分叉口起到分流导流的作用，对后续的ACTION_MOVE和ACTION_UP事件接收起到非常大的作用
 - 接收了ACTION_DOWN事件的函数不一定能收到后续事件

### GestureDetector

使用 SimpleOnGestureListener 时注意要 onDown 返回 true，否则不会接管触摸事件

```
            root.setOnTouchListener(object : View.OnTouchListener {
                private val detector = GestureDetectorCompat(context,
                    object : GestureDetector.SimpleOnGestureListener() {
                        override fun onSingleTapConfirmed(e: MotionEvent?): Boolean {
                            if (doSomething()) return false
                            return true
                        }

                        override fun onDoubleTap(e: MotionEvent?): Boolean {
                            doSomeThingElse()
                            return true
                        }
												
                        override fun onDown(e: MotionEvent?): Boolean = true
                    })

                override fun onTouch(v: View?, event: MotionEvent?): Boolean =
                    detector.onTouchEvent(event)
            })
```

###  其他技巧

- sdk 初始化失败，可能是初始化所在的进程不对
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

- 在实体对象中，新增了属性，也正确赋值了，但是最后拿到的值却是默认值：调试发现，是通过 Intent 传过去后，值发生了变化。原来此对象实现了`Parcelable`，新增属性后需要也在相关方法中添加对应该属性的方法，否则肯定无法通过 Intent 传输

- 在 shape 中，使用带有透明度的颜色 + corners 时，如果 corners 的值小于 stroke 边框的宽度的一半，会在拐角出出现重叠的现象。解决办法：调高 corners 值；或者禁用硬件加速：`view.setLayerType(View.LAYER_TYPE_SOFTWARE, null)`

- ```
  <?xml version="1.0" encoding="utf-8"?>
  <shape xmlns:android="http://schemas.android.com/apk/res/android"
         android:shape="rectangle">
      <stroke
          android:dashGap="10dp"
          android:dashWidth="60dp"
          android:width="8dp"
          android:color="@color/frame"/>
      <corners android:radius="5dp"/>
  
  </shape>
  ```

- 接近矩形的小圆点（不是椭圆），也可以用上一条 Shape + Corner 的方式做

- ScaleX 和 ScaleY 只影响 View 的绘制（通过对原始矩阵变换后成为新矩阵），并不影响其宽高。其点击事件区域的变换是由于绘制时，原始矩阵和新矩阵的点进行了 mappoint

- 动态的布局和 DialogFragment 中使用 ConstraintLayout 在部分机型会出现布局很奇怪的情况。

- 分享图片到微信时，MIMEType 设为 image/* 会偶尔出现无法分享的问题，改为 `*/*` 后正常：

  ```
          Intent shareIntent = new Intent();
          shareIntent.setAction(Intent.ACTION_SEND_MULTIPLE);
          ArrayList<Uri> files = new ArrayList<>();
          for (String path : filePaths) {
              File file = new File(path);
              Uri uri = Uri.fromFile(file);
              files.add(uri);
          }
          shareIntent.putParcelableArrayListExtra(Intent.EXTRA_STREAM, files);
          shareIntent.setType("*/*");
  ```

- EditText 的 imeOptions 设置 action 的时候需要跟 inputType合用才有效：
```
               android:imeOptions="actionNext"
                android:inputType="text" 
                android:maxLines="1"
```

- 关于从fragment到 Activity中的fragment，如果在Activity中设置transition，则不会生效（会是默认的fade transition）哪怕你在fragment中调用activity.startPostponeEnterTransition也不行。但是sharedelement是可以正常transition的。如果需要transition，建议直接在fragment中加（记得在onCreate里setTransition而不是onCreateView）【注：目前看起来结论是这样，后期可能会修复，也可能是我自己写法有问题】
