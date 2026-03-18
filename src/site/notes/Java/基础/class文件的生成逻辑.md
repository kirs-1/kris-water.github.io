---
{"dg-publish":true,"permalink":"/Java/基础/class文件的生成逻辑/"}
---

>[!note]
>这是在学习class和Class[[Java/Reflection\|Reflection]]中牵扯出来的概念,Java真的挺深的,我学到哪就说到哪儿吧
>
###### .java文件
.java 文件就是人类阅读的程序源代码,也就是编译前的文件

---
###### .class文件
.class 就是编译后的字节码文件,给JVM用的,它的数量是由类的数量来决定的.
类并不只有 class XX{} 这种顶层类,如下的情况都会生成一个.class文件.
生成的.class全名中,是包含包名的,就是包名.类名.class,所以.class文件不会重名,后续代码会省略包名的方式表示.

```java
// 顶层类  
class A { }  
class B { }  
public class classFileAndJavaFile { }  
  
// 编译后生成 A.class B.class classFileAndJavaFile.class  
//内部类  
//public class Outer {  
//    class Inner { }           // 成员内部类  
//    static class StaticNested { } // 静态嵌套类  
//    void method() {  
//        class Local { }       // 局部内部类（方法内）  
//    }  
//}  
  
// 编译后生成 Outer.class Outer$Inner.class Outer$StaticNested.class Outer$1Local.class  
// 匿名类  
//public class Test {  
//    Runnable r = new Runnable() {  
//        public void run() { }  
//    };  
//}  
  
// 编译后生成 Test.class Test$1.class// .class名称中带$带数字是javac(java编译器即javac的要求)
```

