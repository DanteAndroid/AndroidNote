# Android面试题总结

## 事件分发

事件分发就是把触摸事件传递到具体view的整个过程。事件类型包括`MotionEvent.ACTION_DOWN, ACTION_UP, ACTION_MOVE, ACTION_CANCEL`。

从Activity到ViewGroup最终到 View，从上到下依次传递，然后从下到上依次返回true/false，true则代表控件已经消费此事件，false表示不消费此事件。下层返回false的时候，则调用自己的`onTouchEvent`，如果自己也不消费，则继续向上传递。

主要方法有`dispatchTouchEvent`分发事件, `onInterceptTouchEvent`拦截事件, `onTouchEvent` 触摸事件。`onTouch`是TouchListener里的方法主要是用来添加监听的，执行比`onTouchEvent`早；`onTouchEvent`用来作为事件分发的消费，如果实现了`onTouch`返回true，则不会执行`onTouchEvent`。

其中`onInterceptTouchEvent`是在ViewGroup通过`dispatchTouchEvent`分发事件时，都会调用`onInterceptTouchEvent`判断是否拦截事件。通过覆写此方法可以自己处理ViewGroup的触摸事件，返回true则拦截（拦截后，onInterceptTouchEvent不再接受down之后的事件，所有事件分发到`onTouchEvent`），然后通过调用`super.dispatchTouchEvent`走自己这个View的`onTouchEvent`进行事件消费；默认false不拦截，然后遍历子View，如果点击位置有子View则调用其`dispatchTouchEvent`进行事件分发。否则依然走自己这个View的`onTouchEvent`->`performClick`->`onClick`。

或调用ViewGroup的`requestDisallowInterceptTouchEvent()`也可以实现不拦截触摸事件的效果。

View的事件分发从`dispatchTouchEvent`开始，如果view是enable状态，实现了`onTouchListener`并且`onTouch`返回true，则`dispatchTouchEvent`直接返回true；否则依然走`onTouchEvent`->`performClick`->`onClick`。

`dispatchTouchEvent`事件分发时，只有前一个事件返回true，才会收到后一个事件，如果ACTION_DOWN返回了false，后续事件都不会被分发。接受了ACTION_DOWN事件函数也不一定能收到后续事件（`onInterceptTouchEvent`）。

MotionEvent的`getX`和`getRawX`区别：`getX`是触摸点相对于所在控件坐标系的坐标；`getRawX`是相对于屏幕默认坐标系的坐标

## View绘制

view绘制是从View树结构的根节点DecorView开始，逐层自上而下遍历进行测量绘制，也就是measure，layout，draw。

**measure**：就是测量view的大小。`measure`通过由布局参数得到的`MeasureSpec`确定大小，然后会调用`onMeasure`，通过`MeasureSpec`获得view自身宽高。如果是ViewGroup的话，需要遍历测量所有子view的大小，根据布局特性合并处理后得到最终的自身宽高。获得宽高的主要方法是`getDefaultSize`（模式 UNSPECIFIED 时，使用默认值；模式为 AT_MOST 或 EXACTLY时，使用`MeasureSpec`值）。最后通过`setMeasureDimension`存储测量后的宽高。

`MeasureSpec`是int值，高2位存储mode，低30位存储size。**有些情况需要多次测量才能确定最终宽高**

**layout**：就是确定view的位置。`layout`中通过传入的四个顶点的位置，调用`setFrame`方法来设置View自身大小和位置。然后调用`onLayout`确定所有子View的位置。如果自定义ViewGroup需要在`onLayout`中调用所有子View的`layout`来确定子View的位置；如果是单一View，`onLayout`是空实现。

在onLayout中，可以通过`getMeasuredWidth/getMeasuredHeight`获取测量宽高（一般情况与`getWidth/getHeight`值相同，但如果人为设置两者就可能不同，如`super.layout(l, t, r+10, b+10)`）

**draw**：`draw`方法先通过drawBackground绘制背景，然后onDraw绘制内容，dispatchDraw绘制子View，onDrawScrollBars绘制装饰（滚动条，前景色等）

View的`setWillNotDraw`会设置一个跳过`draw`方法的flag，直接走`dispatchDraw`，ViewGroup默认开启此属性

## JVM内存模型

不同cpu的不同线程对同一个变量的缓存值可能不同，为了解决这个问题，引入了内存屏障。它可以阻止指令重排序，强制把缓存区的脏数据写回主内存，让缓存中相应数据失效。`volatile`的作用也是这个（有可见性，不具有原子性）。

**程序计数器**：一块小的内存空间，可以看做当前线程执行字节码的行号指示器（为了线程切换可以恢复到正确执行位置）

**jvm栈**：存储局部变量表，操作栈，动态链接，方法返回地址（加在一起就是栈帧，也就是一次方法调用）

**native方法栈**：与jvm类似，只是为native方法服务

**java堆**：被所有线程共享的内存区域（可以物理不连续），存储绝大部分的对象实例和常量池，也是GC管理的主要区域

**方法区**：与java堆一样是被所有线程共享的内存区域，存储被jvm加载的类、静态变量、JIT编译后的代码等

## HTTPS协议

https是基于http，但是多了SSL也就是安全层。具体包括身份验证（非对称加密RAS/ECC等），传输加密（对称加密AES/DES等），完整性校验（散列算法MD5/SHA）。优点：防止信息被窃听、篡改和冒充。缺点：购买CA证书需要花钱；相比http加密会耗费更多的cpu和内存资源，加载时间会多个几百ms，所以一般传输敏感信息才使用https通信；不能防止服务器攻击，ddos攻击。

SSL/TLS具体的通信过程：client hello中发送客户端支持的加密协议（加密算法和hash算法）-->server hello服务端选择加密协议，并返回CA证书-->客户端检验证书合法性，如果不合法则警告。然后用公钥加密一个随机数，告诉服务端之后的通信用这个随机数来加密-->服务端用私钥解密随机数，告诉客户端已经确认加密方式。握手结束

**非对称加密**：公钥加密，私钥解密；私钥签名（把明文内容做hash处理，然后用私钥把hash加密得到签名），公钥验证签名（把明文内容做hash处理得到hashA，然后用公钥把签名解密得到hashB，对比A和B内容一致则为本人写的，并且没有篡改过）

**证书**：可申请，也可自制。包含颁发机构、公司、域名、公钥、有效期、指纹等

**http2**: 目前只支持https，特点是：多路复用、压缩header、其他的优化

## 类加载机制

TODO

## 加密与编码

**Base64**：把二进制文件内容编码为只包含ASCII码的内容。用于电子邮件、http协议、图片处理

**MD5**：不管长度多大，经过md5都能生成唯一的固定长度值。伪造md5非常困难。用于检查信息传输完整性、数字签名、存放密码，如文件校验

**SHA**（SHA-1）：安全哈希算法，一般用于数字签名。类似于MD5，比MD5安全但是计算更慢

## Kotlin

let返回值是函数的最后一行；also返回的是对象本身；with可以直接调用对象属性，返回值是函数的最后一行；run=let+with，不需要用it代替对象，返回函数最后一行；apply=let+also，可以直接调用对象属性，返回对象本身

sealed class用来表示有限的类继承结构。是枚举类的扩展，但是枚举类常量只有一个实例，而密封类的一个子类可以有多个包含状态的实例

```
sealed class Expression
data class Const(val number: Double) : Expression()
data class Sum(val e1: Expression, val e2: Expression) : Expression()
object NotANumber : Expression()

fun eval(expr: Expression): Double = when (expr) {
    is Const -> expr.number
    is Sum -> eval(expr.e1) + eval(expr.e2)
    NotANumber -> Double.NaN
    // 无需else语句，因为已经覆盖所有情况
}
```

**扩展函数**：编译时，根据类名生成对应的工具类以及public static final的方法，把调用者作为第一个参数传入。如果某个类和其子类都进行同名函数的扩展，使用时，会根据扩展方法的对象类型来决定调用哪个，而不是传入对象的最终类型来调用。成员函数与扩展函数冲突时，总是取成员函数

## Room

Room是轻量级的orm数据库，是对SQLite的再封装，通过注解的方式标记数据库的读写功能，编辑时会语法校验，然后自动生成相关Impl实现类。@Database标记数据库的创建和管理类（继承自RoomDatabase的抽象类），@Entity标记数据库中的表，@DAO标记数据操作的SQL语句

Room的实现从上到下分为三层：抽象接口层（定义增删改查CRUD接口），接口实现层（直接对SQLite进行操作），Room应用层（注解处理器）

## Retrofit

通过Builder模式构建retrofit实例，配置baseUrl、callAdapter、convertFactory，通过retrofit的create(Class<T> service)创建Service的动态代理对象，调用service接口的方法时，就是调用动态代理的invoke方法，然后把我们在接口里配置的各种注解进行解析，构造成ServiceMethod并缓存起来。根据ServiceMethod与接口方法的参数构造成Call对象（默认实现是OkHttpCall），然后通过CallAdapter转换为希望得到的返回对象，通过Call对象的execute或者enqueue进行请求，通过ConvertFactory解析成数据模型。

## HashMap

HashMap是先把Key/Value封装成Node对象，然后通过key的hashCode()函数得出hash值，调用哈希表函数把hash值转换成数组下标，如果该位置没有元素则添加Node到这个位置；如果该位置上有链表，则会用key与链表上面的每个Node的key用equals进行对比，如果没有相同的key则添加到链表末尾，否则覆盖该节点的value。扩容的负载因子为0.75，也就是说，如果数组塞满了75%的对象，则会创建两倍于原来大小的数组，然后把原来的对象进行rehashing，再赋值到新的数组。

注意：作为Key的对象需要实现equals方法，因为equals默认比较的是对象的内存地址。JDK8之后，如果哈希表单向链表中元素超过8个，那么单向链表这种数据结构会变成红黑树数据结构。当红黑树上的节点数量小于6个，会重新把红黑树变成单向链表数据结构。

**为什么负载因子是0.75？** 负载因子是0.75的时候，空间利用率比较高，并且能避免相当多的hash冲突，这样的话链表或者红黑树高度较低。

**SparseArray和HashMap的区别？** SparseArray 底层是两个数组，分别存放key、value。适用于 key 为 int 或者 Long 的情况。特点是免去了 Auto-boxing，所以占用内存较小。但是查找的时候采用了二分法查找，所以数据量大的时候操作效率较低，比如数据量>1000的时候不适合用；HashMap 底层是哈希数组和链表，查找某个元素是根据 key 的 hashCode 来计算数组中的位置，所以效率较高。

## RecyclerView

**绘制**：通过Adapter和四级缓存来创建或者取得缓存的view，然后根据position来进行子view的layout。

**滚动**：通过覆写onInterceptTouchEvent，判断如果是滑动事件，则根据滚动距离调用view.layout来更新所有item的位置，同时对超出屏幕外的item进行回收，对进入屏幕内的item进行创建或从缓存里取得，并通过onBindViewHolder更新view内容。

**Prefetch**：在惯性滚动时，通过Handler来提前从RecycledViewPool中取出即将显示的item，然后放入mCachedViews中，这样当需要对item进行layout时，就可以直接拿来用而不需要绑定数据了

**四级缓存**：

mAttachedScrap：每次layout子view之前，临时存放那些已经添加到recyclerView中的item以及被删除的item。使用场景：滚动过程；可见范围内删除item后调用notifyItemRemoved时

mChangedScrap：存放可见范围内有更新的item。使用场景：可见范围内item更新后调用notifyItemChanged时；临时缓存局部更新，用于播放动画，动画播完viewholder还是会交给recycledViewPool

mCachedViews：存放滚动过程中没有被重新使用且状态无变化的旧item。使用场景：滚动过程；prefetch

RecycledViewPool：缓存item的终点站，用于保存Removed、Changed以及mCachedViews满了以后更旧的item。使用场景：item被移除；item有更新；滚动过程

**与scrollview滑动冲突**：rv的onInterceptTouchEvent方法，判断是否是滑动事件（根据ViewConfiguration拿到touchSlop），判断事件是否在scrollView范围内，如果是，则不拦截

## Flutter

**Flutter**: skia引擎，dart语言，跨平台。原理：三个Tree，Widget负责控件的配置信息，Element负责管理生命周期，RenderObject负责绘制（Configuration、LifeCycle、Canvas）。

**Compose**: skia引擎，kotlin，充分利用平台特性。跨平台版本还是experimental的。

## Glide

TODO

