---
{"dg-publish":true,"permalink":"/Spring Boot/IoC/"}
---


# IoC是什么？
IoC全称Inversion of Control，直译为控制反转。相当于把创建对象的过程交给Spring容器（IoC容器），而不是程序员自己创建对象。

### 那么为什么需要IoC?
下面用代码示例来解释一下IoC的作用。

```java
// 假设有一个需求，是提供图书的信息  
class BookService {  
    // Hikari 是JDBC数据库连接池，我们以它来举例连接数据库的服务  
    // 没有IoC的情况下，如果需要查询到图书的信息，每次需要连接数据库，都需要实例化一次数据库连接实例  
    private HikariConfig config = new HikariConfig();  
    // 为了获取到数据库连接实例（dataSource），需要实例化一个数据库连接池实例  
    private DataSource dataSource = new HikariDataSource(config);  
  
//    public getBook(long bookId) {  
        // 这边处理连接数据库的逻辑  
//    }  
}  
  
class UserService {  
    // 同样的，获取用户信息同样需要在用户类中创建连接数据库的实例。  
    private HikariConfig config = new HikariConfig();  
    private DataSource dataSource = new HikariDataSource(config);  
    // 同样的也会有一个 getUser() 方法  
}  
  
// 如果有一个订单服务的需求，那么它会同时依赖用户和图书类，这就造成更多的重复性代码了  
// public class OrderService { }  
  
// 所以 IoC解决的问题是，创建对象的实例交给容器，程序员只需要使用，而不用关心创建实例的过程。  
// 这就是Spring 的IoC容器的作用。  
// Spring IoC容器只有一个，它可以创建很多Spring Bean对象，就是来解决每次手动创建实例的问题。  
  
// 加这个注解的意思是说将这个类交由Spring管理，然后创建的对象就叫Spring Bean对象  
// @Component是元注解，实际上项目中会使用@Controller、@Service、@Repository等派生注解  
@Component  
class BeanBookService {}  
@Component  
class BeanUserService {}  
  
// 为什么OrderService中依赖Book和User,还要在OrderService类添加@Component注解？  
// 我在下面的Spring Bean的创建流程中有解释。  
@Component  
// Lombok提供了一种更新的方式，只要在类前声明了这个注解，会自动生成构造函数注入未初始化（未显式提供初始值），  
// 带final字段和@NonNull字段的值，如book和user两个java bean。因为是自动生成了构造函数，  
// 就要求类中不能有构造函数。  
@RequiredArgsConstructor  
class OrderService {  
    // Spring的注入有三种方式，官方推荐构造函数注入  
    // 为什么加final,主要是为了编码安全，就是表示beanBookService是只能在构造函数执行完成之前赋值，  
    // 且不可修改  
    private final BeanBookService beanBookService;  
    private final BeanUserService beanUserService;  
    // 这个注解是Spring提供的，它可以自动注入对象实例。  
    // 在Spring4.3+（很老的版本了，现在基本都支持），如果只有一个构造函数，那么这个注解可以省略  
//    @Autowired  
//    OrderService(BeanBookService beanBookService, BeanUserService beanUserService) {  
//        this.beanBookService = beanBookService;  
//        this.beanUserService = beanUserService;  
//    }  
    // 也可以使用字段注入  
//    @Autowired  
//    private final BeanBookService beanBookService;  
    // 开发规范中并不推荐使用字段注入，推荐使用构造方法注入。因为构造方法注入会避免类的循环依赖，  
    // 就是A中注入了B，然后B中又注入了A，这种情况在构造函数注入的的情况下，在应用启动的时候就会报错  
}
```

---

# @Resource和@Autowired的对比

|对比维度|@Resource|@Autowired|
|---|---|---|
|基本作用|依赖注入（与 @Autowired 基本一致）|依赖注入|
|自动匹配策略|优先按 **名称** 匹配（byName）|优先按 **类型** 匹配（byType）|
|是否支持构造器注入|❌ 不支持|✅ 支持|
|规范归属|J2EE 规范（javax.annotation）|Spring 框架规范|

---

# Spring Bean 的完整创建流程

Spring Bean 的生命周期从类被扫描开始，到容器关闭时销毁结束。以下是其核心流转过程：

## 1. 扫描与注册 (Scanning & Registration)
动作：Spring 启动时扫描类路径，识别带有 @Component、@Service、@Controller 等注解的类。也会扫描到@Autowired，记录下要注入的字段。所以只有类标记了@Component，才会记录在实例化或属性填充的过程中注入Bean 对象。
产物：将这些类的信息封装成 BeanDefinition（元数据），并注册到容器中。  
意义：此时对象尚未创建，Spring 只是记录了“将来需要创建什么样的对象”。

## 2. 实例化 (Instantiation)
动作：容器根据 BeanDefinition 调用类的构造函数创建对象实例。  
底层机制：通常通过反射 (Constructor.newInstance) 完成。  
构造函数注入：若采用构造函数注入，Spring 会在此阶段解析依赖参数并传入。  

## 3. 属性填充 (Population)
动作：对象创建完成后，处理类中定义的依赖关系。  
底层机制：由 AutowiredAnnotationBeanPostProcessor 负责，通过反射实现：  
字段注入（@Autowired）：调用 Field.set() 强行赋值。  
Setter 注入：调用 Method.invoke() 执行设置方法。  
状态：此阶段结束后，Bean 的所有依赖（无论是构造还是字段注入）均已就位。

## 4. 初始化 (Initialization)
动作：执行一系列回调方法，标志着 Bean 已完全准备好投入使用。  
执行顺序：  
执行 @PostConstruct 标注的方法。  
执行 InitializingBean 接口的 afterPropertiesSet() 方法。  
执行自定义的 init-method。

## 5. 运行期 (Running)
状态：Bean 驻留在容器的单例池（Singleton Pool）中。  
特点：只要 JVM 不关闭，单例 Bean 就会一直常驻内存，随时响应业务调用。

## 6. 销毁 (Destruction)
触发时机：Spring 容器关闭时（如应用停机、调用 context.close()）。  
执行顺序：  
执行 @PreDestroy 标注的方法。  
执行 DisposableBean 接口的 destroy() 方法。  
执行自定义的 destroy-method。

## 💡 核心要点总结

| 阶段 | 关键动作 | 核心技术/注解 | 备注 |
|------|----------|----------------|------|
| 1. 扫描 | 生成元数据 | @Component, BeanDefinition | 对象的“出生证明” |
| 2. 实例化 | 创建对象壳子 | 反射 (Constructor.newInstance) | 构造函数注入在此发生 |
| 3. 属性填充 | 注入依赖 | 反射 (Field.set / Method.invoke) | 字段/Setter 注入在此发生 |
| 4. 初始化 | 最终检查 | @PostConstruct | 依赖已全部就绪 |
| 5. 销毁 | 释放资源 | @PreDestroy | 容器关闭时触发 |

> 专家提示：虽然实例化和属性填充底层都依赖反射，但 Spring 会通过缓存 BeanDefinition 中的构造函数信息来减少重复的逻辑判断，从而优化启动性能。

---

> [!tip] 扩展阅读：@Component及派生注解在项目中的真实作用  

### 📊 `@Component` 及其派生/组合注解完整对比表

|注解 (Annotation)|用途与语义 (Purpose & Semantics)|特殊处理 / 项目中的实际不同 (Special Handling & Practical Differences)|
|---|---|---|
|**`@Component`**|通用的Spring组件注解，是所有派生注解的元注解[](https://cloud.tencent.cn/developer/article/2223081?from=15425)。|最基本的组件标记，无任何特殊行为。Spring通过扫描将其注册为Bean[](https://worktile.com/kb/ask/841805.html#%3A~%3Atext%3D%E5%9C%A8Spring%E6%A1%86%E6%9E%B6%E4%B8%AD)。|
|**`@Service`**|标记业务逻辑层（Service层）的组件[](https://developer.aliyun.com/article/1218024)[](https://www.henghost.com/news/article/435254/)。|目前没有额外的特殊处理，主要用于**提升代码可读性**和**明确架构分层**[](https://www.php.cn/faq/2241377.html)。开发者常基于此注解编写AOP切面（如事务管理）[](https://www.spring-doc.cn/spring-framework/7.0.0-SNAPSHOT/core_beans_classpath-scanning.html)。|
|**`@Repository`**|标记数据访问层（DAO层）的组件[](https://developer.aliyun.com/article/1218024)[](https://www.henghost.com/news/article/435254/)。|Spring会为其提供**额外的异常转换功能**，能将数据库操作相关的特定异常（如Hibernate、JPA异常）统一转换为Spring的`DataAccessException`体系[](https://www.spring-doc.cn/spring-framework/7.0.0-SNAPSHOT/core_beans_classpath-scanning.html)[](https://www.php.cn/faq/2241377.html)。|
|**`@Controller`**|标记Spring MVC框架中的控制器组件[](https://developer.aliyun.com/article/1218024)[](https://www.henghost.com/news/article/435254/)。|会被Spring MVC的`RequestMappingHandlerMapping`扫描并处理，用于响应HTTP请求[](https://learnku.com/docs/springboot-like-laravel/spring-zhu-jie-da-quan?show_current_version=yes#topics-list)。通常与`@RequestMapping`等注解配合使用[](https://developer.aliyun.com/article/1218024)。|
|**`@RestController`**|用于编写RESTful API的控制器[](https://learnku.com/docs/springboot-like-laravel/spring-zhu-jie-da-quan?show_current_version=yes#topics-list)。|这是一个**组合注解**，它结合了`@Controller`和`@ResponseBody`的功能。这意味着该控制器所有方法的返回值都会被直接序列化为JSON/XML写入HTTP响应体，而不是返回一个视图（如JSP）[](https://developer.aliyun.com/article/1218024)。|
|**`@Configuration`**|标记一个类为Spring的配置类[](https://learnku.com/docs/springboot-like-laravel/spring-zhu-jie-da-quan?show_current_version=yes#topics-list)[](https://developer.aliyun.com/article/1218024)。|被标记的类本身也是一个会被注册到容器中的Bean[](https://worktile.com/kb/ask/841805.html#%3A~%3Atext%3D%E5%9C%A8Spring%E6%A1%86%E6%9E%B6%E4%B8%AD)[](https://developer.aliyun.com/article/1218024)。其内部包含了使用`@Bean`注解的方法，Spring会调用这些方法，并将返回值作为Bean注册到容器中[](https://developer.aliyun.com/article/1218024)。这是基于**Java配置**的核心。|
|**`@ControllerAdvice`**|作为所有`@Controller`的全局增强组件。|其类被`@Component`元注解标记，因此可被扫描注册。常与`@ExceptionHandler`、`@InitBinder`、`@ModelAttribute`等注解结合，为所有或部分控制器提供全局的异常处理、数据绑定和模型数据预处理功能。|
|**`@RestControllerAdvice`**|专用于RESTful控制器的全局增强组件。|这是一个**组合注解**，相当于`@ControllerAdvice`和`@ResponseBody`的组合。与`@ControllerAdvice`的唯一区别是，它确保所有`@ExceptionHandler`方法的返回值都会直接写入响应体（如返回JSON错误信息），而不是渲染为视图。|
|**`@JsonComponent`**|标记用于Jackson序列化/反序列化的组件。|注解本身使用了`@Component`元注解，因此会被自动扫描注册为Bean。Spring Boot会自动将所有`@JsonComponent`标注的Bean（如自定义的`JsonSerializer`和`JsonDeserializer`）注册到`ObjectMapper`中，实现全局的JSON自定义处理。|
|**`@Indexed`**|性能优化注解，本身不直接作为`@Component`的派生注解，但其功能与组件扫描息息相关。|`@Indexed`并不将类标记为Bean，而是作为**元注解**使用（例如添加到`@Component`或其他派生注解上）。它的作用是**加速组件扫描**。在编译时，它会生成`META-INF/spring.components`索引文件，避免了启动时的全类路径扫描，从而**提升应用的启动速度**。|

---

### 💡 核心要点总结

- **基础与派生**：`@Component`是Spring中所有组件注解的基石，所有派生注解都直接或间接地使用了它作为元注解[](https://cloud.tencent.cn/developer/article/2223081?from=15425)。
    
- **语义与作用**：`@Service`、`@Repository`、`@Controller`是`@Component`在分层架构下的语义化特例[](https://www.spring-doc.cn/spring-framework/7.0.0-SNAPSHOT/core_beans_classpath-scanning.html)。`@RestController`则是`@Controller`在REST场景下的功能性增强[](https://developer.aliyun.com/article/1218024)。
    
- **功能增强**：`@Repository`提供了异常转换，`@Configuration`定义了Java配置方式，而`@ControllerAdvice`/`@RestControllerAdvice`则提供了全局横切关注点的处理能力[](https://www.spring-doc.cn/spring-framework/7.0.0-SNAPSHOT/core_beans_classpath-scanning.html)。
    
- **性能提示**：`@Indexed`通过编译期生成索引文件来优化组件扫描的启动性能