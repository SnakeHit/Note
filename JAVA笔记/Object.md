## toString 方法

使用`System.out.println()`打印一个对象时，如果没有重写`toString()`方法，则会输出对象的路径@哈希值。如果重写了`toString()`方法（==包括从父类继承下的方法==），则会按照给定的方式输出。

## equals()方法

```java
    /*
        Object类的equals方法,默认比较的是两个对象的地址值,没有意义
        所以我们要重写equals方法,比较两个对象的属性(name,age)
        问题:
            隐含着一个多态
            多态的弊端:无法使用子类特有的内容(属性和方法)
            Object obj = p2 = new Person("古力娜扎",19);
            解决:可以使用向下转型(强转)把obj类型转换为Person
     */
    /*@Override
    public boolean equals(Object obj) {
        //增加一个判断,传递的参数obj如果是this本身,直接返回true,提高程序的效率
        if(obj==this){
            return true;
        }

        //增加一个判断,传递的参数obj如果是null,直接返回false,提高程序的效率
        if(obj==null){
            return false;
        }

        //增加一个判断,防止类型转换一次ClassCastException
        if(obj instanceof Person){
            //使用向下转型,把obj转换为Person类型
            Person p = (Person)obj;
            //比较两个对象的属性,一个对象是this(p1),一个对象是p(obj->p2)
            boolean b = this.name.equals(p.name) && this.age==p.age;
            return b;
        }
        //不是Person类型直接返回false
        return false;
    }*/
    @Override
    public boolean equals(Object o) {
        if (this == o) return true;
        //getClass() != o.getClass() 使用反射技术,判断o是否是Person类型  等效于 obj instanceof Person
        if (o == null || getClass() != o.getClass()) return false;
        com.itheima.demo01.Object.Person person = (com.itheima.demo01.Object.Person) o;
        return age == person.age &&
                Objects.equals(name, person.name);
    }
```

因为`Object`类是所有类的父类，所以所有类都继承了`Object`类的方法。其中就包括`equal()`方法，而`euqal()`方法内的参数是`Object类型`的，所以需要对其进行向下转型。这里容易产生异常，因为可能传递过来的对象不是向下转型后的对象的类型，就会出现`ClassCastException`。所以应该做一些保护性的判断语句。通过`Alt+insert`自动生成`equals()`的构造对象。

## Objects类

在刚才IDEA自动重写equals代码中，使用到了`java.util.Objects`类，那么这个类是什么呢？

在**JDK7**添加了一个Objects工具类，它提供了一些方法来操作对象，它由一些静态的实用方法组成，这些方法是null-save（空指针安全的）或null-tolerant（容忍空指针的），用于计算对象的hashcode、返回对象的字符串表示形式、比较两个对象。

在比较两个对象的时候，Object的equals方法容易抛出空指针异常，而Objects类中的equals方法就优化了这个问题。方法如下：

* `public static boolean equals(Object a, Object b)`:判断两个对象是否相等。

我们可以查看一下源码，学习一下：

~~~java
public static boolean equals(Object a, Object b) {  
    return (a == b) || (a != null && a.equals(b));  
}
~~~

