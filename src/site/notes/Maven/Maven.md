---
{"dg-publish":true,"permalink":"/Maven/Maven/"}
---

> [!info] 学习原因  
> 学习maven的scope参数时，花了一些时间才搞明白这几个参数究竟有什么区别，而且看完还是有点云里雾里，所以回来学习一下maven的生命周期，更有利于理解这几个参数的意义和区别吧。

# 为什么需要Maven?
- 可以解决包之间的依赖关系


---
# Maven 生命周期详解

## 一、Maven 的三大生命周期

Maven 有三套相互独立的生命周期：

1. **clean**：清理项目，删除 `target` 目录等。
    
2. **default**：构建项目，包括编译、测试、打包、部署等核心步骤。
    
3. **site**：生成项目报告、站点文档。
    

每个生命周期由多个**阶段（phase）** 组成，阶段之间有先后顺序。执行某个阶段时，会**自动执行该阶段之前的所有阶段，此处我们重点关注default生命周期**。

---

## 二、default 生命周期的主要阶段（按顺序）

| 阶段         | 描述                                  | 与依赖范围的关系                                                  |
| ---------- | ----------------------------------- | --------------------------------------------------------- |
| `validate` | 验证项目配置是否正确，必要信息是否齐全                 | -                                                         |
| `compile`  | 编译主代码（`src/main/java(rget/classes`） | **编译 classpath** 包含：`compile` + `provided` 范围的依赖          |
| `test`     | 执行单元测试（但不打包）                        | **测试 classpath** 包含：`compile` + `provided` + `test` 范围的依赖 |
| `package`  | 将编译后的代码打包成 JAR、WAR 等                | 打包时，`compile` 和 `runtime` 依赖会被复制进最终包                      |
| `verify`   | 运行集成测试，检查包是否符合质量标准                  | -                                                         |
| `install`  | 将包安装到本地仓库（供其他项目使用）                  | -                                                         |
| `deploy`   | 将包部署到远程仓库（供团队共享）                    | -                                                         |

**compile阶段具体的运行步骤**：

1. Maven 会调用 `maven-compiler-plugin` 插件，它会将存放项目自身的 `.class` 文件和你声明的第三方依赖（如 Spring、JDBC 驱动）。 并解析 `pom.xml`，自动将它们组合成 classpath 传给 `javac`。而JAVA API是在 javac 中的bootclasspath，它本身存在，且查找时优先级更高。
    
2. 该插件在内部调用 JDK 中的 `javac` 编译器（它包含了java语言）。
    
3. `javac` 对 `src/main/java` 下的所有 `.java` 源文件进行：
    
    - **语法检查**（关键词、括号匹配等）
        
    - **语义检查**（类型是否正确、方法是否存在、访问权限等）
        
    - **依赖检查**（引用的类是否在 classpath 中能找到）
        
4. 如果检查全部通过，`javac` 生成 `.class` 字节码文件到 `target/classes`；如果任何一项检查失败，`compile` 阶段就会报错并终止。

**注意**：每个阶段都会使用该阶段对应的 classpath（编译、测试、运行等），这些 classpath 的构成由依赖范围决定。

---

## 三、依赖范围如何影响各阶段

| scope    | 编译 classpath | 测试 classpath | 运行 classpath(JVM运行阶段) | 打包是否包含 |
| -------- | ------------ | ------------ | --------------------- | ------ |
| compile  | ✅            | ✅            | ✅                     | ✅      |
| provided | ✅            | ✅            | ❌（由环境提供）              | ❌      |
| runtime  | ❌            | ✅            | ✅                     | ✅      |
| test     | ❌            | ✅            | ❌                     | ❌      |

**以 `compile` 阶段为例**：Maven 构建编译 classpath 时，会加入 `compile` 和 `provided` 范围的依赖。此时 `runtime` 和 `test` 依赖不在 classpath 中，所以如果你的代码直接 import 了某个 `runtime` 依赖的类，编译会报错。

**以 `test` 阶段为例**：Maven 构建测试 classpath 时，会加入 `compile` + `provided` + `test` 所有依赖。因此测试代码可以使用 JUnit（`test`）和 MySQL 驱动（`runtime`），也能使用 Spring（`compile`）。

**runtime存在的必要性（以mysql包举例）：** 
- 强制开发者使用面向接口编程：如果填写为compile，开发者在代码中import了特定的类，这会造成代码之间的强耦合，减少了代码的可移植性。填写runtime，在编译期间检查到引入后，会直接报错
- 去除控制传递依赖：如果模块A依赖了mysql，B依赖了A，虽然B不一定使用了Mysql，但是B的classpath中也会增加mysql包。**这里就出现了两个大坑！什么是模块？classpath在编译过程中和运行过程中不一样！**

### 模块：
指一个maven项目，就是一个项目会有一个.pom文件。一个项目中可以有多个模块。

**classpath在编译和运行的空间不同！**
###### 为什么编译的classpath是独立的？
- 为了解决多个模块会依赖相同的包但是版本不同的问题，所以才会出现上面的问题，模块A加载了mysql，模块B依赖了模块A，所以B的classpath中也有mysql，而且maven编译的时候，会提示两个模块依赖了相同的包但是版本不同，可以及时处理。
- 如果编译时的classpath是扁平的，会出现模块A依赖的包实际上没有在pom文件中配置，但是模块B配置了。独立编译A的时候就会出现问题。

###### 为什么运行的classpath又是合并的呢？
在JVM中，**所有类都是通过类加载器加载到内存的，同一个类只会被加载一次**（由同一个类加载器）。因此，即使你的应用由多个 Maven 模块组成，它们在运行时往往会被合并到一个 classpath 中（例如所有模块的 jar 都在 classpath 下），相同的第三方类库不会重复加载。

Java 的**类加载器（ClassLoader）** 采用**双亲委派模型**：

- 系统类加载器（`AppClassLoader`）负责加载 classpath 下的类。
    
- 当需要加载一个类（如 `org.springframework.core.io.Resource`）时，类加载器会先委托给父加载器（`ExtClassLoader`，再到 `BootstrapClassLoader`），如果父加载器找不到，才由自己加载。
    
- 因此，**无论 classpath 中有多少个模块，只要它们通过同一个类加载器加载，每个类在内存中只有一份**。
    

如果你将多个模块的 JAR 都放在 classpath 中，JVM 只会把每个类加载一次，不会因为多个 JAR 中包含同一个类而重复加载（实际上，如果同一个类的不同版本出现在 classpath 中，会出现“类冲突”问题。

如何解决类冲突问题？Maven使用了冲突仲裁机制
1. 就近原则：路径短的优先。
2. 最先声明原则：路径一样长时，谁先在 `pom.xml` 里写谁优先。


  
---

## 四、生命周期与插件绑定

Maven 本身不执行编译、测试等任务，而是通过**插件目标（plugin goal）** 绑定到生命周期阶段来实现。

例如：

- `maven-compiler-plugin:compile` 绑定到 `compile` 阶段。
    
- `maven-surefire-plugin:test` 绑定到 `test` 阶段。
    
- `maven-jar-plugin:jar` 绑定到 `package` 阶段。
    

你可以通过 `mvn clean compile` 只执行 clean 生命周期 + default 生命周期的 compile 阶段（及之前所有阶段）。

---

## 五、为什么理解生命周期有助于理解依赖范围？

- **编译、测试、运行时使用不同的 classpath**，依赖范围正是用来控制依赖出现在哪个 classpath 中。
    
- 生命周期决定了这些 classpath 的构建时机和组成。
    
- 当你想知道一个依赖为什么在测试时可用、但在运行时却报错时，就去看它的 scope 和生命周期阶段的关系。
    

---

## 六、结合具体POM文件的理解

假设 `pom.xml` 中有以下依赖：

```xml
<project>
<!--  Maven有继承的机制，比如说子项目的 lombok都是一个版本的，那就可以写在父包的配置文件中，
这样子包只需要继承就可以了  -->
    <parent>
        <artifactId>bc-tax-ai-hub</artifactId>
        <groupId>com.bosssoft.taxjoy.bc</groupId>
        <version>1.0.0-SNAPSHOT</version>
    </parent>

    <modelVersion>4.0.0</modelVersion>

    <artifactId>bc-tax-ai-hub-api</artifactId>
    <groupId>org.example</groupId>
    <artifactId>test</artifactId>
    <version>1.0-SNAPSHOT</version>

    <properties>
<!--        这是设置编译器能接受的源码版本-->
        <maven.compiler.source>8</maven.compiler.source>
<!--        这是编译后生成的字节码版本-->
        <maven.compiler.target>8</maven.compiler.target>
<!--        java 9后还增加了一个release,是限定了API的版本，防止在源码中增加了高于生成的字节码版本的API，-->
<!--        这样在编译阶段就能检查到了-->
<!--        但是，即使是把源码版本和API版本都限定了，使用高版本的语法仍然存在危险，因为release只限制了官方的API，-->
<!--        还有三方库的引用以及不同版本的序列化反序列化结果不同等问题还是会存在，所以升级升级仍然具有风险，-->
<!--        release只是减少一些风险而已。-->
        <maven.compiler.release>9</maven.compiler.release>
        <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
    </properties>
    <!-- 从这里可以看到，dependencies和build有相同的结构，但是为什么要分开呢？  -->
    <!--  dependencies是依赖关系，build是构建信息，他们两个在生命周期的位置不同。  -->
    <!-- 把他们写成相同结构的原因是Maven认为他们都是包，不管是在什么阶段的，这是一种模型统一的设计理念。 -->
    <dependencies>
        <dependency>
<!--            groupId+artifactId+version构成了唯一ID，Maven规定了包一经发布，不允许修改。如要需要更改，
只有发布新版本，又称为GAV坐标-->
<!--            组织名称-->
            <groupId>org.projectlombok</groupId>
<!--            包名-->
            <artifactId>lombok</artifactId>
<!--            版本号-->
            <version>1.18.24</version>
<!--            这是依赖关系，依赖关系分为4种-->
<!--            compile 编译时需要，这是默认项，默认打包-->
<!--            test 会在test阶段放入classpath中-->
<!--            runtime 编译时不需要，运行时需要 （比如在项目中没有引用或者声明mysql包中的任意类型，-->
<!--            所以编译时不会检查，也不会加入classpath。但是实际项目运行中，是需要这个jar包的，所以也是会打包的。）-->
<!--            provided 编译时需要，运行时不需要 （lombok就是个例子，会在编译过程中通过注解处理器自动生成代码，-->
<!--            运行时就不需要把它放入jar包了）-->
            <scope>provided</scope>
        </dependency>
    </dependencies>
    <build>
        <plugins>
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-compiler-plugin</artifactId>
                <configuration>
                </configuration>
            </plugin>
        </plugins>
    </build>

</project>
```

