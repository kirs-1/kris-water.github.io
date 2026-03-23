---
{"dg-publish":true,"permalink":"/Java/Collection/List/"}
---



### Collection
 >[!概念]
> - Collection是一个java对象中,包含其他的java对象,并提供对外的访问接口,这种对象被成为Collection.
> - 它包含三种类型
> - List 通过索引存取,大小可变(也可以生成固定长度的)  
> - Set (元素不重合)
> - Map 映射表

---
### 为什么需要List?

> [!概念]
> - List的作用是对数组的扩展,首先因为数组初始化后不能改变大小,其次是只能通过索引存取.

```java
class List1 {  
    public static void main (String[] args) {  
//        这是java9以上的写法,创建的是定长的List  
//        List<String> list = List.of("a", "b", "c");  
//        java8的写法是这样的  
//        List<String> list = Arrays.asList("a", "b", "c");  
//        为什么 List会出现定长的呢,List中明明有add,remove方法?  
//        因为List是接口,使用asList的时候,重写了add和remove方法  
//        长度可变的List写法  
        List<String> list1 = new ArrayList<>();  
        // 因为ArrayList是可变的,在需要扩容的时候,他会自动扩容到需要容纳的list大小,所以可以这样写  
        List<String> list2 = new ArrayList<>(Arrays.asList("a", "b", "c"));  
        // 第一种是显式调用ListIterator,ListIterator继承了Iterator,可以修改和添加元素,Iterator只能删除  
        // 因为不同的集合结构不同,所以有各种各样的遍历器,都继承于Iterator  
        for (ListIterator<String> it = list2.listIterator(); it.hasNext();){  
            // 因为ListIterator维护了一个游标,游标仅会在开头的左侧,结尾的右侧和两个元素中间,  
            // 所以.next会指向游标右侧的元素  
            // TIPS 为什么要使用游标?不能直接指向一个元素  
            // 如果指向的是当前的元素,那么插入一个数据后,是应该指向插入的数据,还是要指向之前的数据,会产生歧义  
            // 如果删除结尾的元素,那指向的位置会处于虚无状态,产生边界问题,虽然上面的问题都可以解决  
            // 但是游标模型的操作是最清晰的,且不会产生歧义,因为游标不指向任何一个元素,在元素删除或者添加时,  
            // 游标的移动逻辑不会引起歧义  
            String s = it.next();  
            it.set(s.toUpperCase());  
            // 注意,这里不能调用previous,因为前面游标已经向前挪动了,调用previous后,游标又挪回来了,无限循环  
            // 如果需要获取当前修改的元素,后面再加一个next挪动一下游标就可以了  
            // System.out.println(it.previous());  
            System.out.println(s);  
        }  
        // 这是Iterator的语法糖,但是也没有游标操作了,所以是单向流动的,很适合遍历操作  
        for (String s : list2) {  
            System.out.println(s);  
        }  
        // 也可以用传统的for循环来实现  
        for (int i = 0; i < list2.size(); i++) {  
            list2.set(i, list2.get(i).toLowerCase());  
            System.out.println(list2.get(i));  
        }  
        // equal 方法  
        List<person> list3 = new ArrayList<>();  
        list3.add(new person("A"));  
        list3.add(new person("B"));  
        list3.add(new person("C"));  
        System.out.println(list2.contains("a")); // true  
        System.out.println(list3.contains(new person("A"))); // false  
    }  
    // 之所以产生这种现象的原因是contains其实是继承了Object中的equals方法,这个方法是通过==来比较的,  
    // 其实是比较的内存地址,下面是Object中equals的实现方法,Java标准库提供的String、Integer等已经覆写了equals()方法  
    // 所以结果是true  
    public boolean equals(Object obj) {  
        return this == obj;  
    }  
  
}  
// 我们可以自己实现equals和hashCode方法(一般这两个方法要成对出现),因为java中规定,两个对象通过equals比较相等,  
// 那他们的hashCode必须返回相等的整数,一些collection会同时依赖两个方法来做判断,所以都要覆写  
// 下面这个是lombok实现的,它会扩展Java的内部注解,再编译中可以生成一些常用的代码,减少冗余代码  
// 下面这个注解的意思就是自动生成equals和hashCode方法  
// @EqualsAndHashCode  
class person {  
    public String name;  
    person (String initName) {  
        this.name = initName;  
    }  
}
```