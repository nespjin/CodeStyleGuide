# 面向贡献者的 AOSP Java 代码样式指南

向 Android 开源项目 (AOSP) 贡献 Java 代码时，请务必严格遵守本页介绍的代码样式规则。如果向 Android 平台贡献的代码没有遵守这些规则，平台通常不会接受这些代码。我们知道，并非所有现有的代码都遵守这些规则，但我们希望所有新代码都能遵守这些规则。

**注意**：这些规则针对的是 Android 平台，Android 应用开发者可以不遵守这些规则。应用开发者可以遵守他们选择的标准，如 [Google Java 样式指南](https://google.github.io/styleguide/javaguide.html)。

## 保持一致

最简单的一条规则就是要保持一致。如果您正在修改代码，请花几分钟时间看一下修改部分前后的代码，并据此确定修改部分应使用的样式。如果这些代码在 `if` 语句周围使用空格，那么您也应该这样做。如果代码注释的周围是用星号组成的小方框，您也应该将您的注释放在这样的小方框内。

制定样式指南的目的是整理出通用的编码词汇表，以便读者可以专注于您所表达的内容，降低您的表达方式对内容的影响。我们在此提出整体样式规则，让您了解这一词汇表，但局部样式也很重要。如果您添加到文件中的代码看起来与其周围的现有代码明显不同，那么当读者读到此处时，这些代码会打乱他们的节奏。请尽量避免这种情况。

## Java 语言规则

Android 遵循标准 Java 编码规范以及下文所述的其他规则。

### 请勿忽略异常

开发者可能会倾向于编写忽略异常的代码，例如：

```Java
  void setServerPort(String value) {
      try {
          serverPort = Integer.parseInt(value);
      } catch (NumberFormatException e) { }
  }
```

请不要这样做。虽然您可能认为自己的代码永远不会遇到这种错误，或者认为无需费心处理这种错误，但忽略这类异常会在您的代码中埋下隐患，这种错误总有一天会被他人触发。您必须有原则地处理代码中的每个异常；处理方式因具体情况而异。

“*无论何时，只要遇到空的 catch 子句，就应该保持警惕。当然，在某些时候，空的 catch 语句确实没什么问题，但至少你得停下来想一想。在 Java 中，无论怎么小心都不为过。*”— [James Gosling](http://www.artima.com/intv/solid4.html)

可接受的替代方案（按优先顺序排列）包括：

- 将异常抛给方法调用方。

  ```java
    void setServerPort(String value) throws NumberFormatException {
        serverPort = Integer.parseInt(value);
    }
  ```

- 抛出一个适合您的抽象级别的新异常。

  ```java
    void setServerPort(String value) throws ConfigurationException {
      try {
          serverPort = Integer.parseInt(value);
      } catch (NumberFormatException e) {
          throw new ConfigurationException("Port " + value + " is not valid.");
      }
    }
  ```

- 妥善处理错误，并替换

   

  ```java
  catch {}
  ```

   

  块中的相应值。

  ```
    /** Set port. If value is not a valid number, 80 is substituted. */
  
    void setServerPort(String value) {
      try {
          serverPort = Integer.parseInt(value);
      } catch (NumberFormatException e) {
          serverPort = 80;  // default port for server
      }
    }
  ```

- 捕获异常并抛出一个新的

   

  ```java
  RuntimeException
  ```

   

  实例。这样做比较危险，因此除非您确定发生此错误时最适当的处理方式就是让应用崩溃，否则请勿采用这种方案。

  ```java
    /** Set port. If value is not a valid number, die. */
  
    void setServerPort(String value) {
      try {
          serverPort = Integer.parseInt(value);
      } catch (NumberFormatException e) {
          throw new RuntimeException("port " + value " is invalid, ", e);
      }
    }
  ```

  **注意**：原始异常会传递到 `RuntimeException` 的构造函数。如果您的代码必须采用 Java 1.3 进行编译，您必须忽略作为根源的异常。

- 最后一种方案：如果您确信忽略异常是合适的处理方式，那么您可以忽略异常，但您必须添加注释来充分说明理由：

  ```java
  /** If value is not a valid number, original port number is used. */
  
  void setServerPort(String value) {
      try {
          serverPort = Integer.parseInt(value);
      } catch (NumberFormatException e) {
          // Method is documented to just ignore invalid user input.
          // serverPort will just be unchanged.
      }
  }
  ```

### 请勿捕获常规异常

在捕获异常时，开发者可能会为了偷懒而倾向于采用以下处理方式：

```java
  try {
      someComplicatedIOFunction();        // may throw IOException
      someComplicatedParsingFunction();   // may throw ParsingException
      someComplicatedSecurityFunction();  // may throw SecurityException
      // phew, made it all the way
  } catch (Exception e) {                 // I'll just catch all exceptions
      handleError();                      // with one generic handler!
  }
```

请不要这样做。几乎所有情况下都不适合捕获常规 `Exception` 或 `Throwable`（最好不要捕获 `Throwable`，因为它包含 `Error` 异常）。这样做非常危险，因为这意味着系统会在处理应用级错误时捕获到您意料之外的异常（包括 `ClassCastException` 之类的运行时异常）。这种处理方式掩盖了代码的故障处理属性，也就是说，如果有人在您调用的代码中引入了一种新类型的异常，编译器不会指出您需要以不同的方式处理该错误。在大多数情况下，您不应以相同的方式处理不同类型的异常。

这条规则的特例是：在测试代码和顶级代码中，您需要捕获所有类型的错误（以防它们显示在界面中，或者以便某个批处理作业保持运行）。在这些情况下，您可以捕获常规 `Exception`（或 `Throwable`）并妥善地处理错误。但在这样做之前，请认真考虑，然后添加注释说明为何在这种情况下执行这类操作是安全之举。

捕获常规异常的替代方案：

- 将每个异常作为多个 catch 块的一部分分别进行捕获，例如：

  ```java
  try {
      ...
  } catch (ClassNotFoundException | NoSuchMethodException e) {
      ...
  }
  ```

- 通过多个 try 块重构您的代码，获得更精细的错误处理过程。从解析中分离出 IO，然后分别处理每种情况下的错误。

- 重新抛出异常。很多时候，您无需在该级别捕获异常，只需让相应方法抛出异常即可。

请谨记，异常是您的朋友！当编译器“抱怨”您没有捕获异常时，别闷闷不乐。您应该微笑！因为编译器让您能够更加轻松地捕获代码中的运行时错误。

### 请勿使用终结器

终结器可以在对象被垃圾回收器回收时执行一段代码。虽然终结器在进行资源清理（尤其是外部资源）时很好用，但我们无法保证终结器何时被调用（或者会不会被调用）。

Android 不使用终结器。在大多数情况下，您可以使用良好的异常处理代替终结器。如果您确实需要使用终结器，请定义一个 `close()` 方法（或类似方法），并注明需要调用该方法的确切时机（有关示例，请参阅 [InputStream](https://developer.android.google.cn/reference/java/io/InputStream?hl=zh-CN)）。这种情况下，可以（但并非必须）在终结器中输出简短的日志消息，前提是不会输出大量日志消息。

### 完全合格的导入

当您想要使用 `foo` 软件包中的 `Bar` 类时，可以使用以下两种方式导入：

- ```java
  import foo.*;
  ```

  可能会减少 import 语句的数量。

- ```java
  import foo.Bar;
  ```

  明确指出实际使用了哪些类，而且代码对于维护者来说更清晰易读。

使用 `import foo.Bar;` 导入所有 Android 代码。对于 Java 标准库（`java.util.*`、`java.io.*` 等）和单元测试代码 (`junit.framework.*`)，确立了一种明确的异常。

## Java 库规则

使用 Android 的 Java 库和工具时需要遵守相关规范。在某些情况下，具体规范发生了一些重大变化，旧代码使用的可能是已弃用的模式或库。使用此类代码时，可以继续遵循现有样式。不过，在创建新组件时，切勿使用已弃用的库。

## Java 样式规则

### 使用 Javadoc 标准注释

每个文件都应该在顶部放置版权声明，后跟 package 和 import 语句（各个块之间用空行分隔），最后是类或接口声明。在 Javadoc 注释中，说明类或接口的作用。

```java
/*
 * Copyright 2020 The Android Open Source Project
 *
 * Licensed under the Apache License, Version 2.0 (the "License");
 * you may not use this file except in compliance with the License.
 * You may obtain a copy of the License at
 *
 *      http://www.apache.org/licenses/LICENSE-2.0
 *
 * Unless required by applicable law or agreed to in writing, software
 * distributed under the License is distributed on an "AS IS" BASIS,
 * WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
 * See the License for the specific language governing permissions and
 * limitations under the License.
 */

package com.android.internal.foo;

import android.os.Blah;
import android.view.Yada;

import java.sql.ResultSet;
import java.sql.SQLException;

/**
 * Does X and Y and provides an abstraction for Z.
 */

public class Foo {
    ...
}
```

您编写的每个类和重要的公共方法都必须包含 Javadoc 注释，至少用一句话说明类或方法的用途。句子应以第三人称描述性动词开头。

**示例**

```java
/** Returns the correctly rounded positive square root of a double value. */

static double sqrt(double a) {
    ...
}
```

或

```java
/**
 * Constructs a new String by converting the specified array of
 * bytes using the platform's default character encoding.
 */
public String(byte[] bytes) {
    ...
}
```

对于普通的 get 和 set 方法（如 `setFoo()`），如果 Javadoc 的全部内容只包含“设置 Foo”，则无需编写 Javadoc。如果相关方法执行的操作更复杂（例如强制实施约束条件或具有严重的副作用），那么您必须添加注释。如果“Foo”属性的意思不明确，您也应该添加注释。

对于您所编写的每一种方法（无论是公共方法还是其他方法），编写 Javadoc 都是有好处的。公共方法是 API 的一部分，因此需要 Javadoc。Android 目前并不强制要求以特定样式编写 Javadoc 注释，但建议您遵循[如何为 Javadoc 工具编写文档注释](http://www.oracle.com/technetwork/java/javase/documentation/index-137868.html)中的说明。

### 编写简短方法

尽可能编写短小精炼的方法。我们了解，有些情况下适合编写较长的方法，因此对方法的代码长度没有做出硬性限制。如果某个方法的代码超出 <font color="#FF0000">40</font> 行，请考虑是否可以在不破坏程序结构的前提下对其进行拆解。

### 在标准位置定义字段

在文件的顶部或者紧接在使用字段的方法之前定义字段。

### 限制变量的作用域

尽可能缩小局部变量的作用域。这样做有助于提高代码的可读性和可维护性，并降低出错的可能性。在包含变量所有使用情形的最内层的块中声明每个变量。

首次使用局部变量时需进行声明。几乎每个局部变量声明都应该包含一个初始化程序。如果您拥有的信息尚不足以用来合理地初始化某个变量，请推迟到信息充足时再声明。

try-catch 语句是例外情况。如果通过一个会抛出受检异常的方法的返回值来初始化变量，则必须在 try 块中进行初始化。如果必须在 try 块之外使用该值，则必须在 try 块之前对其进行声明，这时候尚无法合理地初始化：

```java
// Instantiate class cl, which represents some sort of Set

Set s = null;
try {
    s = (Set) cl.newInstance();
} catch(IllegalAccessException e) {
    throw new IllegalArgumentException(cl + " not accessible");
} catch(InstantiationException e) {
    throw new IllegalArgumentException(cl + " not instantiable");
}

// Exercise the set
s.addAll(Arrays.asList(args));
```

不过，您可以通过将 try-catch 块封装在某个方法中来避免这种情况：

```java
Set createSet(Class cl) {
    // Instantiate class cl, which represents some sort of Set
    try {
        return (Set) cl.newInstance();
    } catch(IllegalAccessException e) {
        throw new IllegalArgumentException(cl + " not accessible");
    } catch(InstantiationException e) {
        throw new IllegalArgumentException(cl + " not instantiable");
    }
}

...

// Exercise the set
Set s = createSet(cl);
s.addAll(Arrays.asList(args));
```

在 for 语句本身中声明循环变量，除非有令人信服的理由不这么做：

```java
for (int i = 0; i < n; i++) {
    doSomething(i);
}
```

和

```java
for (Iterator i = c.iterator(); i.hasNext(); ) {
    doSomethingElse(i.next());
}
```

### 为 import 语句排序

import 语句的顺序为：

1. 导入 Android 包
2. 导入第三方包（`com`、`junit`、`net`、`org`）
3. `java` 和 `javax`

为了完全匹配 IDE 设置，导入顺序应为：

- 每个分组内按字母顺序排序，其中大写字母开头的语句位于小写字母开头的语句前面（例如 Z 在 a 前面）
- 每个主要分组（`android`、`com`、`junit`、`net`、`org`、`java`、`javax`）之间用空行隔开

最初对于语句顺序并没有样式要求，这意味着 IDE 经常会改变顺序，或者 IDE 开发者必须停用自动导入管理功能并手动维护导入语句。这样相当不方便。当提及 Java 样式时，开发者们喜欢的样式五花八门，最终简单归结为：针对 Android，只需“选择一种兼容一致的排序方式”。因此我们选择了一种样式，更新了样式指南，并让 IDE 遵循该指南。我们希望 IDE 用户在编写代码时，对所有软件包的导入都符合此模式，无需再进行额外的工程处理。

这种样式是按以下原则选取的：

- 用户希望首先看到的导入往往位于顶部 (`android`)。
- 用户最不希望看到的导入往往位于底部 (`java`)。
- 用户可以轻松遵循该样式。
- IDE 可以遵循该样式。

将静态导入置于所有其他导入之上（与常规导入一样的排序方式）。

### 使用空格缩进

我们使用四 (4) 个空格缩进块，绝不使用制表符。如果您有疑问，请与周围的代码保持一致。

我们使用八 (8) 个空格缩进自动换行，包括函数调用和赋值。

**推荐**

```java
Instrument i =
        someLongExpression(that, wouldNotFit, on, one, line);
```

**不推荐**

```java
Instrument i =
    someLongExpression(that, wouldNotFit, on, one, line);
```

### 遵循字段命名规范

- 非公共且非静态字段的名称以 `m` 开头。
- 静态字段的名称以 `s` 开头。
- 其他字段以小写字母开头。
- 公共静态 final 字段（常量）采用 `ALL_CAPS_WITH_UNDERSCORES` 形式。

例如：

```java
public class MyClass {
    public static final int SOME_CONSTANT = 42;
    public int publicField;
    private static MyClass sSingleton;
    int mPackagePrivate;
    private int mPrivate;
    protected int mProtected;
}
```

### 使用标准大括号样式

左大括号不单独占一行，而与其前面的代码位于同一行：

```java
class MyClass {
    int func() {
        if (something) {
            // ...
        } else if (somethingElse) {
            // ...
        } else {
            // ...
        }
    }
}
```

我们需要在条件语句周围添加大括号。例外情况：如果整个条件语句（条件和主体）适合放在同一行，那么您可以（但不是必须）将其全部放在一行上。例如，我们接受以下样式：

```java
if (condition) {
    body();
}
```

同样也接受以下样式：

```java
if (condition) body();
```

但不接受以下样式：

```java
if (condition)
    body();  // bad!
```

### 限制代码行长度

代码中每一行文本的长度都不应超过 <font color="#FF0000">100 </font>个字符。虽然关于此规则存在很多争论，但最终决定仍是以 <font color="#FF0000">100 </font>个字符为上限，不过存在以下例外情况：

- 如果注释行包含长度超过 <font color="#FF0000">100</font> 个字符的示例命令或文字网址，为了便于剪切和粘贴，该行可以超过 <font color="#FF0000">100 </font>个字符。
- 导入语句行可以超出此限制，因为用户很少会看到它们（这也简化了工具编写流程）。

### 使用标准 Java 注释

注释应该位于同一语言元素的其他修饰符之前。简单的标记注释（例如 `@Override`）可以与语言元素列在同一行。如果有多个注释或参数化注释，则应各占一行并按字母顺序排列。

Java 中 3 个预定义注释的 Android 标准做法如下：

- 如果不建议使用带注释的元素，请使用 `@Deprecated` 注释。如果您使用 `@Deprecated` 注释，则还必须为其添加 `@deprecated` Javadoc 标记，并且该标记应该指定一个替代实现方案。另请注意，`@Deprecated` 方法应该仍然可以使用。如果您看到带有 `@deprecated` Javadoc 标记的旧代码，请添加 `@Deprecated` 注释。

- 如果某个方法替换了父类中的声明或实现，请使用 `@Override` 注释。例如，如果您使用 `@inheritdocs` Javadoc 标记，并且派生于某个类（而非接口），则也必须为方法添加注释，说明该方法替换了父类的方法。

- 请仅在无法消除警告的情况下使用

   

  ```java
  @SuppressWarnings
  ```

   

  注释。如果某个警告通过了“无法消除”测试，那么必须使用

   

  ```java
  @SuppressWarnings
  ```

   

  注释，确保所有警告都反映出代码中的实际问题。

  

  当需要 `@SuppressWarnings` 注释时，必须在前面添加一个 `TODO` 注释，用于说明“无法消除”情况。这通常会标识出是哪个违规类使用了不合适的接口。例如：

  ```jaVa
  // TODO: The third-party class com.third.useful.Utility.rotate() needs generics
  @SuppressWarnings("generic-cast")
  List<String> blix = Utility.rotate(blax);
  ```

  当需要使用 `@SuppressWarnings` 注释时，请重构代码以分离出适用该注释的软件元素。

### 将首字母缩写词视为字词

在为变量、方法和类命名时，请将首字母缩写词和缩写形式视为字词，使名称更具可读性：

| 良好           | 不佳           |
| :------------- | :------------- |
| XmlHttpRequest | XMLHTTPRequest |
| getCustomerId  | getCustomerID  |
| class Html     | class HTML     |
| String url     | String URL     |
| long id        | long ID        |

由于 JDK 和 Android 代码库在首字母缩写词方面不一致，因此几乎不可能与周围的代码保持一致。因此，请始终将首字母缩写词视为字词。

### 使用 TODO 注释

为代码使用 `TODO` 注释是短期的临时解决方案，或者说此方法虽然足够好但并不完美。这些注释应包含全部大写的字符串 `TODO`，后跟一个英文冒号。

```java
// TODO: Remove this code after the UrlTable2 has been checked in.
```

和

```java
// TODO: Change this to use a flag instead of a constant.
```

如果您的 `TODO` 采用“在未来的某个日期做某事”的形式，请确保在其中包含一个具体日期（“在 2005 年 11 月前修复”）或者一个具体事件（“在所有生产环境合成器都可处理 V7 协议后移除此代码”）。

### 谨慎使用日志记录

虽然日志记录非常有必要，但对性能却有负面影响，如果不能保持一定程度的简洁性，就会失去实用性。日志记录工具提供以下 5 种不同级别的日志记录：

- `ERROR`：在出现极其严重的情况时使用，这种情况即是指，某些事件会导致用户可见的后果，如果不删除某些数据、卸载应用、擦除数据分区或重新刷写整个设备（或更糟），则无法恢复。系统一直会记录此级别的日志。最好向统计信息收集服务器报告能够说明 `ERROR` 级别的一些日志记录情况的问题。

- `WARNING`：在出现比较严重和意外的情况时使用，这种情况即是指，某些事件会导致用户可见的后果，但是通过执行某些明确的操作（等待或重启应用、重新下载新版应用或重新启动设备）可在不丢失数据的情况下恢复。系统一直会记录此级别的日志。可以考虑向统计信息收集服务器报告能够说明 `WARNING` 级别的一些日志记录情况的问题。

- `INFORMATIVE`：用于记录大多数人感兴趣的信息。换言之，当检测到某种情况会造成广泛的影响时，尽管不一定是错误，系统也会记录下来。这种情况应该仅由一个被视为该领域最具权威性的模块来记录（避免由非权威组件重复记录）。系统一直会记录此级别的日志。

- ```
  DEBUG
  ```

  ：用于进一步记录设备上发生的可能与调查和调试意外行为相关的情况。只记录收集有关组件情况的足够信息所需的信息。如果您的调试日志是主要日志，那么您应采用详细日志记录。

  系统会记录此级别的日志（即使在发布 build 中），并且周围要有 `if (LOCAL_LOG)` 或 `if LOCAL_LOGD)` 块，其中 `LOCAL_LOG[D]` 在您的类或子组件中定义。这样一来，就可以停用所有此类日志记录。因此，`if (LOCAL_LOG)` 块中不得包含有效逻辑。为日志构建的所有字符串也需要放在 `if (LOCAL_LOG)` 块中。如果日志记录调用会导致字符串构建发生在 `if (LOCAL_LOG)` 块之外，则不应将其重构为方法调用。

  有些代码仍然在使用 `if (localLOGV)`。虽然名称并不规范，但也可接受。

- `VERBOSE`：用于记录其他所有信息。系统仅针对调试 build 记录此级别的日志，并且周围要有 `if (LOCAL_LOGV)` 块（或同类块），以便能够默认进行编译。所有字符串构建都将从发布 build 中删除，并且需要在 `if (LOCAL_LOGV)` 块中显示。

#### 备注

- 在指定模块中，除了 `VERBOSE` 级别之外，一个错误应该尽可能只报告一次。在模块内的单个函数调用链中，只有最内层的函数应当返回错误，同一模块中的调用方只能添加一些明显有助于查明问题的日志记录。
- 在一个模块链中，除了 `VERBOSE` 级别之外，当较低级别的模块检测到来自较高级别模块的无效数据时，较低级别的模块应该只在 `DEBUG` 日志中记录该情况，并且仅当该日志提供的信息对调用方来说无法获取时进行记录。具体来说，当抛出异常时（异常中应该会包含所有相关信息）或者所记录的所有信息都包含在错误代码中时，则不需要记录此类情况。这在框架和应用之间的交互中尤为重要，而且由第三方应用造成的情况经过框架妥善处理后，不应该触发高于 `DEBUG` 级别的日志记录。应该触发 `INFORMATIVE` 级别或更高级别日志记录的唯一情况是，模块或应用在其自身级别或更低级别检测到错误。
- 当事实证明某些日志记录可能会发生多次时，最好实施一种频率限制机制，防止出现具有相同（或非常相似）信息的大量重复日志副本。
- 失去网络连接属于完全在预期之内的常见情况，没有必要记录下来。如果失去网络连接后导致在应用内出现某种后果，则应该记录为 `DEBUG` 或 `VERBOSE` 级别（具体取决于后果是否足够严重以及足够意外，足以记录在发布 build 中）。
- 如果在第三方应用可访问或代表第三方应用的文件系统上拥有完整的文件系统，则不应该记录高于 INFORMATIVE 级别的日志。
- 来自任何不受信任来源的无效数据（包括共享存储空间中的任何文件或通过网络连接获取的数据）被视为符合预期，在被检测到无效时不应触发高于 `DEBUG` 级别的任何日志记录（甚至应该尽可能地限制日志记录）。
- 针对 `String` 对象使用 `+` 运算符时，该运算符会隐式创建一个具有默认缓冲区大小（16 个字符）的 `StringBuilder` 实例，还可能会创建其他临时 `String` 对象。因此，显式创建 `StringBuilder` 对象并不比依赖默认的 `+` 运算符成本更高（实际上可能更高效）。请注意，即使没有读取日志信息，调用 `Log.v()` 的代码也会在发布 build 中进行编译和执行，包括构建字符串。
- 任何供其他人阅读并且出现在发布 build 中的日志记录都应当简洁易懂。这包括一直到 `DEBUG` 级别的所有日志记录。
- 请尽可能使日志记录保持在一行之内。一行长度在 80 个字符或 100 个字符内是可以接受的。请尽可能避免长度超过 130 个字符或 160 个字符（包括标记的长度）。
- 如果日志记录报告成功事件，切勿采用高于 `VERBOSE` 级别的日志记录。
- 如果使用临时日志记录诊断难以重现的问题，应采用 `DEBUG` 或 `VERBOSE` 级别，并且应当将其包裹在 if 块中，以便可在编译期间将其停用。
- 请务必谨慎，避免在日志中泄露安全方面的信息。避免记录隐私信息。尤其要避免记录有关受保护内容的信息。这在编写框架代码时尤为重要，因为事先无法轻易得知哪些是隐私信息或受保护的内容，哪些不是。
- 切勿使用 `System.out.println()`（或针对原生代码使用 `printf()`）。`System.out` 和 `System.err` 会重定向到 `/dev/null`，导致您的 print 语句不会产生可见效果。不过，为这些调用构建的所有字符串仍会得以执行。
- 日志记录的黄金法则是，您的日志不一定要将其他日志排挤出缓冲区，正如其他日志不会这样对待您的日志一样。

## Javatests 样式规则

请遵循测试方法的命名规范，并使用下划线将被测试的内容与被测试的具体案例区分开来。这种样式可让您更轻松地看出正在测试的案例。例如：

```
testMethod_specificCase1 testMethod_specificCase2void testIsDistinguishable_protanopia() {  ColorMatcher colorMatcher = new ColorMatcher(PROTANOPIA)  assertFalse(colorMatcher.isDistinguishable(Color.RED, Color.BLACK))  assertTrue(colorMatcher.isDistinguishable(Color.X, Color.Y))}
```