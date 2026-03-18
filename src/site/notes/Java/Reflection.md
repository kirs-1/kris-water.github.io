---
{"dg-publish":true,"permalink":"/Java/Reflection/"}
---



首先会遇到两个让我感觉非常疑惑的概念，class 和 Class

## class

是指当 JVM 加载 .class 文件后生成的，把一个类的字段、方法、构造器等放入方法区中，形成了类的元数据 

###### 内容

- 类的全限定名、父类名、接口名
    
- 每个字段的名称、类型、修饰符
    
- 每个方法的名称、返回类型、参数列表、修饰符，以及方法的字节码指令
    
- 构造器信息（就是构造方法）
    
---

> [!tip] 扩展知识，什么是方法区？
> 方法区是 JVM 的一块逻辑内存区
>  ###### 内容
> - class(类的原数据)
> - 运行时常量池（每个类或接口都有一个运行时常量池，存放编译期生成的各种字面量和符号引用）
> - 即时编译（JIT）后的代码缓存：热点代码编译后的本地机器码
> - 类加载器引用：每个类都记录了是由哪个类加载器加载的
> - Class 对象的引用：方法区中存储了指向堆中 Class 对象的指针（或引用）
> ###### 作用
> - 因为 JVM 需要知道类的结构信息才能正确执行程序。
> - 例如，创建对象时需要知道对象的大小，调用方法时需要知道方法的字节码位置，这些信息都存储在方法区中。
> - 方法区是 JVM 实现动态链接、反射和多态的基础。
## Class

是在 JVM 生成方法区中的 class 时同步生成的，它放在堆区，与方法区中的 class 一一对应，它是对方法区中 class 的封装，  
目的是解决像反射、类型检查这种问题。

所以反射就是通过 Class 来获取类的信息  
###### 内容

- 类的基本信息：类名（getName()）、简单名（getSimpleName()）、包名（getPackage()）、修饰符（getModifiers()）等。
    
- 父类与接口：父类的 Class 对象（getSuperclass()），实现的接口 Class 数组（getInterfaces()）。
    
- 字段信息：可以通过 getFields()、getDeclaredFields() 获取 Field 对象数组，每个 Field 对象对应一个字段的元数据。
    
- 方法信息：通过 getMethods()、getDeclaredMethods() 获取 Method 对象数组。
    
- 构造器信息：通过 getConstructors() 等获取 Constructor 对象。
    
- 注解信息：获取类上的注解。
    
- 类加载器引用：加载该类的 ClassLoader。
    
- 其他运行时信息：如是否为接口、是否为枚举、是否为数组、是否为基本类型等（通过 isInterface()、isEnum()、isArray()、isPrimitive() 等方法判断）。
    

> [!note] 注意
> Class 对象本身并不包含方法的字节码指令或字段的默认值等细节（这些仍然在方法区或元空间中），但提供了反射入口来访问它们。

---

```java
import java.lang.reflect.Field;  
import java.lang.reflect.Method;  
  
class Reflection {  
    public String name;  
    private int age;  
    public Reflection (String name) {  
        this.name = name;  
        this.age = 18;  
    }  
    public String func(String arg) {  
        return arg;  
    }  
  
    private String priFunc(String arg) {  
        return arg;  
    }  
  
}  
  
class Main {  
    public static void main(String[] args) throws Exception{  
        Reflection r = new Reflection("k");  
        printClassInfo(Reflection.class);  
        printObjectInfo(r);  
    }  
  
    static void printClassInfo(Class cls) throws Exception {  
        System.out.println("Class name(完整的类名): " + cls.getName());  
        System.out.println("Simple name(单纯的类名): " + cls.getSimpleName());  
        if (cls.getPackage() != null) {  
            System.out.println("Package name: " + cls.getPackage().getName());  
        }  
        System.out.println("is interface: " + cls.isInterface());  
        System.out.println("is enum: " + cls.isEnum());  
        System.out.println("is array: " + cls.isArray());  
        System.out.println("is primitive(是否是基础类型): " + cls.isPrimitive());  
        // 获取当前对象的访问修饰符,返回的是int值  
        System.out.println("is getModifiers: " + cls.getModifiers());  
        System.out.println("is getSuperclass: " + cls.getSuperclass());  
        // 后续这些方法直接返回的是引用的内存地址,是字符串,如果需要了解具体内容,需要遍历  
        // 获取对象的所有public字段,包含继承字段  
        System.out.println("is getFields: " + cls.getFields());  
        System.out.println("=== Fields ===");  
        System.out.println("--- getFields() (public fields, including inherited) ---");  
        java.lang.reflect.Field[] fields = cls.getFields();  
        for (java.lang.reflect.Field field : fields) {  
            System.out.println("  Field: " + field.getType().getSimpleName() + " " + field.getName());  
        }  
        if (fields.length == 0) {  
            System.out.println("  (no public fields)");  
        }  
        // 获取对象所有的字段,不包含继承字段  
        // getDeclaredFields();  
        // 获取对象的所有public方法,包含继承方法  
        // getMethods();  
        // 获取对象的所有方法,不包含继承方法  
        // getDeclaredMethods();  
        // 获取构造器方法  
        // getConstructors();  
    }  
  
    static void printObjectInfo(Object r) throws Exception{  
        Class obj = r.getClass();  
        System.out.println("字段name的值: " + obj.getField("name").get(r));  
        // Field类,是为了表示反射类中的字段的元数据  
        Field privateField = obj.getDeclaredField("age");  
        // 如果要访问私有成员变量,需要增加setAccessible,这样做是为了安全,防止私有变量被随意修改  
        // 也可以使用getFields先遍历然后依次调用可访问权限的方法即可  
        privateField.setAccessible(true);  
        // 如果要修改字段值,可以通过set方法(instance, value)  
        privateField.set(r, 22);  
        System.out.println("字段age的值: " + privateField.get(r));  
        // 如果要调用方法,同样也要根据访问修饰符来,如果是public,getMethod('成员方法名称', 成员方法的参数)  
        // 因为String是一个类型,类型不能直接作为参数传递,加上.class之后相当于传递了这个类型值本身  
        // Field类,是为了表示反射类中的方法的元数据  
        Method publicMethod = obj.getMethod("func", String.class);  
        // 为什么这里要强制转换,因为反射是运行时执行的,因为无法确认执行后的结果是什么类型,所以反射的方法都是返回OBJ  
        String publicMethodReturn = (String) publicMethod.invoke(r, "a");  
        System.out.println("方法func的值: " + publicMethodReturn);  
        Method privateMethod = obj.getDeclaredMethod("priFunc", String.class);  
        privateMethod.setAccessible(true);  
        String privateMethodReturn = (String) privateMethod.invoke(r, "b");  
        System.out.println("方法priFunc的值: " + privateMethodReturn);  
    }
```

---
- `- [ ] 反射还有很多需要学习的,比如使用构造方法,获取继承关系等`

