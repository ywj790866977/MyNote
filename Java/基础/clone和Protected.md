

## 1.静态代码块到底是不是最先执行



```java
/**
 * @program: design
 * @description:
 * @author: 好人
 * @date: 2020-02-10 22:55
 **/
public class Emp {
    public static final int age=10;
    static {
        System.out.println("呵呵..");
    }
}

 class Test01 {
    public static void main(String[] args) {
        System.out.println(Emp.age);
    }
}

###
10
```

问题:

为什么没有调用静态代码块

答案:

因为使用static和final修饰的变量是常量, 常量在编译的时候就会复制, 我们使用对象.调用的时候,因为这个常量已经有值了, 所以就不用初始化类, 就可以获取到.

当我们去掉final, 他就会先加载类, 然后才是调用这个属性





## 2. clone方法的调用



```java
public class TestDemo01 {

    public static void main(String[] args) {
        Person p = new Person();
        //不能调用
      
        p.clone();
    }
}
class Person{
		public Object test() throws CloneNotSupportedException {
       //可以调用
        return super.clone();
    }
}
```

![image-20200212000546330](/Users/yan/Library/Application Support/typora-user-images/image-20200212000546330.png)

Protected 修饰符 的原因, 是包内可见的，并且对子类可见

所有类都集成于OBject类, 按道理说都可以执行Object类的clone方法, 

但是因为Protected修饰的原因,  调用地方的不同,就会有问题

测试类和Object类没有直接的关系, 所以他不能直接调用Person类集成的Clone方法,而在Person类中,满足了Protected的条件.他是Object的子类,所以可以修改.

但是我们的test类,不是Object的直接子类,

相当于他们两个虽然都是一个祖宗,但是他们的父亲都不是一个人, 所以不能叫另一个人的父亲为爸爸. 这样就乱套了

当我们重写了Person类中的clone方法, Test方法就可以调用了