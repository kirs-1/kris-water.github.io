---
{"dg-publish":true,"permalink":"/Java/Generics/PECS/"}
---

> [!info]
> 这个概念是解释两种通配符的用法extends和super
> Producer Extends Consumer Super
> 意思是说extents通配符是生产者(只读不写),super是消费者(只写不读)
> 
> 

下面是具体的代码来解释这两种通配符
```java
public class PECS<T> {  
    private T a;  
    public PECS(T a) {  
        this.a = a;  
    }  
    public T get() {  
        return a;  
    }  
  
    public void set(T b) {  
        this.a = b;  
    }  
}  
  
// extends通配符  
class pe {  
    public static void main(String[] args) {  
        PECS<Double> p = new PECS<>(123.1);  
        int n = set(p);  
        System.out.println(n);  
    }  
  
    // 这里的<?extends Number>,表示这个方法可以接受Number或者Number的子类  
    static int set(PECS<? extends Number> p) {  
        // 但是get是没有问题的,因为上面不论传入什么类型,get方法返回的都是Number类型  
        Number a = p.get();  
        // 这里是会报错的,因为a.intValue() + 1这里的结果一定是int  
        // PECS<Double> p = new PECS<>(123.1);但是上面如果传入了其他类型(例如Double),  
        // 那么set方法预期的参数就是Double,传入Integer,那么就会报错  
        p.set(a.intValue() + 1);  
  
        return p.get().intValue();  
    }  
}  
  
class cs {  
    public static void main(String[] args) {  
        PECS<Double> p = new PECS<>(123.1);  
        Number n = p.get();  
        System.out.println(n);  
    }  
  
    // 这里的<?super Integer>,表示这个方法可以接受Integer或者Integer的父类  
    static Number get(PECS<? super Integer> p) {  
        // 这里会报错,因为上面可能会返回Object,因为Object也是Number的父类,所以如果想接收参数,只能写成Object  
        Number a = p.get();  
        return a;  
    }  
}
```