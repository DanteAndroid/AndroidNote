# KotlinNote
Kotlin 学习笔记

- Kotlin 的接口与 java8 类似，既包含抽象方法的声明，也可包含实现：
```
interface MyInterface {
    fun bar()
    fun foo() {
      // 可选的方法体
    }
}
```
- 扩展函数是由函数调用所在的表达式的类型来决定的， 而不是由表达式运行时求值结果决定的。例如：
```
open class C

class D: C()

fun C.foo() = "c"

fun D.foo() = "d"

fun printFoo(c: C) {
    println(c.foo())
}

printFoo(D())
```
会输出 "c"，因为调用的扩展函数只取决于参数 c 的声明类型，该类型是 C 类。
如果一个类定义有一个成员函数和一个扩展函数，而这两个函数又有相同的接收者类型、相同的名字并且都适用给定的参数，这种情况总是取成员函数。 例如：
```
class C {
    fun foo() { println("member") }
}

fun C.foo() { println("extension") }
```
c.foo() 将输出“member”，而不是“extension”。（显然**成员函数优先于扩展函数**）

- 中缀表示法：标有 infix 关键字的函数可使用中缀表示法（忽略该调用的点与圆括号）调用。中缀函数必须满足以下要求：
它们必须是成员函数或扩展函数；
它们必须只有一个参数；
其参数不得接受可变数量的参数且不能有默认值。
ps.中缀函数调用的优先级低于算术操作符、类型转换以及 rangeTo 操作符。中缀函数调用的优先级高于布尔操作符 && 与 ||、is- 与 in- 检测以及其他一些操作符。
```
infix fun Int.shl(x: Int): Int {
    // ……
}

// 用中缀表示法调用该函数
1 shl 2

// 等同于这样
1.shl(2)
```

- Kotlin 函数都是[头等的](https://zh.wikipedia.org/wiki/%E5%A4%B4%E7%AD%89%E5%87%BD%E6%95%B0)，这意味着它们可以存储在变量与数据结构中、作为参数传递给其他高阶函数以及从其他高阶函数返回。可以像操作任何其他非函数值一样操作函数。**高阶函数**就是将函数用作参数或返回值的函数。

- 把一个对象 解构 成很多变量会很方便，例如:`val (name, age) = person`这种语法称为 解构声明 。一个解构声明同时创建多个变量。
例如从函数中返回两个变量：
```
data class Result(val result: Int, val status: Status)
fun function(……): Result {
    // 各种计算

    return Result(result, status)
}

// 现在，使用该函数：
val (result, status) = function(……)
```
遍历时解构一个映射：
```
for ((key, value) in map) {
   // 使用该 key、value 做些事情
}
```
如果在解构声明中你不需要某个变量，那么可以用下划线取代其名称：`val (_, status) = getResult()`

- 使用`typealias`缩短类型名称，如：`typealias NodeSet = Set<Network.Node>`

- with 接收一个对象和一个扩展函数作为它的参数，然后使这个对象扩展这个函数。这表示所有我们在括号中编写的代码都是作为对象（第一个参数） 的一个扩展函数，我们可以就像作为 this 一样使用所有它的 public 方法和属性。**针对同一个对象做很多操作的时候**非常有利于简化代码。
```
with(helloWorldTextView) {
    text = "Hello World!"
    visibility = View.VISIBLE
}
```

- apply 看起来于 with 很相似，但是是有点不同之处。apply 可以避免创建 builder 的方式来使用，因为对象调用的函数可以根据自己的需要来初始化自己，然后 apply 函数会返回它同一个对象：
```
user = User().apply {
    firstName = Double
    lastName = Thunder
}
```

- 在 Rxjava2 中，使用`Flowable.defer { Flowable.just(doHeavyStuff()) }`使得 doHeavyStuff 可以运行在指定线程，
