---
marp: true
theme: default
paginate: true
_class: lead
---

# Kotlin 函数

@Nano

Github: https://github.com/NanoMichael

---

# 函数基本结构

Kotlin 中函数使用 `fun` 关键字声明

```kotlin
fun dobule(x: Int): Int {
  return 2 * x
}
// 或者单行函数
fun dobule(x: Int) = 2 * x

fun test() {
  val doubleTwo = double(2)
  println("double(2) = $doubleTwo")
}
```

---

# 函数作用域

在 Kotlin 中函数可以在文件顶层声明，这意味着你不需要像一些语言如 Java、C# 或 Scala 那样创建一个类来保存一个函数。此外除了顶层函数，Kotlin 中函数也可以声明在局部作用域、作为成员函数以及扩展函数。

- 成员函数：略

---

## 局部函数（嵌套函数）

Kotlin 支持局部函数，即一个函数可在另一个函数内部

```kotlin
fun doSomething() {
  val delta = 2
  fun add(a: Int) = a + delta
  println("${add(1)}") // 输出 3
  println("${add(3)}") // 输出 5
}
```

局部函数可以访问外部函数（即闭包）中的局部变量delta。

---

# 扩展函数

通过扩展声明完成一个类的新功能扩展，而无需继承该类

例：反转一个列表

```kotlin
fun <T> List<T>.reversed(): List<T> {
  val list = mutableListOf()
  val iterator = listIterator()
  while (iterator.hasPrevious()) {
    list.add(iterator.previous())
  }
  return list
}
```

---

# 高阶函数

高阶函数是将函数用作参数或返回值的函数。

例：筛选列表中符合条件的元素

```Kotlin
inline fun <T> Iterable<T>.filter(crossinline predicate: (T) -> Boolean): List<T> {
  val list = mutableListOf()
  for (elem in this) {
    if (predicate(elem)) list.add(elem)
  }
  return list
}
```

---

它的输入参数 `predicate: (T) -> Boolean` 就是一个函数。其中，函数类型声明的语法是：

```Kotlin
(X) -> Y
```

表示这个函数是从类型 `X` 到类型 `Y` 的映射。我们可以这么调用：

```kotlin
fun isOdd(x: Int) = x % 2 == 1
val list = listOf(1, 2, 3, 4)
list.filter(::isOdd)
```

**::** 用来引用一个函数

---

函数也可作为返回值：

```Kotlin
fun isOddFun(x: Int): () -> Boolean {
  fun xIsOdd(): Boolean {
    return x % 2 == 1
  }
  return ::xIsOdd
}
```

调用：

```Kotlin
val f = isOddFun(3)
val `3 is odd` = f()
```

---

# 匿名函数和 Lambda

我们也可以使用匿名函数来实现 `predicate`：

```Kotlin
val list = listOf(1, 2, 3)
list.filter((fun(x: Int): Boolean {
  return x % 2 == 1
}))
```

当然，lambda 是更好的方式

```Kotlin
list.filter({ x -> x % 2 == 1})
```

---

- lambda 的返回值为最后一条表达式的值
- 如果 lambda 是该函数调用中的最后一个参数，则圆括号可以省略：
  
  ```Kotlin
  list.filter { x -> x % 2 == 1 }
  ```

- 如果函数仅接受一个参数，则参数可被省略，使用 `it` 替代：

  ```Kotlin
  list.filter { it % 2 == 1 }
  ```

- lambda 可作为变量使用，形如：

  ```Kotlin
  val isOdd = { x: Int -> x % 2 == 1 }
  ```

---

改写一下筛选列表中符合条件的元素：

```Kotlin
inline fun <T> Iterable<T>.filter(predicate: (T) -> Boolean) =
  fold(mutableListOf()) { acc, elem ->
    acc.apply { if (predicate) add(elem) }
  }
```

注意几个 `this`

- `fold` 函数的 this 是 Iterable<T>
- `add` 函数的 this 是 acc

---

## 一个例子：柯里化函数

[柯里化](https://en.wikipedia.org/wiki/Currying)将需要接受多个参数的函数转化为**一次只接受一个参数**的函数。

```Kotlin
fun <T1, T2, R> ((T1, T2) -> R).currying = { t1: T1 -> { t2: T2 -> this(t1, t2) } }

fun <T1, T2, R> ((T1) -> (T2) -> R).uncurrying() = { t1: T1, t2: T2 -> this(t1)(t2) }

val sum = { x: Int, y: Int -> x + y }

val sumCurrying = sum.currying() // (Int) -> (Int) -> Int

val `2 add` = sumCurrying(2) // (Int) -> Int

val `sum of 2 and 3 is 5` = `2 add`(3) // Int

val sumUncurrying = sumCurrying.uncurrying() // (Int, Int) -> Int
```

---

# 中缀函数

在以下场景中，函数还可以用中缀表示法调用：

- 成员函数或扩展函数
- 只有一个参数
- 用 `infix` 关键字标注

---

检查一个元素是否在集合中:

```Kotlin
inline infix fun <T> T.isElem(list: List<T>) = this in list
// 调用
val list = listOf(1, 2, 3)
val `1 is element of the list` = 1 isElem list
// 当然我们可以直接使用 in 操作符
```

输出 0，2，4，6，8

```Kotlin
(0 until 10 step 2).foreach { print("$it, ") }
```

---

## 一个例子：函数组合

假设有两个函数，`f` 和 `g`：

```Haskell
f: a -> b
g: b -> c
```

可将函数 `f` 和 `g` 组合为一个新的函数 `h`，直接将 `a` 映射为 `c`，记作 `h = g . x`

```Kotlin
infix fun <A, B, C> ((B) -> C).`&`(f: (A) -> B) = { a: A -> this(f(a)) }

val double2 = { x: Int -> x * 2 } // (Int) -> Int
val div3 = { x: Int -> x.toFloat() / 3f } // (Int) -> Float

val composed = div3 `&` double2 // (Int) -> Float
println("${composed(3)}") // 2.0
```

---

# [inline, noinline, crossinline](https://kotlinlang.org/docs/reference/inline-functions.html)

实际上，如果没有特殊处理，创建一个（lambda）函数需要创建一个 `Function`，如果频繁调用高阶函数将导致内存浪费。`inline` 函数（内联函数）则不会创建新的 `Function` 对象，而是将函数内需要执行的语句拷贝到执行处。

```Kotlin
inline fun inlined(f: () -> Unit) {
  println("before")
  f()
  println("after")
}

fun test() {
  inlined { print("do something here") }
}
```

---

上述代码经过反编译之后看起来像这样：

```Kotlin
void test() {
  System.out.println("before");
  System.out.println("do something here");
  System.out.println("after");
}
```

因此， 在 `inline` 函数内 `return` 将返回调用函数；在封装类内，公开的 `inline` 函数不能访问 `private` 成员。

`noinline` 使用在有多个函数参数的 `inline` 函数内，用以表示这个函数不会被拷贝。

```Kotlin
inline fun noinlined(f1: () -> Unit, noinline f2: () -> Unit) {
  f1()
  f2()
}
```

---

## Non-local returns（非局部返回）

如果我们要从一个 lambda 函数中返回，则必须要使用标签标注

```Kotlin
fun foo() {
  ordinaryFunction label@{
    return       // ERROR 不能直接返回 foo
    return@label // 返回 ordinaryFunction
  }
}
```

而在 `inline` 函数中则直接返回调用函数

```Kotlin
fun foo() {
  inlined { return }
}
```

---

如果 `inline` 函数不直接调用其参数函数，而是在其他上下文中执行，则 `non-local returns` 也是不允许的，这种情况下应使用 `crossinline` 标注

```Kotlin
inline bar(crossinline block: () -> Unit) {
  val f = { block() }
  f()
}

fun testBar() {
  foo {
    return // ERROR
    return@foo // OK
  }
}
```

---

## reified（具体化类型参数）

有个时候我们需要检查参数的类型，例如找到树的特定类型的父节点，以 `Java` 的写法，大概是：

```Kotlin
fun <T> TreeNode.findParentOfTypeT(clazz: Class<T>): T? {
  var p = parent
  while (p != null && !clazz.isInstance(p)) {
    p = p.parent
  }
  return p as? T
}

treeNode.findParentOfTypeT(MyTreeNode::class.java)
```

---

Kotlin 的写法则更加简洁

```Kotlin
inline fun <reified T> TreeNode.parentOfType(): T? {
  var p = parent
  while (p != null && p !is T) {
    p = p.parent
  }
  return p as? T
}
```

在调用时：

```Kotlin
treeNode.parentOfType<MyTreeNode>()
```

---

# tailrec（尾递归）

Kotlin 支持尾递归，这允许通常使用循环来写的算法改用递归函数来写，而无堆栈溢出的风险。尾递归函数必须将其自身的调用作为它执行的最后一个操作。

```Kotlin
// 不会被优化为尾递归，因为最后一个操作是
// n * factorial(n - 1)
tailrec fun factorial(n: Int): Long =
  if (n == 0) 1 else n * factorial(n - 1)
```

改写：

```Kotlin
tailrec fun factorial(n: Int, result: Long) =
  if (n == 0) result else factorial(n - 1, n * result)
```

---

<!-- template: gaia -->

# Q&A
