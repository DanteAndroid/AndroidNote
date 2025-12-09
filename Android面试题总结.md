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

类加载机制本质上就是 Java 虚拟机把类的 .class 文件加载到内存、并转换为 Class 对象的过程。简单来说可以分为几个阶段：

1. **加载（Loading）**
   - JVM 找到类的二进制数据（通常是 .class 文件），并把它读入内存。
   - 生成一个 Class 对象来表示这个类。
   - 这个阶段主要由 **类加载器**（ClassLoader）完成，有三种主要加载器：
     - **Bootstrap ClassLoader**：加载 JDK 核心类库（如 java.lang.*）。
     - **Extension ClassLoader**：加载 JDK 扩展类库（jre/lib/ext 下的 jar）。
     - **App ClassLoader**：加载应用程序类路径（classpath）下的类。
2. **验证（Verification）**
   - 确保被加载的类符合 JVM 规范，不会破坏虚拟机安全。
   - 包括文件格式验证、元数据验证、字节码验证等。
3. **准备（Preparation）**
   - 为类的静态变量分配内存，并设置默认初始值（如 0、null）。
   - 注意这里只是默认值，还没执行静态代码块或显式赋值。
4. **解析（Resolution）**
   - 将符号引用（如类名、方法名）替换为直接引用（内存地址）。
   - 也可以理解为把代码里的符号标识和内存对象对应起来。
5. **初始化（Initialization）**
   - 执行类的静态变量显式赋值和静态代码块（static {}）。
   - 这个阶段是真正开始类的“运行”的阶段。
6. **使用和卸载**
   - 类加载后就可以实例化对象、调用方法了。
   - 如果类不再使用，且没有任何引用，它会被 GC 卸载（卸载条件严格，通常只针对自定义类加载器加载的类）。

简单理解就是：**加载 → 验证 → 准备 → 解析 → 初始化 → 使用/卸载**。

## 加密与编码

**Base64**：把二进制文件内容编码为只包含ASCII码的内容。用于电子邮件、http协议、图片处理

**MD5**：不管长度多大，经过md5都能生成唯一的固定长度值。伪造md5非常困难。用于检查信息传输完整性、数字签名、存放密码，如文件校验

**SHA**（SHA-1）：安全哈希算法，一般用于数字签名。类似于MD5，比MD5安全但是计算更慢

## Kotlin

let返回值是函数的最后一行；also返回的是对象本身；with可以直接调用对象属性，返回值是函数的最后一行；run=let+with，不需要用it代替对象，返回函数最后一行；apply=let+also，可以直接调用对象属性，返回对象本身

sealed class用来表示有限的类继承结构。是枚举类的扩展，但是枚举类常量只有一个实例，而密封类的一个子类可以有多个包含状态的实例

**扩展函数**：编译时，根据类名生成对应的工具类以及public static final的方法，把调用者作为第一个参数传入。如果某个类和其子类都进行同名函数的扩展，使用时，会根据扩展方法的对象类型来决定调用哪个，而不是传入对象的最终类型来调用。成员函数与扩展函数冲突时，总是取成员函数

## Room

Room是轻量级的orm数据库，是对SQLite的再封装，通过注解的方式标记数据库的读写功能，编辑时会语法校验，然后自动生成相关Impl实现类。@Database标记数据库的创建和管理类（继承自RoomDatabase的抽象类），@Entity标记数据库中的表，@DAO标记数据操作的SQL语句

Room的实现从上到下分为三层：抽象接口层（定义增删改查CRUD接口），接口实现层（直接对SQLite进行操作），Room应用层（注解处理器）

## Retrofit

通过Builder模式构建retrofit实例，配置baseUrl、callAdapter、convertFactory，通过retrofit的create(Class<T> service)创建Service的动态代理对象，调用service接口的方法时，就是调用动态代理的invoke方法，然后把我们在接口里配置的各种注解进行解析，构造成ServiceMethod并缓存起来。根据ServiceMethod与接口方法的参数构造成Call对象（默认实现是OkHttpCall），然后通过CallAdapter转换为希望得到的返回对象，通过Call对象的execute或者enqueue进行请求，通过ConvertFactory解析成数据模型。

## HashMap

HashMap是先把Key/Value封装成Node对象，然后通过key的hashCode()进行hash()散列函数得出hash值，通过indexFor转成数组下标，如果该位置没有元素则添加Node到这个位置；如果该位置上有链表，则会用key与链表上面的每个Node的key用equals进行对比，如果没有相同的key则添加到链表末尾，否则覆盖该节点的value。扩容的负载因子为0.75，也就是说，如果数组塞满了75%的对象，则会创建两倍于原来大小的数组，然后把原来的对象进行rehashing，再赋值到新的数组。

    // length是数组容量
    static int indexFor(int h, int length) {
        return h & (length-1);
    }


注意：作为Key的对象需要实现equals方法，因为equals默认比较的是对象的内存地址。JDK8之后，如果哈希表单向链表中元素超过8个，那么单向链表这种数据结构会变成红黑树数据结构。当红黑树上的节点数量小于6个，会重新把红黑树变成单向链表数据结构。

**为什么负载因子是0.75？** 负载因子是0.75的时候，空间利用率比较高，并且能避免相当多的hash冲突，这样的话链表或者红黑树高度较低。

**SparseArray和HashMap的区别？** SparseArray 底层是两个数组，分别存放key、value。适用于 key 为 int 或者 Long 的情况。特点是免去了 Auto-boxing，所以占用内存较小。但是查找的时候采用了二分法查找，所以数据量大的时候操作效率较低，比如数据量>1000的时候不适合用；HashMap 底层是哈希数组和链表，查找某个元素是根据 key 的 hashCode 来计算数组中的位置，所以效率较高。

## RecyclerView

### 绘制流程

- 绘制子 View 由 **LayoutManager** 控制。
- LayoutManager 先计算可见 item 的位置，然后通过 Recycler.getViewForPosition(position) 获取 View：
  - 先从 mAttachedScrap 查找；
  - 再从 mCachedViews 查找；
  - 再从 RecycledViewPool 获取或新建。
- 如果获取的 view 是新 view 或需要更新数据，则调用 onBindViewHolder 绑定数据。

### 滚动机制

- 滚动事件由 onInterceptTouchEvent 判断是否拦截（依据滑动方向和 touchSlop）。
- LayoutManager 处理实际滚动，通过 scrollBy/scrollVerticallyBy 调整可见 item 位置，并决定：
  - 超出屏幕的 item 回收；
  - 新进入屏幕的 item 创建或复用，并绑定数据。

### 预取机制（Prefetch）

- LayoutManager 会在惯性滚动时提前计算即将显示的 item，并调用 Recycler 预取 view，减少滚动卡顿。
- 预取的 item 可先加入缓存（mCachedViews），滚动到屏幕时直接复用，无需重新绑定。

### RecyclerView 四级缓存

1. **mAttachedScrap**：可见范围内被临时移除的 item，用于重用，布局完成后清空。
2. **mChangedScrap**：可见范围内需要局部更新的 item（例如 notifyItemChanged），动画结束后可能回收。
3. **mCachedViews**：滚动过程中未被使用、状态未变化的旧 item，用于快速复用。
4. **RecycledViewPool**：缓存最终站，可跨 RecyclerView 共享，用于存放回收或更新后的 item，减少 view 创建开销。

### 与 ScrollView 滑动冲突

- RecyclerView 的 onInterceptTouchEvent 判断是否拦截滑动。
- 当嵌套在 ScrollView 内时，通常通过 requestDisallowInterceptTouchEvent(true) 让 RecyclerView 拦截滑动事件，避免冲突。

## Flutter

**Flutter**: skia引擎，dart语言，跨平台。原理：三个Tree，Widget负责控件的配置信息，Element负责管理生命周期，RenderObject负责绘制（Configuration、LifeCycle、Canvas）。

**Compose**: skia引擎，kotlin，充分利用平台特性

## Glide

Glide 在加载图片时，会按顺序依次检查以下缓存：

1. **活动资源（Active Resources）**

   - 检查当前是否有其他 View 正在使用这张图片。
   - 如果有，直接返回正在使用的 Bitmap/Drawable，避免重复解码。

2. **内存缓存（Memory Cache）**
   - LRU Cache，存储最近加载过且未被回收的图片。
   - 命中则立即返回，保证快速显示。

3. **磁盘缓存 - 已解码资源（Resource Cache）**

   - 检查之前是否已解码、转换并写入磁盘缓存的 Bitmap/Drawable。
   - 异步加载到内存后返回，避免重复解码和转换。

4. **磁盘缓存 - 原始数据（Data Cache）**
   - 检查原始图片文件是否已写入磁盘缓存（通常是下载的网络图片）。
   - 异步解码后返回，可以避免每次都重新从网络下载。

5. **原始资源加载（Source）**

   - 如果以上步骤都未命中，则回退到原始来源：网络、资源文件、Uri 等。
   - 下载/读取后，再写入缓存，供下次使用。




   ## **Coil 核心流程**

1. **请求发起**

```
imageView.load(url)
```

创建 ImageRequest，包含数据源、协程上下文、缓存策略、占位图和转换器等。

1. **检查缓存**

   先查内存缓存（LRU 存储已解码的 Bitmap/Drawable），命中直接返回；再查磁盘缓存（基于 OkHttp 存储原始数据），异步解码后写入内存缓存。Coil 没有 Glide 那种多层缓存和 BitmapPool。

2. **协程异步加载**

   网络下载、文件读取、解码都在 Dispatcher.IO 或自定义调度器中执行。支持挂起函数，Activity/Fragment 销毁时自动取消，避免内存泄漏或无效加载。

3. **解码与转换**

   使用 Decoder 将原始数据解码为 Bitmap/Drawable，可做圆角、模糊等 Transformation，加载完成写入内存缓存。

4. **显示到 ImageView**

   切换到主线程显示图片，协程保证异步安全，不阻塞 UI。   

| **特性**     | **Glide**                                  | **Coil**                           |
| ------------ | ------------------------------------------ | ---------------------------------- |
| 异步机制     | 线程池                                     | 协程挂起函数                       |
| 内存复用     | BitmapPool                                 | 系统管理，不手动复用               |
| 缓存层级     | Active → Memory → Resource → Data → Source | Memory → Disk → Source             |
| API 风格     | Java/Kotlin 链式                           | Kotlin DSL 风格，更轻量            |
| 生命周期管理 | RequestManager 感知生命周期                | 协程自动绑定生命周期，自动取消请求 |

**Coil 核心在于用协程安全、轻量、现代化地异步加载图片，同时利用内存和 OkHttp 缓存减少重复解码和网络请求，而不依赖 Glide 那种复杂的多层缓存和 BitmapPool。**

   

## 每个App都是单独的虚拟机？

每个 App 都运行在自己独立的进程中，并且在这个进程中会启动一个专用的运行时环境（Runtime），你可以将其理解为“虚拟机”或“运行时”。

这个运行时环境在不同的版本中有不同的名字：

### Android 运行时环境 (ART & Dalvik)

1. Android 5.0 之后（主流）：ART

   从 Android 5.0 (Lollipop) 开始，ART (Android Runtime) 彻底取代了 Dalvik，成为了默认的运行时环境。

   - 每个 App 进程 都会创建一个 ART 实例 来执行它的代码。
   - ART 实现了 AOT (Ahead-Of-Time) 编译，在 App 安装时就把 DEX 字节码预先编译成机器码，大大提高了 App 的启动和运行速度。
   - **独立性： 每个 App 依然拥有一个独立的 ART 实例和独立的进程，确保了 App 之间的隔离性、安全性和稳定性。一个 App 崩溃不会影响其他 App。**

2. Android 5.0 之前：Dalvik

   在 Android 5.0 之前，App 运行在 Dalvik 虚拟机 (DVM) 中。

   - 同样，每个 App 进程 都有一个 DVM 实例。
   - DVM 采用 JIT (Just-In-Time) 编译，在 App 运行时即时编译字节码。

无论是 Dalvik 还是 ART，它们的核心设计哲学都是：

- **进程隔离：** 每个 App 运行在一个独立的 Linux 进程中。
- **沙箱机制 (Sandbox)：** App 之间的文件、内存和资源是相互隔离的，没有授权不能互相访问，这就像把每个 App 放在一个“沙箱”里运行。

所以，虽然严格来说 ART/Dalvik 不是传统的 Java 虚拟机 (JVM)，但它们在 Android 中起到了类似的作用：为每个 App 提供一个独立、安全、专用的代码执行环境。你可以形象地将这个“独立的进程 + 独立的 ART/Dalvik 实例” 理解为“每个 App 一个虚拟机”



## 协程机制

协程的核心机制本质上是**轻量级的线程与状态机结合**，它通过挂起与恢复来实现异步或并发执行，而不占用系统线程资源。可以从以下几个角度来理解：

---

### 1. **挂起与恢复**
- 协程中的 `suspend` 函数可以挂起当前协程而不阻塞线程。
- 挂起时，协程的执行状态（包括局部变量、调用栈等）会被保存到一个对象中，这就是 **Continuation（续体）**。
- 恢复时，Continuation 会记录上一次挂起的位置，直接从挂起点继续执行。

### 2. **Continuation接口**

```
interface Continuation<in T> {
    val context: CoroutineContext
    fun resumeWith(result: Result<T>)
}
```
Continuation 会保存

- 局部变量：挂起时的局部变量会被存储到 Continuation 的字段里。
- label：表示下次恢复的状态。
- context：协程上下文，用于调度和异常处理。

因此，调用 resumeWith(result) 时，状态机会检查 label，然后执行对应代码块，并把之前保存的局部变量恢复到作用域中。

### 3. **状态机实现**
- Kotlin 编译器会将 `suspend` 函数编译成状态机。
- 每个挂起点对应一个状态（label），执行时根据状态跳转。
- 这就是为什么协程可以在一个线程里多次挂起和恢复而不会丢失状态。

### **线程池在协程中的作用**

Kotlin 协程提供了 **CoroutineDispatcher**，它决定协程在哪个线程或线程池上执行。

常用的调度器：

- Dispatchers.Default：底层是**共享的通用线程池**，适合 CPU 密集型任务。
- Dispatchers.IO：底层也是线程池，但线程数量可伸缩，适合 IO 密集型任务。
- Dispatchers.Main：主线程（UI 线程），没有线程池。
- 自定义 newSingleThreadContext 或 Executor.asCoroutineDispatcher() 可以绑定自定义线程池。

协程调度器会把协程任务提交给线程池执行，但协程本身只是一个**任务单元**，不会直接创建线程。

### 为什么协程挂起不占用线程

因为协程是状态机+Continuation
- 挂起函数立即返回，线程可以继续执行其他任务。
- 当协程需要恢复时，调度器调用 Continuation 的 resume 方法，从上次挂起的位置继续执行。



## Compose 核心机制

**Composable 函数**

- 不同于传统 View 系统，Compose 不直接操作 View 层，而是通过 **函数声明 UI**，根据状态生成 UI。
- UI 的描述是函数的结果，状态改变时，函数可以被重新调用生成新的 UI。

**Compose Compiler**

- @Composable 函数在编译期由 **Compose Compiler** 转换：
  - 增加 Composer 参数，用于管理 SlotTable 和重组。
  - 增加 changed 位掩码，用于快速判断参数是否变化。
  - 插入 startRestartGroup / endRestartGroup 标记重组边界。
- 优化点：
  - 未变化参数跳过重组。
  - Lambda 缓存避免重复对象。
  - Inline 提高性能。

**重组（Recomposition）**

- 是 Compose 响应状态变化重新生成 UI 的过程。
- 当 **State** 发生变化时，Compose 会标记依赖这个 State 的 Composable 为 **脏（dirty）**。
- Compose 在下一次重组时，只会重新执行这些脏的 Composable，而不是整个 UI 树。
- **核心数据结构**：
  - **SlotTable**：Compose 内部存储函数调用和 UI 层次的表格，用于跟踪 Composable 的位置、状态和子树。
  - **Recomposer**：负责调度脏的 Composable 执行重组。

**状态管理（State）**

- **State** 是 Compose 响应式 UI 的基础。
- Compose 提供：
  - mutableStateOf
  - remember / rememberSaveable
  - derivedStateOf
- **原理**：
  - State 是可观察对象，当其值改变时，会通知依赖它的 Composable 标记为脏。
  - Compose 内部使用 **Snapshot** 来追踪状态变化。
- **Snapshot** 用于在多线程环境下安全管理状态。
- 核心概念：
  - **MutableSnapshot**：每个线程可以有自己的快照，读写隔离。
  - **Global Snapshot**：用于跨线程合并状态。
- Compose 使用快照实现 **一致性视图**，即在重组期间 UI 可以安全读取状态，而不会被同时修改破坏。


**怎么实现自动追踪状态的**：
Compose 的 state-driven 本质类似于“重写 get / set”，但是它比普通的 setter 通知机制更复杂和高效：
	1.	get
	•	读取状态时，Compose 会收集依赖，记录当前 Composable 与这个状态的关系。
	•	这个过程在内部通过 slot table + snapshot 实现，不是普通的变量访问。
	2.	set
	•	修改状态时，会触发通知观察者（依赖的 Composable）。
	•	Compose 会标记这些 Composable 为“脏”，等待下一帧进行局部重组。
	•	不会直接更新 UI，而是通过 重组（Recomposition） 在安全的 UI 调度时更新界面。

所以核心确实是 “读收集依赖 + 写通知 + 重组更新”，而不是像传统监听器那样主动刷新界面。

**重组优化**

- Compose 会尽量减少重组开销：

  - 仅脏 Composable 会重组。
  - remember 用于缓存值避免重复计算。
  - derivedStateOf 用于避免不必要的重组。
  - Compose 会维护函数调用的 **SlotTable**，通过位置和 key 来识别 Composable 是否需要重新执行。

**布局与绘制（Layout & Draw）**

- Compose 的布局与绘制机制类似于传统 View，但完全在 Compose 内部实现：
  1. **Measure**：测量子 Composable 尺寸。
  2. **Layout**：确定每个子 Composable 的位置。
  3. **Draw**：绘制到 Canvas 上。
- Compose 使用 **布局修饰符（Modifier）链式调用**，在布局、绘制、触摸事件处理等环节灵活扩展。

**协程与异步**

- Compose 与 **协程高度集成**：
  - LaunchedEffect、rememberCoroutineScope 等用于在 Composable 内安全执行异步操作。
  - 状态变化在协程内更新时，会自动触发重组。

**总结核心流程**

1. Composable 函数执行生成 **UI 描述树（SlotTable）**。
2. State 发生变化 → Recomposer 标记脏节点。
3. Recomposer 调度脏节点重组 → Composable 函数重新执行。
4. 生成新的 UI 描述 → Compose 将差异应用到实际绘制上。
5. 通过 Snapshot 保证状态一致性，Modifier 控制布局、绘制、交互。

