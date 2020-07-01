







## 类的初始化时机

1. 创建类的实例
2. 访问类的静态变量(注意：当访问类的静态并且final修饰的变量时，不会触发类的初始化。)，或者为静态变量赋值。
3. 调用类的静态方法（注意：调用静态且final的成员方法时，会触发类的初始化！一定要和静态且final修饰的变量区分开！！）
4. 使用java.lang.reflect包的方法对类进行反射调用的时候，如果类没有进行过初始化，则需要先触发其初始化。如：Class.forName("********");注意通过类名.class得到Class文件对象并不会触发类的加载。
5. 初始化某个类的子类
6. 直接使用java.exe命令来运行某个主类(java.exe运行，本质上就是调用main方法，所以必须要有main方法才行)。



​		类的初始化就是执行`clinit()`方法，该方法属于JVM虚拟机帮助创建的方法。方法中有静态类变量显示赋值代码和静态代码块组成，其中类变量的显示赋值代码和静态代码块代码从上到下的顺序执行。而且此方法只执行一次。

​		实例的初始化就是执行`init() `方法，`init()`方法可能重载很多个，有几个构造器就有几个init方法。`init()`方法由非静态实例变量显示赋值代码和非静态代码块、对应构造器代码组成，同时非静态实例变量显示赋值代码和非静态代码块代码从上到下的顺序执行，而对应的构造器最后执行。每次创建实例对象，调用对应的构造器，执行的就是对应的`init`方法，`init()`方法的首行代码是`super()`会调用父类的`init()`方法。

​		

代码示例：

```java
public class MyClassloaderTest1 {

  public static void main(String[] args) {
    System.out.println(MyChild.j1);

    System.out.println(MyChild.j2);

    System.out.println(MyChild.k1);

	System.out.println(new MyChild().i1);

    System.out.println(new MyChild().i);
  }
}

class MyParent{
    public int i = geti();

    public static final int j = 5;

    public static int k = getk();

    static {
        System.out.println("父类静态块初始化");
    }

    {
        System.out.println("父类实例块初始化");
    }

    public int geti(){
        System.out.println("父类实例变量初始化");
        return 0;
    }

    public static int getk(){
        System.out.println("父类静态变量初始化");
        return 0;
    }

    public MyParent() {
        System.out.println("父类构造器初始化");
    }
}

class MyChild extends MyParent {
    public int i1 = geti1();

    public static final int j1 = 5;

    public static final int j2 = getj2();

    public static int k1 = getk1();

    static {
        System.out.println("子类静态块初始化");
    }

    {
        System.out.println("子类实例块初始化");
    }


    public int geti1(){
        System.out.println("子类实例变量初始化");
        return 1;
    }

    public static int getj2() {
        System.out.println("子类final静态变量初始化");
        return 2;
    }

    public static int getk1(){
        System.out.println("子类静态变量初始化");
        return 3;
    }

    public MyChild() {
        System.out.println("子类构造器初始化");
    }
}
```

**（1） `System.out.println(MyChild.j1);`**

*执行结果：*

```shell
5
```

*分析：*

只调用子类或父类常量不会导致其他的初始化，此时`j`或`j1`都属于常量，是放置在子类的常量池中，并且父类都不会加载。

**（2）`System.out.println(MyChild.j2);`**

*执行结果：*

```shell
父类静态变量初始化
父类静态块初始化
子类final静态变量初始化
子类静态变量初始化
子类静态块初始化
2
```

*分析：*

调用子类的final静态变量，会导致父类与子类静态代码块初始化。此时`j2`因为编译时无法确定具体的值，所以不能认为是常量。

**（3）`System.out.println(MyChild.k1);`**

*执行结果：*

```shell
父类静态变量初始化
父类静态块初始化
子类final静态变量初始化
子类静态变量初始化
子类静态块初始化
3
```

*分析：*

调用子类的静态变量`k1`，会导致父类与子类静态代码块初始化。与`j2`的唯一区别就是`k2`时可变的，但是`j2`不可变。

**（4）``System.out.println(MyChild.k1);``**

*执行结果：*

```shell
父类静态变量初始化
父类静态块初始化
0
```

*分析：*

子类直接调用父类的静态变量`k`，只会导致父类的静态变量和静态代码块初始化。

**（5）`System.out.println(new MyChild().i1);`**

*执行结果：*

```shell
父类静态变量初始化
父类静态块初始化
子类final静态变量初始化
子类静态变量初始化
子类静态块初始化
父类实例变量初始化
父类实例块初始化
父类构造器初始化
子类实例变量初始化
子类实例块初始化
子类构造器初始化
0
```

*分析：*

此时无论调用子类的`i1`还是父类的`i`都会导致，子类和父类同时全部初始化。需要注意一点，如果代码做部分变更，即子类重写`public int geti()`，并将`geti()`赋值给`i1`，那么这时打印结果应该为：

```shell
父类静态变量初始化
父类静态块初始化
子类final静态变量初始化
子类静态变量初始化
子类静态块初始化
子类实例变量初始化   <------
父类实例块初始化
父类构造器初始化
子类实例变量初始化
子类实例块初始化
子类构造器初始化
0
```

