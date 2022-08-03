# Spark基础知识笔记

> 第一章Java基础
```java
package ChouXiangLei;

abstract class Person {
    abstract void eat();
}

class Chinese extends Person{
    @Override
    void eat() {
        System.out.println("chinese实现的eat");
    }
}

public class Abstract {
    public static void main(String[] args) {
        Person p = new Chinese();
//        Person p = new Person();
        p.eat();
        //使用Chinese类来创建抽象类的对象
    }
}

```
