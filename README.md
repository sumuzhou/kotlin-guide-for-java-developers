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
Kotlin的if既可以是语句，也可以是表达式，意味着可以将if的执行结果赋给一个变量，所以我们也就不再需要三元操作符：
```Kotlin
val a = 1
val b = 2

var traditionalMax = a
if (a < b) traditionalMax = b

var withElseMax: Int
if (a > b) {
    withElseMax = a
} else {
    withElseMax = b
}

val expressionMax = if (a > b) a else b

val blockExpressionMax = if (a > b) {
    print("Choose a")
    a // return value
} else {
    print("Choose b")
    b // return value
}
```
如果把if当作表达式，那么else语句就是必须的，因为编译器要确保一定能计算一个值出来。在if中使用表达式块时，最后一个表达式得到的结果就是整个块的返回值。

当有多种情况进行分支判断时，用if就会产生大量的嵌套，而Java的switch又有诸多限制。针对这个问题，Kotlin提供了when关键字，帮助我们更好进行多分支控制。和if一样，when既可以是语句，也可以是表达式；同时when的判断对象是可选的：
```Kotlin
val x = 1
// 表达式, i = 2
val i = when(x) {
    1, 2, "3".toInt() -> x * 2 // 值判断，可以调用函数
    in 3..5 -> 10 // 范围判断
    !in 6..10 -> 20 // 范围判断
    else -> 30 // 必须
}

// 语句
// 可以把变量scope限制在when block
// 可以用复杂类型
when (val list = listOf(1, 2, 3)) {
    listOf(1, 2, 3) -> println(list.last())
}

// 不传判断对象，则每个分支的条件都是一个Boolean值
when {
    x < 10 -> println("less than 10")
    x is Int -> println("of course")
    x !is Int -> println("are you kidding")
}
```
对于表达式形式的when，通常也必须加上else，除非编译器能够确认已经穷举所有可能（当判断对象是enum或者sealed class时）。

Kotlin也提供了for和while循环，语法基本和Java一致。在循环中，Kotlin也支持break和continue，但值得注意的是Kotlin通过标签[Label]强化了对break和continue的控制，在嵌套循环中，可以直接跳到外部循环：
```Kotlin
// implicit break
for (i in 1..5) {
    if (i == 2) break
    println(i)
}
// labeled break, same as implicit break
anyName@ for (i in 1..5) {
    if (i == 2) break@anyName
    println(i)
}

// break inner loop
// (1,1) (2,1) (3,1)
for (i in 1..3) {
    for (j in 1..3) {
        if (j == 2) break
        print("($i,$j) ")
    }
}
// break outer loop
// (1,1)
loop@ for (i in 1..3) {
    for (j in 1..3) {
        if (j == 2) break@loop
        print("($i,$j) ")
    }
}
```
标签可以用于标记任意表达式，以小写驼峰形式+@来命名。注意break和@之间没有空格。除了break和continue，return也会和标签结合起来使用，在后续章节中会讲到。

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
Surprise!我们看下Kotlin对return关键字的描述：“By default returns from the nearest enclosing function or anonymous function.”，很明显，Lambda表达式没有函数scope，所以这里的return是外围函数（也就是main函数）的return，返回类型当然对不上。可以用标签或者匿名函数的方式来解决这个问题：
```Kotlin
fun main() {
    // explicit label
    intArrayOf(1, 2, 3).filter label@ {
        val shouldFilter = it > 0
        return@label shouldFilter
    }
    // default label
    intArrayOf(1, 2, 3).filter {
        val shouldFilter = it > 0
        return@filter shouldFilter
    }
    // anonymous function
    intArrayOf(1, 2, 3).filter(fun(i: Int): Boolean {
        val shouldFilter = i > 0
        return shouldFilter
    })
}
```
:bell:**注意**:bell:
> 和Java不同，Kotlin的Lambda表达式没有函数scope，同样的代码会出现不一样的效果，要特别注意：
```Java
// Java
// will println 1 3 4 5
IntStream.rangeClosed(1, 5).forEach(i -> {
    if (2 == i) return;;
    System.out.println(i);
});
System.out.println("Done");
```
```Kotlin
// Kotlin
// will println 1
(1..5).forEach {
    if (it == 2) return
    println(it)
}
println("this point is unreachable")
```
> Kotlin的Lambda中把main函数给return了，导致后续的语句都不会执行。

:+1:**最佳实践**:+1:
> 在Kotlin的Lambda中尽量避免写return，比如上述的return可以用filter功能代替。如果一定要写return，不要忘了带上标签。

### 扩展函数[Extension Function]
在Java里，如果我们想在不修改类代码的情况下扩展一个对象的功能（提供更多的方法），可以怎么做呢？简单的，可以写静态Utils函数，将对象作为入参；规范一点的，可以继承该类，或者做一个装饰器[Decorator]，后续使用这些新创建的类。说实话，都有点麻烦。Kotlin提供了扩展函数来解决这个问题：
```Kotlin
val vowels = setOf('a', 'e', 'i', 'o', 'u')
fun String.filterNonVowel() = this.filter { it.toLowerCase() in vowels }
fun String.countVowel() = filterNonVowel().count() // this可以省略
println("Hello".filterNonVowel()) // eo
println("Hello".countVowel()) // 2
```
.前面的类型被称为接收者类型[Receiver Type]，在函数中接收者以this的形态出现，可以通过this访问接收者的方法和属性，this关键字可以省略。能够访问的方法和属性遵循可见性原则，后续章节会讲述。

:+1:**最佳实践**:+1:
> 扩展函数中的this关键字，在变量/方法较少，能一眼看出变量/方法来自接收者时可以省略，让代码更简洁。但是如果函数内使用的变量/方法较多，最好加上this，让阅读者能够轻易区分出哪些来自接收者，哪些不是。

:bell:**注意**:bell:
> 扩展函数的接收者类型是静态解析的，也就是说不支持多态，只依赖编译时的声明类型，而非运行时的实际类型：
```Kotlin
open class Shape
class Rectangle: Shape()

fun Shape.getName() = "Shape"
fun Rectangle.getName() = "Rectangle"

fun printClassName(s: Shape) {
    println(s.getName())
}
// print Shape
printClassName(Rectangle())
```

:bell:**注意**:bell:
> 如果扩展函数和接收者的方法都可适用时，优先使用方法：
```Kotlin
class Example {
    fun printFunctionType() { println("Class method") }
}
fun Example.printFunctionType() { println("Extension function") }

// print Class method
Example().printFunctionType()
```

### 语法糖：TODO、中缀函数与运算符函数[Syntax Sugar: TODO & Infix Function & Operator Function]
Kotlin的真香很大一部分原因来自语法糖，本章讲述了和函数相关的三个语法糖，这些语法糖在Kotlin的库函数中也大量使用。

* TODO

在写Java代码时，无论是出于对接需要还是TDD开发，我们很可能在声明接口后，先给接口做一个Mock实现，来保证代码编译运行不会报错。这样做会遇到一些问题：
1. 用null作为Mock返回值抛出的空指针异常信息没有指导意义，而人造数据又费时间；
2. 除了在注释里写TODO，没有机制来跟踪这些Mock的实现。

Kotlin提供了TODO函数来解决这个问题，TODO函数的返回类型是Nothing，可以用于任何的函数：
```Kotlin
fun main() {
    // Exception in thread "main" kotlin.NotImplementedError: An operation is not implemented: Waiting for feedback from accounting
    calcTaxes()
}

fun calcTaxes(): Int = TODO("Waiting for feedback from accounting")
```
TODO函数能够帮助我们快速实现Mock功能，并且异常信息说明了原因，IDE一般也提供了追踪的功能，无需再写注释。

* 中缀函数

中缀函数就是用infix关键字修饰的函数，它要求这个函数
1. 是一个类方法或者扩展函数；
2. 函数只有一个参数，并且这个参数不是可变长参数，也没有默认值。
```Kotlin
infix fun Int.shl(x: Int): Int { ... }
// calling the function using the infix notation
1 shl 2
// is the same as
1.shl(2)
```
:+1:**最佳实践**:+1:
> 中缀函数的优先级低于算数操作，高于布尔操作，所以为了代码可读性，在混合时使用()来明确执行逻辑：

> `1 shl 2 + 3`写成`1 shl (2 + 3)`

> `a && b xor c`写成`a && (b xor c)`

* 运算符函数

运算符函数就是使用operator关键字修饰的函数，它的最初目的是为了重载运算符，但实际上应用范围远不止运算符。Kotlin的机制很简单，就是设置了特定语法到特定名称运算符函数的转换，当一个类实现了指定的运算符函数时，就可以有特定的语法来调用：
```Kotlin
fun main() {
    // Huh?
    // print I'm an Int, my value is 1
    1()
}

operator fun Int.invoke() = println("I'm an Int, my value is $this")
```
:bell:**注意**:bell:
> 实际代码中请不要因为fancy而滥用运算符函数，比如上面的代码，调用Int没有任何意义。

常用的运算符函数如下表所示：
| 表达式 | 转换到函数 |
| --- | --- |
| `a()`  | `a.invoke()` |
| `a(i)`  | `a.invoke(i)` |
| `a(i, j)`  | `a.invoke(i, j)` |
| `a[i]`  | `a.get(i)` |
| `a[i, j]`  | `a.get(i, j)` |
| `a[i] = b` | `a.set(i, b)` |
| `a[i, j] = b` | `a.set(i, j, b)` |
| `a + b` | `a.plus(b)` |
| `a - b` | `a.minus(b)` |
| `a * b` | `a.times(b)` |
| `a / b` | `a.div(b)` |
| `a % b` | `a.rem(b)` |
| `a..b ` | `a.rangeTo(b)` |
| `a in b` | `b.contains(a)` |
| `a !in b` | `!b.contains(a)` |
| `a == b` | `a?.equals(b) ?: (b === null)` |
| `a != b` | `!(a?.equals(b) ?: (b === null))` |
| `a > b`  | `a.compareTo(b) > 0` |
| `a < b`  | `a.compareTo(b) < 0` |
| `a >= b` | `a.compareTo(b) >= 0` |
| `a <= b` | `a.compareTo(b) <= 0` |

除此之外，还有两组特别一点的运算符函数，解构和遍历：
```Kotlin
// 遍历
operator fun Int.iterator(): MyIterator = MyIterator(this)
class MyIterator(var i: Int) {
    operator fun hasNext() = i > 0
    operator fun next() = i--
}
// 解构
operator fun Int.component1() = this - 1
operator fun Int.component2() = this + 1
// 使用
fun main() {
    /*
    i=4, j=6
    i=3, j=5
    i=2, j=4
    i=1, j=3
    i=0, j=2
     */
    for ((i, j) in 5) println("i=$i, j=$j")
}
```
解构可以支持到N个，只要增加函数即可；遍历不仅要求对象本身有运算符函数，还要求函数的返回对象拥有对应的运算符函数（可以不实现Iterator接口，只要实现运算符函数即可）。

:bell:**注意**:bell:
> 在解构Lambda表达式的参数时，请注意语法：
```Kotlin
{ a -> ... } // one parameter
{ a, b -> ... } // two parameters
{ (a, b) -> ... } // a destructured pair
{ (a, b), c -> ... } // a destructured pair and another parameter
```

### 函数入参[Function Argument]
Java的函数不支持参数默认值，一般只能通过重载来实现：
```Java
public String sayHello() {
    return sayHello("Stranger");
}

public String sayHello(String name) {
    return "Hello " + name;
}
```
Kotlin提供了入参默认值，可以直接实现：
```Kotlin
fun sayHello(name: String = "Stranger") = "Hello $name"
```
在Java里面，我们遇到入参较多的函数调用时往往需要在调用和签名之间来回切换，才能理解传入参数的意义；对此，Kotlin提供了命名入参，可以增强调用处的代码可读性：
```Kotlin
// 定义
fun reformat(str: String,
             normalizeCase: Boolean,
             upperCaseFirstLetter: Boolean,
             divideByCamelHumps: Boolean,
             wordSeparator: Char) {
/*...*/
}
fun main() {
    // 调用
    reformat(str = "Hello World",
            normalizeCase = true,
            upperCaseFirstLetter = true,
            divideByCamelHumps = false,
            wordSeparator = '_'
    )
}
```
和Java一样，Kotlin也支持不定长形式的入参，并且在调用的时候可以通过扩展操作符[Spread Operator]*来便利使用。不定长入参本质上是一个数组：
```Kotlin
fun main() {
    val list = arrayOf("Hello", "World")
    // println Hello World Kotlin
    printArgs(*list, "Kotlin")
}

fun printArgs(vararg ss: String) {
    for (s in ss) println(s)
}
```
带默认值的参数和不定长参数不一定在最后，并且可以混合使用，调用的时候需要通过命名参数来明确：
```Kotlin
fun main() {
    // $ Hello %
    // $ World %
    printArgs(ss = *arrayOf("Hello", "World"), suffix = "%")
    // # Hello #
    // # World #
    printArgs("#", "Hello", "World", suffix = "#")
}

fun printArgs(prefix: String = "$", vararg ss: String, suffix: String) {
    for (s in ss) println("$prefix $s $suffix")
}
```
:bell:**注意**:bell:
> 从Kotlin调用Java的方法不支持命名参数，因为Java的字节码不一定保留参数的名称。

### 作用域函数[Scope Function]
Kotlin提供了一类特殊的函数，它们的目的是在一个指定的作用域中执行一段Lambda表达式，这种函数就称为作用域函数：
```Kotlin
fun main() {
    val s = "Hello".apply {
        println(length)
    }.also {
        println(it)
    }.let {
        it.plus("World")
    }

    assert("HelloWorld" == s)
}
```
上述代码中，apply、also和let都是作用域函数，对应的作用域是Hello对象，可以不加对象名称，直接访问对象的属性。Kotlin常用的作用域函数如下：
| 函数 | 对象引用 | 返回值 |
| --- | --- | --- |
| let | it | Lambda表达式结果 |
| run | this | Lambda表达式结果 |
| with | this | Lambda表达式结果 |
| apply | this | 作用域对象 |
| also | it | 作用域对象 |
对于this类型的对象引用，作用域函数本质上而言就是一个无入参的扩展函数。

:+1:**最佳实践**:+1:
> 对于以it形式引用对象的作用域函数，一般在Lambda表达式中，会将对象作为整体使用，比如作为参数传递给另外的函数：
```Kotlin
val numbers = mutableListOf("one", "two", "three")
numbers.map { it.length }.filter { it > 3 }.let { println(it) }
numbers.also { println("The list elements before adding new one: $it") }.add("four")
```
> with函数常用于基于对象属性来进行计算，并返回结果：
```Kotlin
val numbers = mutableListOf("one", "two", "three")
val firstAndLast = with(numbers) {
    "The first element is ${first()}," +
    " the last element is ${last()}"
}
```
> apply函数返回的是对象本身，常用于对对象进行设置：
```Kotlin
val adam = Person("Adam").apply {
    age = 32
    city = "London"        
}
```

<!--
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
    val a = 1
    val b = 2
    val blockExpressionMax = foo@ if (a > b) {
        print("Choose a")
        return@foo a
    } else {
        print("Choose b")
        return@foo b
    }
    println(blockExpressionMax) <- 这里就会出问题
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
