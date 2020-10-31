# lambda

> `lambda表达式`是一个可传递的代码块

函数式接口必须有且只有一个抽象方法，其目的是不产生歧义以及方便重写方法。

```java
package lambda;
 
import java.util.ArrayList;
import java.util.List;
import java.util.Random;
 
import charactor.Hero;
 
public class TestLamdba {
    public static void main(String[] args) {
        Random r = new Random();
        List<Hero> heros = new ArrayList<Hero>();
        for (int i = 0; i < 5; i++) {
            heros.add(new Hero("hero " + i, r.nextInt(1000), r.nextInt(100)));
        }
        System.out.println("初始化后的集合：");
        System.out.println(heros);
        System.out.println("使用匿名类的方式，筛选出 hp>100 && damange<50的英雄");
        // 匿名类的正常写法
        HeroChecker c1 = new HeroChecker() {
            @Override
            public boolean test(Hero h) {
                return (h.hp > 100 && h.damage < 50);
            }
        };
        // 把new HeroChcekcer，方法名，方法返回类型信息去掉
        // 只保留方法参数和方法体
        // 参数和方法体之间加上符号 ->
        HeroChecker c2 = (Hero h) -> {
            return h.hp > 100 && h.damage < 50;
        };
 
        // 把return和{}去掉
        HeroChecker c3 = (Hero h) -> h.hp > 100 && h.damage < 50;
 
        // 把 参数类型和圆括号去掉
        HeroChecker c4 = h -> h.hp > 100 && h.damage < 50;
 
        // 把c4作为参数传递进去
        filter(heros, c4);
         
        // 直接把表达式传递进去
        filter(heros, h -> h.hp > 100 && h.damage < 50);
    }
 
    private static void filter(List<Hero> heros, HeroChecker checker) {
        for (Hero hero : heros) {
            if (checker.test(hero))
                System.out.print(hero);
        }
    }
 
}
```

从这个代码可以看出，我们的目的是为了向`filter()`中传递一个代码块（==方法==）。而`java`是面向对象的语言，任何方法或者代码块必须包含在对象当中。所以就有了一下的变革。

#### 普通方法

```java
package lambda;
  
import java.util.ArrayList;
import java.util.List;
import java.util.Random;
  
import charactor.Hero;
  
public class TestLambda {
    public static void main(String[] args) {
        Random r = new Random();
        List<Hero> heros = new ArrayList<Hero>();
        for (int i = 0; i < 10; i++) {
            heros.add(new Hero("hero " + i, r.nextInt(1000), r.nextInt(100)));
        }
        System.out.println("初始化后的集合：");
        System.out.println(heros);
        System.out.println("筛选出 hp>100 && damange<50的英雄");
        filter(heros);
    }
  
    private static void filter(List<Hero> heros) {
        for (Hero hero : heros) {
            if(hero.hp>100 && hero.damage<50)
                System.out.print(hero);
        }
    }
  
}
```

在普通方法中，`filter()`的功能是唯一不可变的，如果用户（实例）想要实现其他的功能，必须事先实现完毕，及其不方便。

#### 匿名类方法

```java
package lambda;
   
import java.util.ArrayList;
import java.util.List;
import java.util.Random;
   
import charactor.Hero;
   
public class TestLambda {
    public static void main(String[] args) {
        Random r = new Random();
        List<Hero> heros = new ArrayList<Hero>();
        for (int i = 0; i < 5; i++) {
            heros.add(new Hero("hero " + i, r.nextInt(1000), r.nextInt(100)));
        }
        System.out.println("初始化后的集合：");
        System.out.println(heros);
        System.out.println("使用匿名类的方式，筛选出 hp>100 && damange<50的英雄");
        HeroChecker checker = new HeroChecker() {
            @Override
            public boolean test(Hero h) {
                return (h.hp>100 && h.damage<50);
            }
        };
           
        filter(heros,checker);
    }
   
    private static void filter(List<Hero> heros,HeroChecker checker) {
        for (Hero hero : heros) {
            if(checker.test(hero))
                System.out.print(hero);
        }
    }
   
}
```

从匿名类的实现可以看出，`filter()`的选择条件交由用户来完成，增加了灵活性。

#### Lambda表达式

```java
package lambda;
 
import java.util.ArrayList;
import java.util.List;
import java.util.Random;
 
import charactor.Hero;
 
public class TestLamdba {
    public static void main(String[] args) {
        Random r = new Random();
        List<Hero> heros = new ArrayList<Hero>();
        for (int i = 0; i < 5; i++) {
            heros.add(new Hero("hero " + i, r.nextInt(1000), r.nextInt(100)));
        }
        System.out.println("初始化后的集合：");
        System.out.println(heros);
        System.out.println("使用Lamdba的方式，筛选出 hp>100 && damange<50的英雄");
        filter(heros,h->h.hp>100 && h.damage<50);
    }
 
    private static void filter(List<Hero> heros,HeroChecker checker) {
        for (Hero hero : heros) {
            if(checker.test(hero))
                System.out.print(hero);
        }
    }
 
}
```

至于`Lambda表达式`只是简化了匿名类的操作，一般的我们把`HeroChecker`称作函数式接口。

#### 方法引用

有时，`lambda`表达式涉及一个方法。例如，假设你希望只要出现一个定时器事件就打印这个事件的对象。

```java
var timer = new Timer(1000, event -> System.out::println)
```

表达式`System.out::println`是一个方法引用，==它指示编译器生成一个函数式接口的实例，覆盖这个接口的抽象方法来调用给定的方法==。