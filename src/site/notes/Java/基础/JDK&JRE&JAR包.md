---
{"dg-publish":true,"permalink":"/Java/基础/JDK&JRE&JAR包/"}
---


### 一、JDK JRE JVM他们的关系是什么？

| 概念      | 全称                       | 包含内容                                        | 用途                    |
| ------- | ------------------------ | ------------------------------------------- | --------------------- |
| **JDK** | Java Development Kit     | JRE + 开发工具（`javac`、`jar`、`javadoc`、`jdb` 等） | 编写、编译、调试 Java 程序      |
| **JRE** | Java Runtime Environment | JVM（Java 虚拟机）、核心类库（`rt.jar` 等）、支持文件         | 运行已编译的 Java 程序        |
| **JVM** | Java Virtual Machine     |                                             | 虚拟机，负责执行 `.class` 字节码 |

**关系**：JDK 包含 JRE，JRE包含JVM。如果你只需要运行 Java 程序，安装 JRE 就够了；如果要开发、编译 Java 程序，则需要 JDK。

### 二、JAR包包含什么？

jar包是.class文件以及相关的资源文件的集合，可以理解为.class文件+元数据

### 三、JDK官方类库包含什么？

根据OpenJDK官方文档的定义，Java核心类库包含的内容非常广泛[](https://openjdk.org/groups/core-libs/index.html)[](https://wiki.openjdk.org/pages/diffpagesbyversion.action?pageId=17957159&selectedPageVersions=9&selectedPageVersions=10)。除了我们日常编码直接使用的`java.lang`、`java.util`等包，它还包括了：

#### 1. 语言和虚拟机核心 (`java.lang`)

这是Java程序的基础，提供了最核心的类，如：

- `Object`：所有类的父类。
    
- `Class`：代表正在运行的类或接口。
    
- `String`、`StringBuffer`、`StringBuilder`：字符串操作。
    
- `Thread`：多线程支持。
    
- 基本类型的包装类：`Integer`、`Long`、`Boolean`等。
    
- 异常体系：`Throwable`、`Exception`、`RuntimeException`[](https://openjdk.org/groups/core-libs/index.html)[](https://cloud.tencent.cn/developer/article/1642713?from=15425)。
    

这部分是实现Java“官方语法”的底层支持，比如你在代码中写 `String s = "hello";`，背后调用的就是`java.lang.String`这个类。

#### 2. 输入/输出 (`java.io`)

包含处理文件、网络和其他数据流的类，如`File`、`InputStream`、`OutputStream`等[](https://openjdk.org/groups/core-libs/index.html)。

#### 3. 实用工具 (`java.util`)

提供了我们最常使用的集合框架（`List`、`Map`、`Set`）、日期时间工具、随机数生成器、正则表达式等[](https://openjdk.org/groups/core-libs/index.html)[](https://cloud.tencent.cn/developer/article/1642713?from=15425)。

#### 4. 并发工具 (`java.util.concurrent`)

在JDK 1.5时引入，提供了强大的并发编程工具，如线程池`ThreadPoolExecutor`、并发集合`ConcurrentHashMap`、锁`ReentrantLock`等。这部分是为了应对多核处理器时代的挑战而专门设计的[](https://cloud.tencent.cn/developer/article/1642713?from=15425)。

#### 5. 网络 (`java.net`)

用于网络编程的类，如`Socket`、`ServerSocket`、`URL`、`HttpURLConnection`等[](https://developer.aliyun.com/article/849900)。

#### 6. 安全 (`java.security`)

提供了Java安全框架的类和接口，用于控制代码执行权限、加密解密等[](https://developer.aliyun.com/article/849900)。

#### 7. 其他功能包

- `java.sql`：用于数据库访问的JDBC API[](https://developer.aliyun.com/article/849900)。
    
- `java.time`：JDK 8引入的全新日期和时间API，用于替代旧的`java.util.Date`和`Calendar`[](https://developer.aliyun.com/article/849900)。
    
- `java.nio`：新的输入/输出API，用于更高效的文件和网络操作[](https://openjdk.org/groups/core-libs/index.html)。
    
- `javax.*`：标准扩展包，最初作为`java.*`的扩展，但现在很多已成为核心API的一部分，如`javax.servlet`（用于Web开发）[](https://m.yisu.com/zixun/128899.html)。
    

---

### 四、JAVA语言和官方API(JAVA API)存放在哪里？

这是一个很好的问题，但我们需要先澄清“官方语法”和“API”这两个概念。

- **Java语言**是规范定义的**关键字**（如`class`、`if`、`for`）**数据类型**、**运算符**和**语法规则**等。这些是JVM和编译器（`javac`）需要解析和执行的指令，它们被实现在JVM的底层代码中，而不是以JAR包的形式提供。
    
- **“官方API”** 指的是JDK提供的一系列**现成的、编译好的类库**，也就是上面提到的`java.lang.*`、`java.util.*`等。这些是我们程序员可以直接在代码中调用的“工具”，它们被打包在JDK或JRE安装目录下的特定JAR文件中。**
    

#### 1. Java 8及之前版本：`rt.jar` 是绝对的核心

在Java 8及之前的版本中，所有核心的Java API都打包在一个巨大的JAR文件里，它就是 **`rt.jar`**（Runtime JAR）[](https://www.silicloud.com/zh/blog/java%e6%a0%b8%e5%bf%83api/)。

- **位置**：在`$JAVA_HOME/jre/lib/rt.jar`。
    
- **内容**：`java.lang.*`、`java.util.*`、`java.io.*`、`java.net.*`、`java.sql.*`...几乎所有你熟悉的`java.`和`javax.`开头的包，都封装在这个文件里[](https://www.silicloud.com/zh/blog/java%e6%a0%b8%e5%bf%83api/)。
    
- **源码**：JDK安装目录下还有一个`src.zip`文件，里面就是`rt.jar`中所有类的源代码，供开发者阅读[](https://www.silicloud.com/zh/blog/java%e6%a0%b8%e5%bf%83api/)。
    

#### 2. Java 9及之后版本：模块化，`rt.jar`消失了

从Java 9开始，为了支持模块化（Project Jigsaw），JDK进行了重构，核心类库被拆分成多个**模块**，存放在`$JAVA_HOME/jmods/`目录下，例如：

- `java.base.jmod`：包含了`java.lang`、`java.util`等最基础的模块。
    
- `java.sql.jmod`：包含了JDBC相关API。
    
- `java.xml.jmod`：包含了XML处理相关API。
    

这么做的好处是，你可以只打包自己应用用到的模块，让程序变得更轻量。

#### 3. JAR包里还有什么？`META-INF`目录的秘密

无论是`rt.jar`还是现代的模块化JAR，它们的内部结构除了`.class`文件，还有一个特殊的文件夹 **`META-INF`**，它包含了JAR包的元数据[](https://cloud.tencent.com.cn/developer/article/1687999?from=15425)。

- **`MANIFEST.MF`**：最重要的文件，定义了JAR包的版本、入口点（`Main-Class`）、类路径等。如果它是一个可执行JAR，就需要在这里指定`Main-Class`。
    
- **签名文件**：`.SF`和`.DSA`/`.RSA`文件，用于验证JAR包的来源和完整性，确保它未被篡改[](https://cloud.tencent.com.cn/developer/article/1687999?from=15425)。
    
- **`services/` 目录**：配合`ServiceLoader`机制使用，用于实现插件化架构。例如，JDBC驱动的`java.sql.Driver`实现类，就是通过这种方式被自动发现的[](https://cloud.tencent.com.cn/developer/article/1687999?from=15425)。
    

---

### 三、一个重要的补充：内部API vs 官方API

在JDK的类库中，还有一部分是供JDK内部使用，或供特定工具（如`javac`）使用的API，它们通常位于`com.sun.*`、`sun.*`包下。

- **官方API**：位于`java.*`、`javax.*`下，是Oracle承诺长期支持和兼容的，你可以放心使用。
    
- **内部API**：如`sun.misc.Unsafe`、`com.sun.net.ssl`等，是JDK的实现细节，**强烈不推荐在你的应用代码中直接使用**[](https://wiki.openjdk.org/pages/viewpage.action?pageId=20416085&navigatingVersions=true)[](https://wiki.openjdk.org/pages/diffpagesbyversion.action?pageId=17957128&originalVersion=6&revisedVersion=62)。因为这些类不保证跨版本兼容，可能在未来的JDK版本中被移除或修改。如果你使用了它们，升级JDK时程序就很可能会崩溃。
    

### 总结

为了让你看得更清楚，我用一个表格来汇总：

|内容分类|具体例子|在哪个包里 (Java 8)|
|---|---|---|
|**官方语法**|`class`, `if`, `for`, `interface`|由JVM直接支持，不在JAR包中|
|**核心API**|`java.lang.String`, `java.util.ArrayList`|`rt.jar` (位于 `jre/lib/`)|
|**扩展API**|`javax.servlet.*`, `javax.sql.*`|`rt.jar` (大部分已纳入)|
|**JAR元数据**|`META-INF/MANIFEST.MF`|每个JAR包内部|
|**内部API (禁止使用)**|`sun.misc.Unsafe`, `com.sun.*`|`rt.jar` 中，但官方禁止直接调用|

所以，当Maven执行`compile`时，它会自动从JDK的`rt.jar`（或模块化路径）里找到`java.lang.String`、`java.util.ArrayList`等类，让你可以正常使用它们。