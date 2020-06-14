> 内部类：即定义在另一个类中的类。使用内部类的主要原因：

* 内部类方法可以访问该类定义所在的作用域中的数据，包括私有的数据。
* 内部类可以对同一个包中的其他类隐藏起来。
* 当要定义一个回调函数时，可以使用匿名内部类。

## 1 基础

1. 内部类既可以访问自身的数据域，也可以访问创建它的外部类对象的数据域。
2. 内部类的对象中有一个隐式引用，指向创建它的外部类对象。语法：`OutClass.this`

代码示例：

```java
JavaLearning：com.prd.innerclass.InnerClassBaseTest
```

## 2 成员内部类

​		定义在另外一个类中，是最为普通的内部类。构造语法如下：`OutClass.InnerClass innerobject = outobject.new InnerClass(construction parameters)`。

​		在外部类范围之外，访问内部类的语法：`OuterClass.InnerClass`。

​		内部类可以拥有访问权限：

1.  private：只能在外部类的内部访问。
2.  protected：只能在同一个包下或者外部类的子类中访问。
3.  包（默认）：只能在同一个包下访问。
4.  public：任何地方都能访问。

## 3 局部内部类

​		定义在一个方法或者作用域之内的类。与成员内部类的不同，局部内部类的访问仅限于方法内或作用域内。

* 在局部内部类前不能用修饰符public和private,protected，static. 
* 可以定义与外部类同名的变量
* 如果内部类没有与外部类同名的变量，在内部类中可以直接访问外部类的实例变量
* 如果内部类中有与外部类同名的变量，直接用变量名访问的是内部类的变量，用**this.变量名访问的也是内部类变量**。用**外部类名.this.内部类变量名访问的是外部类变量**。
* 不可以定义静态变量和静态方法
* 可以访问外部类的局部变量(即方法内的变量)，但是变量必须是**final**的     
* 可以访问外部类的所有成员

代码示例：

```java
public class InnerClassA {
    private String str1 = "abc";
    private int n = 35;
    private static String str2 = "MMM";
    public void show(){
        final int K = 54;
        class InnerOne{
            int n = 12;
            //static String s = "name";内部类中不能定义static变量.
            public void aaa(){
                System.out.println("局部常量K的值为:  "+K);
                System.out.println("外围类成员变量str1的值:  "+str1);
                System.out.println("外围类成员变量n的值:  "+InnerClassA.this.n);
                System.out.println("外围类成员变量str2的值:  "+str2);
                System.out.println("内部类成员变量n的值:  "+n);
            }
        }
        new InnerOne().aaa();
    }
    public static void main(String[] args) {
        InnerClassA a = new InnerClassA();
        a.show();
    }
}
```

## 4 匿名内部类

​		匿名内部类，可以认为是一个特殊的局部内部类。它是一个唯一没有构造器的内部类，大多数情况下，用于接口回调。匿名内部类在编译的时候由系统自动起名为Outter$1.class。

​		匿名内部类用于继承其他类或是实现接口，并不需要增加额外的方法，只是对继承方法的实现或是重写。

* 匿名内部类不能有构造方法。  
* 匿名内部类不能定义任何静态成员、方法和类。  
* 匿名内部类不能是public,protected,private,static。  
* 只能创建匿名内部类的一个实例。
* 一个匿名内部类一定是在new的后面，用其隐含实现一个接口或实现一个类。  
* 因匿名内部类为局部内部类，所以局部内部类的所有限制都对其生效。
* 匿名内部类**不能是抽象**的，它必须要实现继承的类或者实现的接口的所有抽象方法。
* 当所在的方法的形参需要被内部类里面使用时，该形参必须为**final**
* **匿名内部类只要实现接口或抽象类的方法即可，及时另外定义方法，在new后也无法访问。但是通过反射可以**。

示例1：匿名内部类的基本实现

```java
abstract class Person {
    public abstract void eat();
}
public class Demo {
    public static void main(String[] args) {
        Person p = new Person() {
            public void eat() {
                System.out.println("eat something");
            }
        };
        p.eat();
    }
}
```

示例2：在接口上使用匿名内部类

```java
interface Person {
    public void eat();
}

public class Demo {
    public static void main(String[] args) {
        Person p = new Person() {
            public void eat() {
                System.out.println("eat something");
            }
        };
        p.eat();
    }
}
```

示例3：Thread类的匿名内部类实现

```java
public class Demo {
    public static void main(String[] args) {
        Thread t = new Thread() {
            public void run() {
                for (int i = 1; i <= 5; i++) {
                    System.out.print(i + " ");
                }
            }
        };
        t.start();
    }
}
```

示例4：局部内部类和匿名内部类只能访问局部final变量

```java
public class Test { 
    public void test(final int b) {
        final int a = 10;
        new Thread(){
            public void run() {
                System.out.println(a);
                System.out.println(b);
            };
        }.start();
    }
}
```

​		只能访问局部final变量的原因，**在run方法中访问的变量a根本就不是test方法中的局部变量a,而是在匿名内部里面创建一个拷贝。为了避免在run方法中改变变量a的值，从而造成数据不一致性，所以java编译器就限定必须将变量a限制为final变量，不允许对变量a进行更改**。

## 5 静态内部类

​		静态内部类也是定义在另一个类里面的类，只不过在类的前面多了一个关键字static。静态内部类是不需要依赖于外部类的，这点和类的静态成员属性有点类似，并且它不能使用外部类的非static成员变量或者方法，这点很好理解，因为在没有外部类的对象的情况下，可以创建静态内部类的对象，如果允许访问外部类的非static成员就会产生矛盾，因为外部类的非static成员必须依附于具体的对象。

1. 静态内部类只能访问外部类的静态变量或静态方法。

2. 接口中使用,可以实现公共代码

   创建静态内部类对象的一般形式为： ` OutClass.InnerClass innerobject= new OutClass.InnerClass ()`

示例1 基本模式

```java
public class Outer {
    public static int i =500;
    protected static class Inner {
        int i =100;
        String name;
        Inner(String name) {
            this.name = name;
        }
        void sayHello() {
            System.out.println("Hello " + name);
            Outer.i++;
        }
    }
    public Inner genInner(String name) {
        return new Inner(name);
    }
}
class Test {
    public static void main(String[] args) {
        Outer.Inner in1 = new Outer.Inner("1111");
        in1.sayHello();
        System.out.println(Outer.i);
        Outer.Inner in2 = new Outer().genInner("2222");
        in2.sayHello();
        System.out.println(Outer.i);
    }
}
```

示例2：接口内部类

```java
public interface AInterface { 
        void readme(); 

        class Inner1 implements AInterface { 
                public void readme() { 
                        System.out.println("我是一个接口内部类"); 
                } 
        } 
} 
class Main { 
        public static void main(String[] args) { 
                AInterface.Inner1 in1 = new AInterface.Inner1(); 
                in1.readme(); 
        } 
}
```

