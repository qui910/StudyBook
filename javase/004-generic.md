## 1 概念

​		**泛型的定义：**

​		泛型是JDK 1.5的一项新特性，它的本质是参数化类型（Parameterized Type）的应用，也就是说所操作的数据类型被指定为一个参数，在用到的时候在指定具体的类型。这种参数类型可以用在类、接口和方法的创建中，分别称为泛型类、泛型接口和泛型方法。

​		Java语言的泛型因为存在类型擦除的概念，所以被称之为伪泛型。

​		使用泛型机制编写的程序代码要比那些杂乱的使用Object变量，然后再进行强制类型转换的代码具有更好的安全性和可读性。泛型对于集合类来说尤其有用。

​		泛型程序设计（Generic Programming）意味着编写的代码可以被很多不同类型的对象所重用。

​		泛型的类型参数只支持引用类型，如传递类似int 的基本类型，则Java会利用自动装箱机制转为Integer，但是像int[] 是泛型不支持的。

​		泛型的参数类型可以用在类、接口和方法的创建中，分别称为泛型类、泛型接口和泛型方法。1.

### 1.1 基本概念

​			简单介绍下泛型的一些基本术语，以`ArrayList<E>`和`ArrayList<Integer>`做简要介绍：

* `ArrayList<E>`称为泛型类型

* `ArrayList<E>`中的 `E`称为类型变量或者类型参数

* `ArrayList<Integer> `称为参数化的类型

* `ArrayList<Integer>`中的`integer`称为类型参数的实例或者实际类型参数。

* `ArrayList<Integer>`中的`<Integer>`念为`typeof   Integer`

* `ArrayList`称为原始类型

​			注意：类型变量使用大写形式，且比较短，这是很常见的。在Java库中，使用变量E表示集合的元素类型，K和V分别表示关键字与值的类型。（需要时还可以用临近的字母U和S）表示“任意类型”。

代码示例：

```java
JavaLearning：com.prd.generic.GenericTest
```



## 2 泛型类

​		一个泛型类（generic class）就是具有一个或多个类型变量的类。定义一个泛型类十分简单，只需要在类名后面加上<>，再在里面加上类型参数。

​		类型变量的作用域就是类的实例部分，包括方法以及任何实例初始化语句块。类的静态部分不会受到泛型参数化的影响，并且，在静态方法或静态初始化程序中，类型变量是不可见的。

```java
@Slf4j
public class Pair<T extends Comparable> {

    private T first;

    private T second;

    public Pair() {
    }

    public Pair(T first, T second) {
        this.first = first;
        this.second = second;
    }

    public T getFirst() {
        return first;
    }

    public void setFirst(T first) {
        this.first = first;
    }

    public T getSecond() {
        return second;
    }

    public void setSecond(T second) {
        this.second = second;
    }

    public <T> void getAdd(T t1,T t2) {
        log.info("t1 is {},t2 is {}",t1,t2);
    }

    @Override
    public String toString() {
        return "Pair{" +
                "first=" + first +
                ", second=" + second +
                '}';
    }
}
```

### 2.1 泛型类的继承
​		泛型类型可以像任何其他类一样进行子类化，通过泛型或非泛型的子类可以做到这一点。

​		非泛型的子类必须扩展父类型的一个特定实例，填充必须的参数以使其变得实际。
示例：

```java
class DateList extends ArrayList<Date>{}
```

​		一个泛型类的泛型子类可以扩展类的一个具体实例（就像前面的示例所示），或者可以共享一个类型变量，该类型变量通过向上传递可以对父类进行实例化：

示例：

```java
class AdjustableTrap<T> extends Trap<T> {
	public void setSize(int i);
}
```

### 2.2 边界
​		类型变量也可以引用类型声明中的其他类型变量：
```java
class Foo <A,B extends A> {...}
```
​		在extend关键字后指定的第一个边界（在字面上，位于最左边）变成了在擦除中所使用的类型。这意味着，如果该类型扩展了一个类类型，它总是擦除后的类型，因为它必须总是第一个出现。	

## 3 泛型接口

​		定义泛型接口和泛型类差不多
```java
interface Show<T,U>{
    void show(T t,U u);
}
```

## 4 泛型方法

​		泛型类在多个方法签名间实施类型约束。在 List<V> 中，类型参数 V 出现在 get()、add()、contains() 等方法的签名中。当创建一个 Map<K, V> 类型的变量时，您就在方法之间宣称一个类型约束。您传递给 add() 的值将与 get() 返回的值的类型相同。

​		类似地，之所以声明泛型方法，一般是因为您想要在该方法的多个参数之间宣称一个类型约束。

```java
public static <T, U> T get(T t, U u) {
    if (u != null)
        return t;
    else
        return null;
}
```

## 5 泛型变量的类型限定
​		有的时候，类、接口或方法需要对类型变量加以约束。

​		类型限定在泛型类、泛型接口和泛型方法中都可以使用，不过要注意下面几点：

* 不管该限定是类还是接口，统一都使用关键字 extends

* 可以使用&符号给出多个限定

* 如果限定既有接口也有类，那么类必须只有一个，并且放在首位置

  ```java
  public static <T extends Comparable&Serializable> T get(T t1,T t2)
  ```

​		如果Pair这样声明`public class Pair<T extends Serializable&Comparable>` ，那么原始类型就用`Serializable`替换，而编译器在必要的时要向`Comparable`插入强制类型转换。为了提高效率，应该将标签 （tagging）接口（即没有方法的接口）放在边界限定列表的末尾。

## 6 类型擦除

​		正确理解泛型概念的首要前提是理解类型擦出（type erasure）。

​		Java中的泛型基本上都是在编译器这个层次来实现的。在生成的Java字节码中是不包含泛型中的类型信息的。使用泛型的时候加上的类型参数，会在编译器在编译的时候去掉。这个过程就称为类型擦除。

​		Java不能实现真正的泛型，只能使用类型擦除来实现伪泛型，这样虽然不会有类型膨胀的问题，但是也引起了许多新的问题。所以，Sun对这些问题作出了许多限制，避免我们犯各种错误。

```java
public class Test2{
    public static void main(String[] args) {
        /**不指定泛型的时候*/
        int i=Test2.add(1, 2); //这两个参数都是Integer，所以T为Integer类型  
        Number f=Test2.add(1, 1.2);//这两个参数一个是Integer，以风格是Float，所以取同一父类的最小级，为Number  
        Object o=Test2.add(1, "asd");//这两个参数一个是Integer，以风格是Float，所以取同一父类的最小级，为Object  

        /**指定泛型的时候*/
        int a=Test2.<Integer>add(1, 2);//指定了Integer，所以只能为Integer类型或者其子类  
        int b=Test2.<Integer>add(1, 2.2);//编译错误，指定了Integer，不能为Float  
        Number c=Test2.<Number>add(1, 2.2); //指定为Number，所以可以为Integer和Float  
    }

    //这是一个简单的泛型方法  
    public static <T> T add(T x,T y){
        return y;
    }
}
```

### 6.1 先检查，在编译，以及检查编译的对象和引用传递的问题

​		类型变量会在编译的时候擦除掉变为原始类型Object。为了避免因此造成诸如：`ArrayList<String>`中写入Integer等错误，Java编译器是通过先检查代码中泛型的类型，然后再进行类型擦除，在进行编译的。

```java
ArrayList<String> arrayList1=new ArrayList(); //第一种 情况 
ArrayList arrayList2=new ArrayList<String>();//第二种 情况  
```

​		`arrayList1`能正确限定为String，`arrayList2`则不行，所以，类型检查就是针对引用的，谁是一个引用，用这个引用调用泛型方法，就会对这个引用调用的方法进行类型检测，而无关它真正引用的对象。	
​		`new ArrayList()`只是在内存中开辟一个存储空间，可以存储任何的类型对象。而真正涉及类型检查的是它的引用，因为我们是使用它引用 `arrayList1` 来调用它的方法，比如说调用add()方法。所以`arrayList1`引用能完成泛型类型的检查。而引用`arrayList2`没有使用泛型，所以不行。

​		泛型中没有继承关系，在Java中，像下面形式的引用传递是不允许的。

```java
ArrayList<String> arrayList1=new ArrayList<Object>();//编译错误  
ArrayList<Object> arrayList1=new ArrayList<String>();//编译错误 
```

### 6.2 自动类型转换

​		因为类型擦除的问题，所以所有的泛型类型变量最后都会被替换为原始类型，但是在取值时会自动进行类型转换。
示例：`ArrayList的get方法`

```java
public E get(int index) {
        RangeCheck(index);
        return (E) elementData[index];
} 
```

### 6.3 类型擦除与多态的冲突和解决方法

​		子类继承泛型类为了实现多态想重载父类（泛型类）的方法，但是因为类型擦除的存在会导致重载失败，变为重写。而JVM使用桥方法巧妙的继续实现了重载的多态。

​		现在有这样一个泛型类：

```java
class Pair<T> {
    private T value;
    public T getValue() {
        return value;
    }
    public void setValue(T value) {
        this.value = value;
    }
} 
```

​		然后我们想要一个子类继承它

```java
class DateInter extends Pair<Date> {
    @Override
    public void setValue(Date value) {
        super.setValue(value);
    }
    @Override
    public Date getValue() {
        return super.getValue();
    }
} 
```

​		结果

```java
public static void main(String[] args) throws ClassNotFoundException {
    DateInter dateInter=new DateInter();
    dateInter.setValue(new Date());
    // dateInter.setValue(new Object()); //编译错误
}
```

### 6.4 泛型类型变量不能是基本数据类型

​		不能用类型参数替换基本类型。就比如，没有`ArrayList<double>`，只有 `ArrayList<Double>`。因为当类型擦除后，`ArrayList`的原始类型变为Object，但是Object类型不能存储 double值，只能引用Double的值。

​		继承只适用于“基本”泛型类型，而不适用于参数类型。此外，只有当两个泛型类型都是基于完全相同的参数类型进行实例化的时候，可赋值性才适用。换句话说，在基本泛型类型之后，只有单维度的继承，但是还有额外的限制，就是参数类型必须是完全相同的。

示例：

```java
Collection<Date> cd;
List<Date> ld = new ArrayList<Date>();
cd=ld; //ok!  cd 和 ld 必须都是Date参数类型
```

### 6.5 运行时类型查询

​		`ArrayList<String> arrayList=new ArrayList<String>();   `因为类型擦除之后，`ArrayList<String>`只剩下原始类型，泛型信息String不存在了。那么，运行时进行类型查询的时候使用下面的方法是错误的

```java
if( arrayList instanceof ArrayList</>)
```

### 6.6 异常中使用泛型的问题

​		不能抛出也不能捕获泛型类的对象。事实上，泛型类扩展`Throwable`都不合法。
例如：

​		下面的定义将不会通过编译：

```java
public class Problem<T> extends Exception{......}
```

​		不能再catch子句中使用泛型变量

```java
public static <T extends Throwable> void doWork(Class<T> t){
    try{  
        ...
    }catch(T e){ //编译错误  
        ...
    }
}
```

​		但是在异常声明中可以使用类型变量。下面方法是合法的。

```java
public static<T extends Throwable> void doWork(T t) throws T{
    try{  
    ...
    }catch(Throwable realCause){
        t.initCause(realCause);
        throw t;
    }  
}
```

### 6.7 数组

​		不能声明参数化类型的数组。

```java
Pair<String>[] table = new Pair<String>(10); //ERROR  
```

### 6.8 泛型类型的实例化

​		不能实例化泛型类型

```java
first = new T(); //ERROR 
```

​		不能建立一个泛型数组

```java
public<T> T[] minMax(T[] a){
    T[] mm = new T[2]; //ERROR  
 ...
}  
```

​		可以用反射构造泛型对象和数组

```java
public static <T extends Comparable> T[] minmax(T[] a)
{
   T[] mm == (T[])Array.newInstance(a.getClass().getComponentType(),2);
      ...
   // 以替换掉以下代码
   // Obeject[] mm = new Object[2];
   // return (T[]) mm;
} 
```

### 6.9 类型擦除后的冲突

* 当泛型类型被擦除后，创建条件不能产生冲突。如果在Pair类中添加下面的equals方法：

  ```java
  class Pair<T>   {
        public boolean equals(T value) {
            return null;
        }
  } 
  ```
​		考虑一个Pair<String>。从概念上，它有两个equals方法：
  
  ```java
  Boolean equals(String); //在Pair<T>中定义
  boolean equals(Object); //从object中继承
  ```

​		但是，这只是一种错觉。实际上，擦除后方法`boolean equals(T)`变成了方法 `boolean equals(Object)`这与`Object.equals`方法是冲突的！当然，补救的办法是重新命名引发错误的方法。

* 泛型规范说明提及另一个原则“要支持擦除的转换，需要强行制一个类或者类型变量不能同时成为两个接口的子类，而这两个子类是同一接品的不同参数化。”下面的代码是非法的：
```java
  class Calendar implements Comparable<Calendar>{ ... }
  class GregorianCalendar extends Calendar implements Comparable<GregorianCalendar>{...}//ERROR  
```

​		`GregorianCalendar`会实现`Comparable<Calender>`和`Compable<GregorianCalendar>`，这是同一个接口的不同参数化实现。这一限制与类型擦除的关系并不很明确。

非泛型版本：

```java
class Calendar implements Comparable{ ... }
class GregorianCalendar extends Calendar implements Comparable{...} 
```

是合法的。

### 6.10 泛型在静态方法和静态类中的问题

​		泛型类中的静态方法和静态变量不可以使用泛型类所声明的泛型类型参数。

举例说明：

```java
public class Test2<T> {
    public static T one;   //编译错误    
    public static  T show(T one){ //编译错误    
        return null;
    }
}  
```

​		因为泛型类中的泛型参数的实例化是在定义对象的时候指定的，而静态变量和静态方法不需要使用对象来调用。对象都没有创建，如何确定这个泛型参数是何种类型，所以当然是错误的。

​		但是要注意区分下面的一种情况：

```java
public class Test2<T> {
    public static <T >T show(T one){//这是正确的    
        return null;
    }
}  
```

​		因为这是一个泛型方法，在泛型方法中使用的T是自己在方法中定义的T，而不是泛型类中的T。

## 7 通配符

​	通配符有三种：

* 无限定通配符   形式`</>`
* 上边界限定通配符 形式`< / extends Number>`    //用Number举例
* 下边界限定通配符    形式`< / super Number> `   //用Number举例

###　7.1　泛型中的？通配符

​		如果定义一个方法，该方法用于打印出任意参数化类型的集合中的所有数据，如果这样写

```java
public class GernericTest {
    public static void main(String[] args) throws Exception{
        List<Integer> listInteger =new ArrayList<Integer>();
        List<String> listString =new ArrayList<String>();
        printCollection(listInteger);
        printCollection(listString);
    }
    public static void printCollection(Collection<Object> collection){
        for(Object obj:collection){
            System.out.println(obj);
        }
    }
}
```

​		语句`printCollection(listInteger);`报错`The method printCollection(Collection<Object>) in the type GernericTest is not applicable for the arguments (List<Integer>)`这是因为泛型的参数是不考虑继承关系就直接报错。

​		这就得用？通配符

```java
public class GernericTest {
    public static void main(String[] args) throws Exception{
        List<Integer> listInteger =new ArrayList<Integer>();
        List<String> listString =new ArrayList<String>();
        printCollection(listInteger);
        printCollection(listString);
    }
    public static void printCollection(Collection</> collection){
        for(Object obj:collection){
            System.out.println(obj);
        }
    }
}
```

​		在方法`public static void printCollection(Collection</> collection){}`中不能出现与参数类型有关的方法比如`collection.add();`因为程序调用这个方法的时候传入的参数不知道是什么类型的，但是可以调用与参数类型无关的方法比如`collection.size();`

​		总结：使用/通配符可以引用其他各种参数化的类型,/通配符定义的变量的主要用作引用，可以调用与参数化无关的方法，不能调用与参数化有关的方法。

### 7.2.泛型中的/通配符的扩展
#### 7.2.1.界定通配符的上边界
`Vector</ extends 类型1> x = new Vector<类型2>();`类型1指定一个数据类型，那么类型2就只能是类型1或者是类型1的子类。

```java
Vector</ extends Number> x = new Vector<Integer>();//这是正确的
Vector</ extends Number> x = new Vector<String>();//这是错误的
```

#### 7.2.2.界定通配符的下边界
`Vector</ super 类型1> x = new Vector<类型2>();`类型1指定一个数据类型，那么类型2就只能是类型1或者是类型1的父类。

```java
Vector</ super Integer> x = new Vector<Number>();//这是正确的
Vector</ super Integer> x = new Vector<Byte>();//这是错误的
```

提示：限定通配符总是包括自己













