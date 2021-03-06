##  1 接口

这里使用一个**比较函数**接口，来演示如何使用接口。

代码路径：

```Java
JavaLearning：com.prd.interfaces.comparable.ComparableTest
```

1. 接口中的方法自动属于Public，可以不提供public关键字。
2. 在接口中可以定义常量，但不嫩有实例域。
3. Java8中，接口可增加静态方法。但是一般不如此使用，一般都是将静态方法放在伴随类中。如在标准库中，可以看到成对出现的工具类，Collection/Collections、Path/Paths

### 1.1 默认方法

可以为接口方法提供一个默认实现，使用default修饰。

默认方法的一个重要作用就是“接口演化”，即可以理解为接口的版本升级。如1.0版本时，接口A中只有 methodA，但是在1.1中接口A增加methodB，如果不定义默认方法，在使用1.0版本的代码就必须进行修改（增加methodB的实现）。

代码示例：

```java
JavaLearning:com.prd.interfaces.defaultmethod.DefaultTest
```

* 默认方法冲突

  接口中定义默认方法，在超类或另一个接口中定义了同样的方法时，会产生冲突，有以下两种处理方式。
  
  * 超类优先：如果超类提供了一个具体方法，同名而且有相同参数类型的默认方法会被忽略
  * 接口冲突：如果一个超接口提供了一个默认方法，另一个接口也提供了同名且同参的方法，则必须指定明确

代码示例：

```java
JavaLearning:com.prd.interfaces.samemethod.SameMethondTest
```

###  1.2 标记接口

刚才示例的Comparable接口属于一般的接口，用途是确保一个类实现一个或一组特定的方法。而标记接口不包含任何方法，它的唯一作用就是允许在类型查询中使用` instanceof`。

如Cloneable接口，就是标记接口之一。下面借此，正好简述下对象克隆的知识点。

#### 1.2.1 何为克隆

克隆即完全拷贝对象，包括对象在内存中保存为2份，这就是克隆。下图所示只是引用的复制，而内存中不是两份，则不是克隆。

![1.2.1-1](images\1.2.1-1.png)

对于基本数据类型时，如果要克隆一个变量，即复制一个变量，可以直接操作。

```java
int num =20;
int num2 = num;
```

此时，num2就是num的复制。其它七种原始数据类型(boolean,char,byte,short,float,double.long)同样适用于该类情况。

对于复杂类型，想要克隆就比较复杂了。

#### 1.2.2 为何要克隆

​		为什么需要克隆对象？直接new一个对象不行吗？

​		答案是：克隆的对象可能包含一些已经修改过的属性，而new出来的对象的属性都还是初始化时候的值，所以当需要一个新的对象来保存当前对象的“状态”就靠clone方法了。那么我把这个对象的临时属性一个一个的赋值给我新new的对象不也行嘛？可以是可以，但是一来麻烦不说，二来，大家通过上面的源码都发现了clone是一个native方法，就是快啊，在底层实现的。

　　注意，常见的`Object a=new Object();Object b;b=a;`这种形式的代码复制的是引用，即对象在内存中的地址，a和b对象仍然指向了同一个对象。而通过clone方法赋值的对象跟原来的对象时同时独立存在的。

#### 1.2.3 如何实现克隆

这里先介绍2种克隆方法：**浅克隆(ShallowClone)**和**深克隆(DeepClone)**。在Java语言中，数据类型分为值类型（基本数据类型）和引用类型，值类型包括int、double、byte、boolean、char等简单数据类型，引用类型包括类、接口、数组等复杂类型。浅克隆和深克隆的主要区别在于是否支持引用类型的成员变量的复制。下图能好的说明这个：

![1.2.3-1](images\1.2.3-1.png)

**浅克隆：**

1. **被复制的类需要实现Clonenable接口**（不实现的话在调用clone方法会抛出CloneNotSupportedException异常)， 该接口为标记接口(不含任何方法)

2. **覆盖clone()方法，访问修饰符设为public**。**方法中调用super.clone()方法得到需要的复制对象**。（native为本地方法)

代码示例：

```java
JavaLearning:com.prd.interfaces.cloneable.ShallowCloneTest
```

**深克隆：**

1. **被复制的类需要实现Clonenable接口**（不实现的话在调用clone方法会抛出CloneNotSupportedException异常)， 该接口为标记接口(不含任何方法)
2. 克隆类的引用对象**都覆盖clone()方法**，并实现Clonenable接口。（相当于递归克隆类，出基本数据类型，其他的任何子引用对象都实现clone()）
3. **覆盖clone()方法**，访问修饰符设为public**。**方法中调用super.clone()方法得到需要的复制对象**。（native为本地方法)。并在clone()方法中设引用对象为原引用对象的克隆。

代码示例：

```java
JavaLearning:com.prd.interfaces.cloneable.DeepCloneTest
```

#### 1.2.4 解决多层克隆问题

如果引用类型里面还包含很多引用类型，或者内层引用类型的类里面又包含引用类型，使用clone方法就会很麻烦。这时我们可以用序列化的方式来实现对象的深克隆。

代码示例：

```java
public class Outer implements Serializable{
  private static final long serialVersionUID = 369285298572941L;  //最好是显式声明ID
  public Inner inner;
　//Discription:[深度复制方法,需要对象及对象所有的对象属性都实现序列化]
  public Outer myclone() {
      Outer outer = null;
      try { // 将该对象序列化成流,因为写在流里的是对象的一个拷贝，而原对象仍然存在于JVM里面。所以利用这个特性可以实现对象的深拷贝
          ByteArrayOutputStream baos = new ByteArrayOutputStream();
          ObjectOutputStream oos = new ObjectOutputStream(baos);
          oos.writeObject(this);
　　　　　　// 将流序列化成对象
          ByteArrayInputStream bais = new ByteArrayInputStream(baos.toByteArray());
          ObjectInputStream ois = new ObjectInputStream(bais);
          outer = (Outer) ois.readObject();
      } catch (IOException e) {
          e.printStackTrace();
      } catch (ClassNotFoundException e) {
          e.printStackTrace();
      }
      return outer;
  }
}
```

Inner也必须实现Serializable，否则无法序列化：

```java
public class Inner implements Serializable{
  private static final long serialVersionUID = 872390113109L; //最好是显式声明ID
  public String name = "";

  public Inner(String name) {
      this.name = name;
  }

  @Override
  public String toString() {
      return "Inner的name值为：" + name;
  }
}
```

#### 1.2.5 标记接口

在`1.2.1`~`1.2.4`中的覆盖clone()时，必须实现Cloneable接口，否则在运行clone()方法时会提示CloneNotSupportedException异常。

## 2 lambda表达式

​		lambda表达式是一个可传递的代码块，可以执行一次或多次。其主要的思想就是在Java中传递一个代码段。

​		lambda的语法：参数，箭头(->)以及一个表达式。如果代码无法在一个表达式中写完，则把这些代码放在{}中。

​		lambda表达式没有参数时，要提供() 号。

​		如果可以推导出一个lambda表达式的参数类型，则可以忽略其类型。

```java
(String first,String second) -> first.length()-second.length();
```

### 2.1 函数式接口

​		有且仅有一个抽象方法，但是可以有多个非抽象方法的接口称之为函数接口。函数式接口可以被隐式转换为 lambda 表达式和方法引用（实际上也可认为是Lambda表达式）上。只有在需要传递这些函数接口时，可以用lambda表达式代替。

​		如定义了一个函数式接口如下：

```
@FunctionalInterface
interface GreetingService 
{
    void sayMessage(String message);
}
```

​		@FunctionalInterface 注解的作用只是告诉编译器检查这个接口，保证该接口只能包含一个抽象方法，否则就会编译出错。

​		Java8 API中新增了许多函数式接口:

| 接口名         | 参数  | 返回值  |     用途 |
| -------------- | ----- | :-----: | -------: |
| Predicate      | T     | boolean |     断言 |
| Consumer       | T     |  void   |     消费 |
| Function<T,R>  | T     |    R    |     函数 |
| Supplier       | None  |    T    | 工厂方法 |
| UnaryOperator  | T     |    T    |   逻辑非 |
| BinaryOperator | (T,T) |    T    | 二元操作 |

​		JDK提供了大量常用的函数式接口以丰富Lambda的典型使用场景，它们主要在 java.util.function 包中被提供。

* Consumer接口概述

```java
@FunctionalInterface
public interface Consumer<T> {
    /**
     * 对给定参数执行消费操作。
     * @param t 输入参数
     */
    void accept(T t);
	/**
	 * 传入一个Consumer类型的参数，他的泛型类型，
	 * 跟本接口是一致的T，先做本接口的accept操作，
	 * 然后在做传入的Consumer类型的参数的accept操作
	*/
    default Consumer<T> andThen(Consumer<? super T> after) {
        Objects.requireNonNull(after);
        return (T t) -> { accept(t); after.accept(t); };
    }
}
```

代码示例：

```Java
JavaLearning:com.prd.interfaces.functional.ConsumerFunctionalTest
```

* Supplier概述

```java
@FunctionalInterface
public interface Supplier<T> {
    /**
     * Gets a result.
     *
     * @return a result
     */
    T get();
}
```

代码示例：

```java
JavaLearning:com.prd.interfaces.functional.SupplierFunctionalTest
```

* Function概述

  Function接口用来根据一个类型的数据得到另一个类型的数据，前者称为前置条件，后者称为后置条件。类似一种数据管道，将数据传递处理。

```java
@FunctionalInterface
public interface Function<T, R> {
    R apply(T t);
    ......
}
```

Function 接口中最主要的抽象方法为: R apply(T t) ，根据类型T的参数获取类型R的结果。 使用的场景例如:将 String 类型转换为 Integer 类型。

代码示例：

```java
JavaLearning:com.prd.interfaces.functional.FunctionTest
```

​		***Function接口的identity方法用在lamdba方法中的,Collectors.toMap的三个参数的http://www.java2s.com/Tutorials/Java/Java_Stream/0290__Java_Stream_Collect_to_Map.htm 可以参考collect Map的example 4  ????***

* Predicate概述

​       有时候我们需要对某种类型的数据进行判断，从而得到一个boolean值结果。这时可以使用java.util.function.Predicate<T> 接口。

```java
@FunctionalInterface
public interface Predicate<T> {
    boolean test(T t);
    ......
}
```

​		Predicate还实现有多个default方法，可以进行 &（与），|（或），！（非）等组合判断。

代码示例：

```
JavaLearning:com.prd.interfaces.functional.PredicateTest
```

* UnaryOperator接口概述
```java
public interface UnaryOperator<T> extends Function<T, T>
```
​		从`extends Function<T, T>`可以看出, 这个方法是Function的一个简单写法, 即传入一个参数, 返回值类型和参数类型相同的方法。这个方法当然也继承了`andThen`和`compose`; 此外, 由于静态方法不能继承所以在代码中重写了一遍, 其作用是相同的。

代码示例：

```java
JavaLearning:com.prd.interfaces.functional.UnaryOperatorTest
```

### 类似的接口

1. `IntFunction<R>`等, 传入参数为int等基本类型, 返回类型为R。
2. `ToIntFunction<T>` 等, 传入类型为T, 返回类型为int。
3. `IntToDoubleFunction`等, 传入int, 返回double。
4. `BiFunction<T, U, R>`, 传入类型为T和u, 返回类型为R。
5. `ToIntBiFunction<T, U>`, 传入类型为T和U, 返回类型为int。
    需要注意, 上面的方法和`Funciton`本身并没有继承关系, 都是相互独立的。

   

### 2.2 lambda表达式语法

* lambda表达式的一般语法

```java
(Type1 param1, Type2 param2, ..., TypeN paramN) -> {
  statment1;
  statment2;
  //.............
  return statmentM;
}
```

这是lambda表达式的完全式语法，后面几种语法是对它的简化。

* 单参数语法

```java
param1 -> {
  statment1;
  statment2;
  //.............
  return statmentM;
}
```

当lambda表达式的参数个数只有一个，可以省略小括号

例如：

将列表中的字符串转换为全小写

```java
List<String> proNames = Arrays.asList(new String[]{"Ni","Hao","Lambda"});
List<String> lowercaseNames1 = proNames.stream().map(name -> {return name.toLowerCase();}).collect(Collectors.toList());
```

* 单语句写法

```java
param1 -> statment
```

 

当lambda表达式只包含一条语句时，可以省略**大括号、return和语句结尾的分号**

例如：将列表中的字符串转换为全小写

```java
List<String> proNames = Arrays.asList(new String[]{"Ni","Hao","Lambda"});
List<String> lowercaseNames2 = proNames.stream().map(name -> name.toLowerCase()).collect(Collectors.toList());
```

* 方法引用写法

（方法引用和lambda一样是Java8新语言特性，后面会讲到）

```java
Class or instance :: method
```

例如：将列表中的字符串转换为全小写

```java
List<String> proNames = Arrays.asList(new String[]{"Ni","Hao","Lambda"});
List<String> lowercaseNames3 = proNames.stream().map(String::toLowerCase).collect(Collectors.toList());
```

### 2.3 lambda表达式可使用的变量

先举例：

```java
//将为列表中的字符串添加前缀字符串
String waibu = "lambda :";
List<String> proStrs = Arrays.asList(new String[]{"Ni","Hao","Lambda"});
List<String>execStrs = proStrs.stream().map(chuandi -> {
Long zidingyi = System.currentTimeMillis();
return waibu + chuandi + " -----:" + zidingyi;
}).collect(Collectors.toList());
execStrs.forEach(System.out::println);
```

输出:

lambda :Ni -----:1474622341604
lambda :Hao -----:1474622341604
lambda :Lambda -----:1474622341604

变量waibu ：外部变量

变量chuandi ：传递变量

变量zidingyi ：内部自定义变量

​		lambda表达式可以访问给它传递的变量，访问自己内部定义的变量，同时也能访问它外部的变量。

​		不过lambda表达式访问**外部变量有一个非常重要的限制：变量不可变**（只是引用不可变，而不是真正的不可变）。

​		当在表达式内部修改waibu = waibu + " ";时，IDE就会提示你：`Local variable waibu defined in an enclosing scope must be final or effectively final`编译时会报错。因为变量waibu被lambda表达式引用，所以编译器会隐式的把其当成final来处理。以前Java的匿名内部类在访问外部变量的时候，外部变量必须用final修饰。现在java8对这个限制做了优化，可以不用显示使用final修饰，但是编译器隐式当成final来处理。

### 2.4 方法引用

​		方法引用就是在不调用方法的情况下引用方法，即就是简化lambda语法。如：`Timer r = new Timer(1000, event - > System.out.println(event))` 就可以简化为 `Timer ` r = new Timer(1000, System.out::println)。

​		方法引用的标准形式是:`类名::方法名`。（**注意：只需要写方法名，不需要写括号**）

| 类型                             | 示例                      |
| -------------------------------- | ------------------------- |
| 引用静态方法                     | Class::staticMethod       |
| 引用某个对象的实例方法           | object::instanceMethod    |
| 引用某个类型的任意对象的实例方法 | ClassType::instanceMethod |
| 引用构造方法                     | ClassName::new            |



#### 2.4.1 出现方法引用的原因

​		在Java中，我们可以通过创建新对象来使用对象的引用,或者引用已有的对象，例如：

```java
//创建新对象来引用该对象
List list = new ArrayList();
dosth(list);

//引用已有的对象
List list2 = list;
dosth(list2);
```

​		如果我们只在另一个方法中使用对象的方法，我们仍然必须将完整的对象作为参数传递。那么将方法作为参数传递不是更有效吗？**方法引用这种语法糖不能用于所有的方法，它们只能用于只有一个方法的lambda表达式**。

```java
//lambda转方法引用
Consumer<String> c = s -> System.out.println(s);
Consumer<String> c = System.out::println;
```

####  2.4.2 静态方法引用

​		当我们所使用的lambda表达式仅仅调用了一个静态方法，我们就可以使用静态方法引用。格式如：`(args) -> Class.staticMethod(args)` ==> `Class::staticMethod`，不需要显式的传递参数，方法引用中会自动传递参数。

代码示例:

```java
JavaLearning:com.prd.interfaces.reference.ClassMethodReferenceTest
```

#### 2.4.3 特定类型的对象的实例方法的方法引用

​		使用实例的类型而非实例本身，参数隐式传递。格式如：`(obj, args) -> obj.instanceMethod(args)`==>`ObjectType::instanceMethod`

代码示例：

```java
JavaLearning:com.prd.interfaces.reference.ClassInstanceMethodReferenceTest
```

#### 2.4.4 已有对象的实例方法引用

​		直接引用已经在其他地方实例化的对象的一个方法，本身不需要再实现一次。格式如：`(args) -> obj.instanceMethod(args)`==>`obj::instanceMethod`

代码示例：

```java
JavaLearning:com.prd.interfaces.reference.ObjectInstranceMethodReferenceTest
```

#### 2.4.5 构造方法引用

​		唯一的功能就是创建一个新对象，与其他情况一样，参数（如果有）不会在方法引用中传递。格式如：`(args) -> new ClassName(args)`==>`ClassName::new`。

​		如果存在多个构造函数，则Java会根据实际情况，推算使用哪个构造器。

​		Java是无法创建泛型类型T的数组，但是通过构造方法引用，可以轻易的实现这个。eg：`Person[] people = stream.toArray(Person[]::new)`。

代码示例：

```java
JavaLearning:com.prd.interfaces.reference.ConstructorReferenceTest
```

#### 2.4.6 总结

​		方法引用常用户Java8的Streams特性中，使得编码更为简练，同时可以使设计模式更为精巧。