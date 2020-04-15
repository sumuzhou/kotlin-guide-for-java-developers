# 写给Java程序员的Kotlin指南[Kotlin Guide for Java Developers]
<!--
生产力是第一要素，平滑的学习曲线（参考scala）
如何使用kotlin？playground or idea
不是大而全的文档，而是对Java Dev一份划重点指南
kotlin的版本
kotlin koans
-->

## 语法和控制流[Syntax & Control Flow]
### 基本语法[Basic Syntax]
```Kotlin
// 声明一个不可变的整型
val immutableInt: Int = 1

// 声明一个可变的长整型，省略类型，由编译器推断
var mutableLong = 1L

// 声明浮点数
val iAmDouble = 1.1
val iAmFloat = 1.1F

// 可以加_增强可读性
val oneBillion: Int = 1_000_000_000

// 没有默认转换[Implicit Conversion]
/*
    Type mismatch.
    Required:
    Long
    Found:
    Int
 */
//val aLong: Long = oneBillion
/*
    Operator '==' cannot be applied to 'Int' and 'Long'
 */
//val isEquals: Boolean = oneBillion == aLong
// 使用显式转换[Explicit Conversion]
val convertToLong: Long = oneBillion.toLong()

// 使用字符串
val hello: String = "Hello Kotlin\n"
val multilineString = """
    Hello 
    Kotlin
""".trimIndent()
val useStringTemplate = "I have $oneBillion dollar."
val callMethodInTemplate = "1B plus 1 is ${oneBillion + 1}"

// main函数是程序入口，可以省略args参数
//fun main(args: Array<String>) {
fun main() {
    // 字符串是不可变的，生成新的字符串
    val helloAgain = hello + ""
    // Identity, false
    println(hello === helloAgain)
    // Equality, true
    println(hello == helloAgain)
}
```
:bell:**注意**:bell:
> Kotlin中类型后置，无论是变量、函数参数还是函数返回值的类型，都是放在签名后面，通过:分隔。前置放的都是修饰符，比如val、var、public等。

和Java不同名，Kotlin的变量、函数不要求一定放在类里面，可以放在顶层[Top Level]；同时，单个文件可以包含多个类，文件名也不强制要求与类的名称对应。后面会有单独的章节讲述Kotlin的代码规范。

### 控制流

## 函数[Function]
为什么把函数的内容放在了其他章节前面，因为真香:smile:。

### 函数声明[Function Declaration]
Kotlin的函数用fun关键字声明，有如下两种声明方式：
* 函数体[Block Body]
```Kotlin
fun sayHelloTo(name: String): String {
    return "Hello $name"
}
fun printHello(name: String) {
    println(sayHelloTo(name))
}
```
除非返回类型是Unit（即没有返回值，对应Java的Void），否则函数体需要指明返回类型，并且return关键词也必须有。

:bell:**注意**:bell:
> 如果函数体不写返回类型，会被认为没有返回值（即返回类型是Unit）。

* 表达式[Expression]
```Kotlin
fun sayHelloTo(name: String) = "Hello $name"
fun printHello(name: String) = println(sayHelloTo(name))
```
使用表达式形式声明函数时可以不加返回类型，编译器会自动推断出返回类型。

:+1:**最佳实践**:+1:
> 对于能用一行代码来表示的函数，使用表达式形式，如果返回类型不明显，手动加上返回类型；对于复杂的函数，使用函数体的形式。

### 高阶函数与Lambda[Higher-Order Function & Lambda]
函数在Kotlin中是一等公民，意味着可以把函数赋给一个变量，也可以将函数作为入参和返回值，这样形成的函数称为高阶函数（也就是说这个函数的入参或返回值是另外一个函数）。

Kotlin作为静态类型语言，内建了函数类型来表示函数，比如(String) -> Int，可以通过多种方式创建函数类型的实例：
```Kotlin
val byLambda: (String) -> Int = { s -> s.length }
val byAnonymousFunction: (String) -> Int = fun(s: String): Int { return s.length }
val byFunctionReference: (String) -> Int = String::hashCode
val byPropertyReference: (String) -> Int = String::length
val byConstructor: (String) -> Regex = ::Regex // Regex的构造器入参是一个字符串pattern
```
一般来说，用的最多的还是Lambda，所以Kotlin为我们提供了一个针对单入参Lambda的语法糖：默认参数名it。例如，上面通过Lambda创建函数类型的代码就可以简化成：
```Kotlin
val byLambdaWithImplicitName: (String) -> Int = { it.length }
```
在实际使用中，特别是对集合进行操作时，单入参函数特别常见，it语法糖能让代码变得更简洁。

:+1:**最佳实践**:+1:
> 对于单入参Lambda，在语义清晰的时候，使用it语法作为默认参数；在某些复杂情况下，比如函数嵌套，需要使用完整的Lambda语法来给入参命名，以明确语义，并且避免内部的it隐藏外部的it：
```Kotlin
val list = listOf("Hello", "World")
// Shadowed it
list.maxBy({
    it.filter({
        it == 'o'
    }).length
})
// Explicit name
list.maxBy({ str ->
    str.filter({ char ->
        char == 'o'
    }).length
})
```
在上面的例子中，我们将Lambda表达式作为函数入参传入时，还需要用()括起来，非常繁琐。Kotlin也针对这个问题，给我们提供了一个让使用函数类型更自然的语法糖：外置Lambda入参。具体来说，就是如果一个函数的最后一个入参是函数类型，那么在通过Lambda表达式传入时，Lambda表达式不用放在括号里，可以像语句块一样直接放在外面。上述例子可以改成：
```Kotlin
val list = listOf("Hello", "World")
// Shadowed it
list.maxBy {
    it.filter {
        it == 'o'
    }.length
}
// Explicit name
list.maxBy { str ->
    str.filter { char ->
        char == 'o'
    }.length
}
```
对于多入参的Lambda表达式，如果有入参不需要使用，可以用_来命名，明确表示不使用：
```Kotlin
map.forEach { _, value -> println("$value!") }
```
Kotlin的Lambda表达式和Java的Lambda表达式在使用上有两个值得注意的区别：
* 修改闭包[Closure]的数据

在Java中，闭包中的数据对Lambda表达式而言是final的，意味着不能重新赋值：
```Java
int sum = 0;
IntStream.range(0, 10).forEach(
        i -> sum += i // Compile Error: Variable used in lambda expression should be final or effectively final
);
```
在Kotlin中，Lambda表达式可以修改闭包中的数据：
```Kotlin
var sum = 0
intArrayOf(1, 2, 3).forEach {
    sum += it
}
```
* 返回值

Kotlin的Lambda表达式如果没有写return关键字，最后一个语句的执行结果就是返回值：
```Kotlin
intArrayOf(1, 2, 3).filter {
    val shouldFilter = it > 0
    shouldFilter // return shouldFilter, which is a Boolean
}
```
如果写了return关键字会怎么样呢：
```Kotlin
fun main() {
    intArrayOf(1, 2, 3).filter {
        val shouldFilter = it > 0
        /*
        Compile Error
            Type mismatch.
            Required:
            Unit
            Found:
            Boolean
         */
        return shouldFilter
    }
}
```
Surprise!因为Kotlin的Lambda表达式创建的是函数类型实例，并没有函数scope，所以这里的return是外围函数（也就是main函数）的return，返回类型当然对不上。Kotlin提供了标签[Label]的方式来解决这个问题：
```Kotlin
fun main() {
    intArrayOf(1, 2, 3).filter {
        val shouldFilter = it > 0
        return@filter shouldFilter
    }
}
```
这里的@filter就是标签。Kotlin中有多种场景会用到标签，这里不再展开。

<!--
### 扩展函数/Extension Function
extension function - 限制范围

### 语法糖 - 重载运算符、解构、中缀函数/Syntax Sugar - Operator Overloading, Destructuring, Infix Function
syntax sugur - operator overloading + deconstruct + infix function

### 函数入参/Function Argument
default parameter + named parameter + vararg

### 作用域函数/Scope Function
scope function

### 内联函数/Inline Function
inline function

## 基础/语法
注释doc - 不用@param和@return
expression vs statement - for while @label - 注意陷阱
string template
类型后置
pattern match - when
unchecked exception
TODO
一个文件可以放多个类，以及top level variable - best practice
on implict conversion - use explict conversion

## 类型
type inference
null safety - ? ?:
smart cast

## 类和对象
primary/secondary constructor init
data class
Pair
Collection - immutable mutable
getter setter
delegation
generics - variance
companion object
factory
const lateinit
subclass - open override

## 协程

## 和Java互调

## Convention
convention[https://kotlinlang.org/docs/reference/coding-conventions.html]
测试名称可以用``

## Spring工程
-->
