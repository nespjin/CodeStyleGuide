# Kotlin 样式指南

本文档完整定义了 Google 为以 Kotlin 编程语言编写的源代码制定的 Android 编码标准。当且仅当 Kotlin 源文件符合此处所述的规则时，才可将其描述为采用了 Google Android 样式。

与其他编程样式指南一样，所涵盖的问题不仅涉及格式的美观问题，还涉及其他类型的惯例或编码标准。不过，本文档主要介绍我们普遍遵循的清晰硬性规则，尽量避免给出无法明确执行（无论是通过人工还是工具）的建议。

[上次更新日期：2020 年 6 月 10 日](https://developer.android.google.cn/kotlin/guides-changelog)

## 源文件

所有源文件都必须编码为 UTF-8。

### 命名

如果源文件只包含一个顶级类，则文件名应为该类的名称（区分大小写）加上 `.kt` 扩展名。否则，如果源文件包含多个顶级声明，则应选择一个可描述文件内容的名称（采用 PascalCase 大小写形式）并附上 `.kt` 扩展名。

```kotlin
// MyClass.kt
class MyClass { }
// Bar.kt
class Bar { }
fun Runnable.toBar(): Bar = // …
// Map.kt
fun <T, O> Set<T>.map(func: (T) -> O): List<O> = // …
fun <T, O> List<T>.map(func: (T) -> O): List<O> = // …
```

### 特殊字符

#### 空白字符

除了行终止符序列之外，**ASCII 水平空格字符 (0x20)** 是唯一一种可以出现在源文件中任意位置的空白字符。这意味着：

- 字符串和字符字面量中的其他所有空白字符都会进行转义。
- 制表符不用于缩进。

#### 特殊转义序列

对于任何具有特殊转义序列（`\b`、`\n`、`\r`、`\t`、`\'`、`\"`、`\\` 和 `\$`）的字符，将使用该序列，而不是相应的 Unicode 转义字符（例如 `\u000a`）。

#### 非 ASCII 字符

对于其余非 ASCII 字符，要么使用实际的 Unicode 字符（例如 `∞`），要么使用等效的 Unicode 转义字符（例如 `\u221e`）。具体选择仅取决于哪种字符可使代码**更容易阅读和理解**。建议不要对任何位置的可打印字符使用 Unicode 转义字符，强烈建议不要在字符串字面量和注释之外使用 Unicode 转义字符。

| **示例**                           | **说明**                                         |
| :--------------------------------- | :----------------------------------------------- |
| `val unitAbbrev = "μs"`            | 最好：即使没有注释，也非常清楚。                 |
| `val unitAbbrev = "\u03bcs" // μs` | 差：没有理由对可打印字符使用转义。               |
| val unitAbbrev = "\u03bcs"`        | 差：读者不知道这是什么。                         |
| `return "\ufeff" + content`        | 好：对不可打印字符使用转义，并在必要时添加注释。 |



### 结构

`.kt` 文件由下面几部分组成（按顺序列出）：

- 版权和/或许可标头（可选）
- 文件级注释
- package 语句
- import 语句
- 顶级声明

上述各部分用一个空白行隔开。

#### 版权/许可

如果文件中包含版权或许可标头，应将其放在多行注释的最上方。

```kotlin
/*
 * Copyright 2017 Google, Inc.
 *
 * ...
 */
 
```

请勿使用 [KDoc 样式](https://kotlinlang.org/docs/reference/kotlin-doc.html)或单行样式的注释。

```kotlin
/**
 * Copyright 2017 Google, Inc.
 *
 * ...
 */
// Copyright 2017 Google, Inc.
//
// ...
```

#### 文件级注释

应将具有“file”[使用处目标](https://kotlinlang.org/docs/reference/annotations.html#annotation-use-site-targets)的注释放在任何标头文件注释和软件包声明之间。

#### package 语句

package 语句不受任何列限制且从不换行。

#### import 语句

应将类、函数和属性的 import 语句归在单个列表中并按 ASCII 进行排序。

**不允许**（任何类型的）通配符导入。

与 package 语句类似，import 语句也不受列限制且从不换行。

#### 顶级声明

`.kt` 文件可以在顶级声明一个或多个类型、函数、属性或类型别名。

文件的内容应集中在单个主题上。例如，单个公共类型或对多个接收器类型执行同一操作的一组扩展函数。应将不相关的声明分离到它们自己的文件中，并最大限度地减少单个文件中的公共声明。

对文件的内容量和内容顺序没有做出明确的限制。

通常按从上到下的顺序读取源文件，这意味着，顺序通常应反映出位置比较靠上的声明将有助于理解位置比较靠下的声明。不同的文件可能会选择以不同的方式对内容进行排序。同样，一个文件可能包含 100 个属性，另一个文件可能包含 10 个函数，还有一个文件可能只包含一个类。

重要的是，每个类都采用**某种**逻辑顺序，类的维护人员在被问及时应可以解释清楚相应逻辑顺序。例如，新函数不应直接习惯性地添加到类的末尾，因为这样会产生“按添加日期先后顺序”排序，而这不是逻辑排序。

#### 类成员排序

类中成员的顺序遵循的规则与顶级声明相同。

## 格式设置

### 大括号

`when` 分支不需要大括号，没有 `else if/else` 分支且适合放在一行的 `if` 语句主体也不需要大括号。

```kotlin
if (string.isEmpty()) return

when (value) {
    0 -> return
    // …
}
```

除此以外，任何 `if`、`for`、`when` 分支、`do` 和 `while` 语句都需要大括号，即使主体为空或仅包含一个语句也是如此。

```kotlin
if (string.isEmpty())
    return  // WRONG!

if (string.isEmpty()) {
    return  // Okay
}
```

#### 非空块

对于非空块和类似块的构造，大括号遵循 Kernighan 和 Ritchie (K&R) 样式（“埃及括号”）：

- 左大括号前面没有换行符。
- 左大括号后面有换行符。
- 右大括号前面有换行符。
- 仅当右大括号终止语句或者终止函数、构造函数或命名类的主体时，它后面才有换行符。例如，如果右大括号后跟 `else` 或一个逗号，则它后面没有换行符。

```kotlin
return Runnable {
    while (condition()) {
        foo()
    }
}

return object : MyClass() {
    override fun foo() {
        if (condition()) {
            try {
                something()
            } catch (e: ProblemException) {
                recover()
            }
        } else if (otherCondition()) {
            somethingElse()
        } else {
            lastThing()
        }
    }
}
```

下面给出了[枚举类](https://developer.android.google.cn/kotlin/style-guide#enum-classes)的一些例外情况。

#### 空块

空块或类似块的构造必须采用 K&R 样式。

```kotlin
try {
    doSomething()
} catch (e: Exception) {} // WRONG!
try {
    doSomething()
} catch (e: Exception) {
} // Okay
```

#### 表达式

仅当整个表达式适合放在一行时，用作表达式的 `if/else` 条件语句才能省略大括号。

```kotlin
val value = if (string.isEmpty()) 0 else 1  // Okay
val value = if (string.isEmpty())  // WRONG!
    0
else
    1
val value = if (string.isEmpty()) { // Okay
    0
} else {
    1
}
```

#### 缩进

每个新块或类似块的构造开始时，缩进都会增加四个空格。当块结束时，缩进会恢复到上一个缩进级别。缩进级别适用于整个块中的代码和注释。

#### 每行一个语句

每个语句都后跟一个换行符。不使用分号。

#### 换行

代码的列限制为最多 100 个字符。除非是下面说明的情况，否则任何超过此限制的行都必须换行，如下所述。

例外情况：

- 无法遵循列限制的行（例如，KDoc 中的长网址）
- `package` 和 `import` 语句
- 注释中可以剪切并粘贴到 shell 中的命令行

#### 在何处换行

换行的首要原则是：更倾向于在较高的句法级别换行。此外：

- 某行在运算符或 infix 函数名称处换行时，换行符将在该运算符或 infix 函数名称后面。
- 某行在以下“类似运算符”的符号处换行时，换行符将在该符号前面：
  - 点分隔符（`.`、`?.`）。
  - 成员引用的两个冒号 (`::`)。
- 方法或构造函数名称始终贴在它后面的左圆括号 (`(`) 上。
- 逗号 (`,`) 始终贴在它前面的标记上。
- lambda 箭头 (`->`) 始终贴在它前面的参数列表上。

**注意**：换行的主要目标是让代码清晰，而不一定是让代码适合放在最少数量的行中。

#### 函数

当函数签名不适合放在一行上时，应让每个参数声明独占一行。以这种格式定义的参数应使用单缩进 (+4)。右圆括号 (`)`) 和返回类型独占一行，没有额外的缩进。

```kotlin
fun <T> Iterable<T>.joinToString(
    separator: CharSequence = ", ",
    prefix: CharSequence = "",
    postfix: CharSequence = ""
): String {
    // …
}
```

##### 表达式函数

当函数只包含一个表达式时，它可以表示为[表达式函数](https://kotlinlang.org/docs/reference/functions.html#single-expression-functions)。

```kotlin
override fun toString(): String {
    return "Hey"
}
override fun toString(): String = "Hey"
```

只有在表达式函数开始一个块时，才应换行。

```kotlin
fun main() = runBlocking {
  // …
}
```

否则，如果表达式函数增长到需要换行，应改用普通函数主体、`return` 声明和普通表达式换行规则。

#### 属性

当属性初始化式不适合放在一行时，应在等号 (`=`) 后面换行，并使用缩进。

```kotlin
private val defaultCharset: Charset? =
        EncodingRegistry.getInstance().getDefaultCharsetForPropertiesFiles(file)
```

声明 `get` 和/或 `set` 函数的属性应让每个函数独占一行，并使用正常的缩进 (+4)。对它们进行格式设置时，使用的规则与函数相同。

```kotlin
var directory: File? = null
    set(value) {
        // …
    }
```

只读属性可以使用适合放在一行的较短语法。

```kotlin
val defaultExtension: String get() = "kt"
```

### 空白

#### 垂直

一个空白行会：

- 出现在类的连续成员（属性、构造函数、函数、嵌套类等）之间。

  

  - **例外情况**：两个连续属性（它们之间没有其他代码）之间的空白行是可选的。根据需要使用此类空白行来创建属性的逻辑分组，并将属性与其后备属性（如果存在）相关联。
  - **例外情况**：下文对枚举常量之间的空白行进行了介绍。

- 根据需要出现在语句之间，用于将代码划分为一些逻辑子部分。

- （可选）出现在函数中的第一个语句前面、类的第一个成员前面，或类的最后一个成员后面（既不鼓励也不反对）。

- 根据本文档中其他部分（如[结构](https://developer.android.google.cn/kotlin/style-guide#structure)部分）的要求出现。

允许出现多个连续的空白行，但不鼓励也从不要求必须采用这种样式。

#### 水平

在语言或其他样式规则要求的范围之外，除了字面量、注释和 KDoc，还会出现一个 ASCII 空格，它只会出现在以下位置：

- 将任何保留字（如

   

  ```
  if
  ```

  、

  ```
  for
  ```

   

  或

   

  ```
  catch
  ```

  ）与该行中在它后面的左圆括号 (

  ```
  (
  ```

  ) 隔开。

  ```kotlin
  // WRONG!
  for(i in 0..1) {
  }
  ```

  ```kotlin
  // Okay
  for (i in 0..1) {
  }
  ```

- 将任何保留字（如

   

  ```
  else
  ```

   

  或

   

  ```
  catch
  ```

  ）与该行中在它前面的右大括号 (

  ```
  }
  ```

  ) 隔开。

  ```kotlin
  // WRONG!
  }else {
  }
  ```

  ```kotlin
  // Okay
  } else {
  }
  ```

- 在任何左大括号 (

  ```
  {
  ```

  ) 前面。

  ```kotlin
  // WRONG!
  if (list.isEmpty()){
  }
  ```

  ```kotlin
  // Okay
  if (list.isEmpty()) {
  }
  ```

- 在任何二元运算符的两边。

  ```kotlin
  // WRONG!
  val two = 1+1
  ```

  ```kotlin
  // Okay
  val two = 1 + 1
  ```

  这也适用于以下“类似运算符”的符号：

  - lambda 表达式中的箭头 (

    ```
    ->
    ```

    )。

    ```kotlin
    // WRONG!
    ints.map { value->value.toString() }
    ```

    ```kotlin
    // Okay
    ints.map { value -> value.toString() }
    ```

  但不适用于以下符号：

  - 成员引用的两个冒号 (

    ```
    ::
    ```

    )。

    ```kotlin
    // WRONG!
    val toString = Any :: toString
    ```

    ```kotlin
    // Okay
    val toString = Any::toString
    ```

  - 点分隔符 (

    ```
    .
    ```

    )。

    ```kotlin
    // WRONG
    it . toString()
    ```

    ```kotlin
    // Okay
    it.toString()
    ```

  - 范围运算符 (

    ```
    ..
    ```

    )。

    ```kotlin
    // WRONG
     for (i in 1 .. 4) print(i)
     
    ```

    ```kotlin
     // Okay
     for (i in 1..4) print(i)
    ```

- 仅当在用于指定基类或接口的类声明中使用，或在

  泛型约束

  的

   

  ```
  where
  ```

   

  子句中使用时，才会出现在冒号 (

  ```
  :
  ```

  ) 前面。

  ```kotlin
  // WRONG!
  class Foo: Runnable
  ```

  ```kotlin
  // Okay
  class Foo : Runnable
  ```

  ```kotlin
  // WRONG
  fun <T: Comparable> max(a: T, b: T)
  ```

  ```kotlin
  // Okay
  fun <T : Comparable> max(a: T, b: T)
  ```

  ```kotlin
  // WRONG
  fun <T> max(a: T, b: T) where T: Comparable<T>
  ```

  ```kotlin
  // Okay
  fun <T> max(a: T, b: T) where T : Comparable<T>
  ```

- 在逗号 (

  ```
  ,
  ```

  ) 或冒号 (

  ```
  :
  ```

  ) 后面。

  ```kotlin
  // WRONG!
  val oneAndTwo = listOf(1,2)
  ```

  ```kotlin
  // Okay
  val oneAndTwo = listOf(1, 2)
  ```

  ```kotlin
  // WRONG!
  class Foo :Runnable
  ```

  ```kotlin
  // Okay
  class Foo : Runnable
  ```

- 在开始行尾注释的双正斜杠 (

  ```
  //
  ```

  ) 的两边。在此处，允许出现多个空格，但不要求必须采用这种样式。

  ```kotlin
  // WRONG!
  var debugging = false//disabled by default
  ```

  ```kotlin
  // Okay
  var debugging = false // disabled by default
  ```

绝不能将此规则解释为要求或禁止在行的开头或末尾添加额外的空格，它仅规定内部空格。

### 特定构造



#### 枚举类

对于没有函数且没有关于其常量的文档的枚举，可以选择性地将其格式设为单行。

```kotlin
enum class Answer { YES, NO, MAYBE }
```

将枚举中的常量放在单独的行上时，它们之间不需要空白行，但它们定义主体时除外。

```kotlin
enum class Answer {
    YES,
    NO,

    MAYBE {
        override fun toString() = """¯\_(ツ)_/¯"""
    }
}
```

由于枚举类是类，因此用于类格式设置的其他所有规则都适用。

#### 注释

应将成员或类型注释放在单独的行上，让其紧接在标注的构造前面。

```kotlin
@Retention(SOURCE)
@Target(FUNCTION, PROPERTY_SETTER, FIELD)
annotation class Global
```

可以将不带参数的注释放在一行上。

```kotlin
@JvmField @Volatile
var disposable: Disposable? = null
```

如果只存在一个不带参数的注释，可以将其与声明放在同一行上。

```kotlin
@Volatile var disposable: Disposable? = null

@Test fun selectAll() {
    // …
}
```

`@[...]` 语法只能用于明确的使用处目标，并且只能用于将没有参数的两个或更多注释组合在一行中。

```kotlin
@field:[JvmStatic Volatile]
var disposable: Disposable? = null
```

#### 隐式返回/属性类型

如果表达式函数主体或属性初始化式是标量值，或者可以根据主体明确推断出返回类型，则可以将其省略。

```kotlin
override fun toString(): String = "Hey"
// becomes
override fun toString() = "Hey"
private val ICON: Icon = IconLoader.getIcon("/icons/kotlin.png")
// becomes
private val ICON = IconLoader.getIcon("/icons/kotlin.png")
```

在编写库时，如果显式类型声明是公共 API 的一部分，则应将其保留。

### 命名

标识符仅使用 ASCII 字母和数字，在下面所述的少数情况下，还会使用下划线。因此，每个有效的标识符名称都可匹配正则表达式 `\w+`。

不使用特殊前缀或后缀（例如 `name_`、`mName`、`s_name` 和 `kName` 示例中的前缀或后缀），但后备属性除外（请参阅[后备属性](https://developer.android.google.cn/kotlin/style-guide#backing-properties)）。

#### 软件包名称

软件包名称全部为小写字母，连续的单词直接连接在一起（没有下划线）。

```kotlin
// Okay
package com.example.deepspace
// WRONG!
package com.example.deepSpace
// WRONG!
package com.example.deep_space
```

#### 类型名称

类名采用 PascalCase 大小写形式编写，通常是名词或名词短语。例如，`Character` 或 `ImmutableList`。接口名称也可以是名词或名词短语（例如 `List`），但有时还可以是形容词或形容词短语（例如 `Readable`）。

测试类的命名方式是以测试的类的名称开头且以 `Test` 结尾。例如，`HashTest` 或 `HashIntegrationTest`。

#### 函数名称

函数名称采用 camelCase 大小写形式编写，通常是动词或动词短语。例如，`sendMessage` 或 `stop`。

允许在测试函数名称中出现下划线，用于分隔名称的逻辑组成部分。

```kotlin
@Test fun pop_emptyStack() {
    // …
}
```

#### 常量名称

常量名称使用 UPPER_SNAKE_CASE 大小写形式：全部为大写字母，单词用下划线分隔。但究竟什么是常量？

常量是没有自定义 `get` 函数的 `val` 属性，其内容绝对不可变，并且其函数没有可检测到的副作用。这包括不可变类型和不可变类型的不可变集合，以及标量和字符串（如果标记为 `const`）。如果某个实例的任何可观察状态可以改变，则它不是常量。仅仅计划永不改变对象是不够的。

```kotlin
const val NUMBER = 5
val NAMES = listOf("Alice", "Bob")
val AGES = mapOf("Alice" to 35, "Bob" to 32)
val COMMA_JOINER = Joiner.on(',') // Joiner is immutable
val EMPTY_ARRAY = arrayOf()
```

这些名称通常是名词或名词短语。

常量值只能在 `object` 内定义或定义为顶级声明。满足常量的要求但是在 `class` 内定义的值必须使用非常量名称。

作为标量值的常量必须使用 `const` [修饰符](http://kotlinlang.org/docs/reference/properties.html#compile-time-constants)。

#### 非常量名称

非常量名称采用 camelCase 大小写形式编写。这些适用于实例属性、本地属性和参数名称。

```kotlin
val variable = "var"
val nonConstScalar = "non-const"
val mutableCollection: MutableSet = HashSet()
val mutableElements = listOf(mutableInstance)
val mutableValues = mapOf("Alice" to mutableInstance, "Bob" to mutableInstance2)
val logger = Logger.getLogger(MyClass::class.java.name)
val nonEmptyArray = arrayOf("these", "can", "change")
```

这些名称通常是名词或名词短语。



##### 后备属性

需要[后备属性](https://kotlinlang.org/docs/reference/properties.html#backing-properties)时，其名称应与实际属性的名称完全匹配，只不过带有下划线前缀。

```kotlin
private var _table: Map? = null

val table: Map
    get() {
        if (_table == null) {
            _table = HashMap()
        }
        return _table ?: throw AssertionError()
    }
```

#### 类型变量名称

可以采用以下两种样式之一对每个类型变量命名：

- 一个大写字母，可选择在其后跟一个数字（例如 `E`、`T`、`X` 和 `T2`）
- 用于类的形式的名称，后跟大写字母 `T`（如 `RequestT` 和 `FooBarT`）

#### 驼峰式大小写

有时，有多种合理的方法可将英语短语转换为驼峰式大小写，例如，当存在首字母缩写词或者诸如“IPv6”或“iOS”之类不寻常的构造时。要提高可预测性，请使用以下方案。

从名称的普通形式入手：

1. 将短语转换为纯 ASCII 并移除所有撇号。例如，“Müller’s algorithm”可能会变为“Muellers algorithm”。
2. 将此结果分成单词，在空格和其余所有标点符号（通常是连字符）处分割。建议：如果任何单词在常见用法中已有惯用的驼峰式大小写外观，请将其分成各个组成部分（例如，“AdWords”将变为“ad words”）。请注意，诸如“iOS”之类的单词实际上本身并未采用驼峰式大小写；它违反了任何惯例，所以此建议不适用。
3. 现在，将所有内容（包括首字母缩写词）小写，然后执行以下某项操作：
   - 将每个单词的第一个字符大写，以生成 Pascal 大小写。
   - 将除第一个单词之外的每个单词的第一个字符大写，以生成驼峰式大小写。
4. 最后，将所有单词连接成一个标识符。

请注意，原始单词的大小写几乎完全被忽略。

| **普通形式**           | **正确**            | **不正确**          |
| :--------------------- | :------------------ | :------------------ |
| “XML Http Request”     | `XmlHttpRequest`    | `XMLHTTPRequest`    |
| “new customer ID”      | `newCustomerId`     | `newCustomerID`     |
| “inner stopwatch”      | `innerStopwatch`    | `innerStopWatch`    |
| “supports IPv6 on iOS” | `supportsIpv6OnIos` | `supportsIPv6OnIOS` |
| “YouTube importer”     | `YouTubeImporter`   | `YoutubeImporter`*  |

（* 可接受，但不建议。）

**注意**：在英语中，有些单词并未明确规定是否需要使用连字符连接。例如，“nonempty”和“non-empty”都正确，所以方法名称 `checkNonempty` 和 `checkNonEmpty` 同样也都正确。

### 文档

#### 格式设置

要了解 KDoc 块的基本格式设置，请查看以下示例：

```kotlin
/**
 * Multiple lines of KDoc text are written here,
 * wrapped normally…
 */
fun method(arg: String) {
    // …
}
```

或查看以下单行示例：

```kotlin
/** An especially short bit of KDoc. */
```

基本形式始终可接受。当整个 KDoc 块（包括注释标记）适合放在一行时，就可以替换为单行形式。请注意，仅当没有块标记（如 `@return`）时，此规则才适用。

##### 段落

一个空白行（即，仅包含对齐的前导星号 (`*`) 的行）会出现在段落之间，以及成组的块标记（如果存在）前面。

##### 块标记

使用的任何标准“块标记”都按 `@constructor`、`@receiver`、`@param`、`@property`、`@return`、`@throws` 和 `@see` 的顺序出现，并且这些块标记从不与空说明一起出现。当块标记不适合放在一行时，连续行会从 `@` 的位置开始缩进 4 个空格。

#### 摘要片段

每个 KDoc 块都以一个简短的摘要片段开头。此片段非常重要：它是唯一一个会出现在某些上下文（例如类和方法索引）中的文本部分。

这是一个片段（名词短语或动词短语），而不是一个完整的句子。它不以“`A `Foo` is a...`”或“`This method returns...`”开头，也不必形成一个完整的祈使句（如“`Save the record.`”）。不过，此片段就像一个完整的句子一样首字母大写并加标点符号。

#### 用法

至少每个 `public` 类型以及这样的类型的每个 `public` 或 `protected` 成员都应存在 KDoc，下面列出了一些例外情况。

##### 例外情况：不言自明的函数

对于像 `getFoo` 这样“简单明了”的函数以及像 `foo` 这样“简单明了”的属性，KDoc 是可选的。这是因为，对于这些函数和属性，除了“Returns the foo”，真的没有什么需要说明的。

不能用这一例外情况证明省略一般读者可能需要知道的相关信息的合理性，这种做法不当。例如，对于名为 `getCanonicalName` 的函数或名为 `canonicalName` 的属性，请勿省略其文档（理由是它只写着 `/** Returns the canonical name. */`），因为一般读者可能不知道“canonical name”一词是什么意思！

##### 例外情况：替换

替换超类型方法的方法并不总是存在 KDoc。