# 什么是泛型

在不确定数据类型的情况下使用参数进行替代，例如，这个参数T就是数据类型。

```java
public class Box<T> {
    private T content;

    public void set(T content) {
        this.content = content;
    }

    public T get() {
        return content;
    }
}
```

泛型的本质就是**数据类型参数化**

# 为什么要用泛型

没有泛型机制之前，如果有不确定的数据类型只能使用Object来进行维护。例如ArrayList

```java
public class ArrayList{
    private Object[] elecmentData;
    .......
    public Object get(int i){}
    public void add(Object o){}
}
```

这样做会有两个问题

+ 在获取一个值的时候必须进行强制类型转换
+ 编译阶段没有错误检查，可以往里面添加任意类型的值
  
  

使用泛型可以获得更好的可读性，一眼就可以看到这个数据是什么类型

```java
List<String> list = new ArrayList<>();
```

+ 在获取值的时候不需要进行强制类型转换
+ 在编译阶段就可以进行错误类型检查

# 泛型类

## 泛型类定义

泛型类的定义的基本语法

```sql
class 类名<泛型标识>{
  private 泛型标识 变量名字;
  ...
  ...
  ...
}
```

<> 括号里面的泛型标识 又被叫做 类型参数，用来设置任何数据类型



定义泛型类举例

```java
// 这个参数类型T 是由外部传入
public class Generic<T>{
    private T key;

    public Generic(T key) {
        this.key = key;
    }

    public T getKey() {
        return key;
    }

    public void setKey(T key) {
        this.key = key;
    }
}
```

<> 里面可以任意设置，但是如果不写<>或者<>里面不写任何东西，默认就是object



**那些情况可以写泛型类的类型参数**

+ 非静态方法的成员类属性
+ 非静态方法的形参类型
+ 非静态的成员方法的返回值类型
  
  

**那些情况下不能写泛型类所声明的类型参数**

+ 静态成员变量
+ 静态成员方法的返回参数
+ 静态成员方法的形参
  
  

**泛型类可以接受多个类型参数**

```java
public class MutipartParamGeneric<T,E>{
    private E value1;
    private T value2;

    public E getValue1(){
        return value1;
    }

    public T getValue2(){
        return value2;
    }
}
```

## 泛型类的使用

泛型类创建的时候，需要指定类型参数T的具体数据，即尖括号里面传入的数据类型，如果不写默认是Object类型。

泛型类

```java
public class Generic<T>{
    private T key;
    public Generic(T key) {
        this.key = key;
    }
    public T getKey() {
        return key;
    }
    public void setKey(T key) {
        this.key = key;
    }
}
```



```java
public class DEMO {
    public static void main(String[] args) {
        // <> 里面不写任何内容，相当于<Object>
        Generic generic = new Generic("TEST");
        // 传String，原来的泛型类可以想象成他会自动扩展，其他类型参数会被替换
        Generic<String> generic = new Generic<>("TEST");
        System.out.println(generic.getKey());
    }
}
```

# 泛型接口

## 泛型接口定义

定义语法

```java
public interface 接口名称<参数类型>{

}
```

举例

```java
public interface GenericInterface<T> {
     void show(T t) ;
}
```



定义类AIGenericInterfaceImpl 继承泛型接口GenericInterface，不传类型参数 默认Object

```java
public class AIGenericInterfaceImpl implements GenericInterface<String> {
    @Override
    public void show(String s) {
        System.out.println("do something");
    }
}
```



定义接口AI继承泛型接口GenericInterface，不传类型参数 默认Object

```java
public interface AI extends GenericInterface<String>{
}
```

# 泛型方法

## 泛型方法定义

定义语法

```java
public <类型参数> 返回类型 方法名（类型参数 变量名） {
    ...
}
```

举例

```java
public class MethodGeneric<T> {

    public T getValue(T value){
        return value;
    }
}

```

**泛型方法中可以同时声明多个类型参数**

```java
public class MethodGeneric<T> {

    public <T,S> T union(T value,S values){
        return value + values;
    }
}
```

**泛型方法也可以使用泛型类中定义的泛型参数**

```java
public class MethodGeneric<T> {

    public  void union(T value){
        System.out.println("T is:" + value);
    }
}
```

泛型类中定义的类型参数和泛型方法中定义的类型参数是相互独立的，他们之间没有任何关系

**定义静态方法泛型**

```java
public class MethodGeneric<T> {
    // 重新定义的类型参数
    public static <T> T show1(T one) { // 编译成功
        return null;
    }

    public static T show2(T one){ // 编译错误
        return null;
    }
}
```

## 泛型方法的使用

```java
public class DEMO1 {
    public static void main(String[] args) {
        GenericMethod d = new GenericMethod(); // 创建 GenericMethod 对象
        String str = d.fun("汤姆"); // 给GenericMethod中的泛型方法传递字符串
        int i = d.fun(30);  // 给GenericMethod中的泛型方法传递数字，自动装箱
        System.out.println(str); // 输出 汤姆
        System.out.println(i);  // 输出 30
        GenericMethod.show("Lin");// 输出: 静态泛型方法 Lin
    }
}
class GenericMethod {
    // 普通的泛型方法
    public <T> T fun(T t) { // 可以接收任意类型的数据
        return t;
    }
    // 静态的泛型方法
    public static <E> void show(E one) {
        System.out.println("静态泛型方法 " + one);
    }
}
```



泛型类：创建类对象的时候确认类型参数的具体类型

泛型方法：在调用方法的时候在确认类型参数的具体类型



## 泛型方法的类型推断

泛型方法中有多个类型参数，在调用的时候不指定类型参数的情况，方法中声明的类型参数为泛型方法中的几种类型参数的共同父类的最顶，一直到Object。

```java
public class Test {

    // 这是一个简单的泛型方法  
    public static <T> T add(T x, T y) {  
        return y;  
    }

    public static void main(String[] args) {  
        // 一、不显式地指定类型参数
        //（1）传入的两个实参都是 Integer，所以泛型方法中的<T> == <Integer> 
        int i = Test.add(1, 2);

        //（2）传入的两个实参一个是 Integer，另一个是 Float，
        // 所以<T>取共同父类的最小级，<T> == <Number>
        Number f = Test.add(1, 1.2);

        // 传入的两个实参一个是 Integer，另一个是 String，
        // 所以<T>取共同父类的最小级，<T> == <Object>
        Object o = Test.add(1, "asd");

        // 二、显式地指定类型参数
        //（1）指定了<T> = <Integer>，所以传入的实参只能为 Integer 对象    
        int a = Test.<Integer>add(1, 2);

        //（2）指定了<T> = <Integer>，所以不能传入 Float 对象
        int b = Test.<Integer>add(1, 2.2);// 编译错误

        //（3）指定<T> = <Number>，所以可以传入 Number 对象
        // Integer 和 Float 都是 Number 的子类，因此可以传入两者的对象
        Number c = Test.<Number>add(1, 2.2); 
    }  
}
```

# 类型擦除

泛型信息只会存在于编译的阶段，在进入JVM执行之前，泛型的信息会被擦除掉。指的是在编译过程中泛型类型的类型参数会被移除掉，并替换为他们限定的类型或者原始类型（即Object                                                                                                                                                                                                                                                     ）。

**类型擦除的过程**

+ 泛型类型参数会被替代成他们限定的类型，如果类型参数没有指定参数，则使用Object作为替代
+ 类型检查：编译器会插入强制类型转换来保证线程安全
+ 擦除类型参数信息：在编译后的字节码中，泛型类型参数的相关信息被移除掉

泛型类

```java
public class Box<T> {
    private T content;

    public void set(T content) {
        this.content = content;
    }

    public T get() {
        return content;
    }
}
```

这个泛型类被编译成字节码的时候，发生类型擦除，编译器会将T替换成Object

```java
public class Box {
    private Object content;

    public void set(Object content) {
        this.content = content;
    }

    public Object get() {
        return content;
    }
}
```



**类型擦除带来的影响**

+ 类型信息丢失：运行阶段没有办法获取到泛型的类型参数信息，例如没有办法直接判断List对象是List<String>还是List<Integer>
+ 泛型方法重载：由于类型会被擦除，不能仅仅通过类型参数的不同来重载方法。例如：

```java
public void print(List<String> list){
    ......
} 

public void print(List<Integer> list){
    ......
} 
上面的两个方法在编译之后变成如下样子
public void print(List<Object> list){
    ......
} 
```



**类型擦除的好处**

兼容性：允许新的代码和旧的Java代码库兼容，不需要重新编译旧代码就可以使用泛型

更方便迁移：开发者可以逐步将非泛型代码迁移到泛型，而不是一次性重写

# 类型的通配符

**什么是类型的通配符？**

类型通配符一般使用 ？代替，代替的是具体的类型实参

看一个例子：

```java
public class Box <E>{
    private E first;

    public E getFirst() {
        return first;
    }

    public void setFirst(E first) {
        this.first = first;
    }
}

```

按照多态的思想去理解，Integer 是继承 Number，传Number的子类按理来说应该不会报错。所以这里不可以按照多态的思维去理解。

![](https://cdn.nlark.com/yuque/0/2024/png/12477403/1720260267452-fac059c7-834e-406e-9242-e8fd4e1167d9.png)

这个问题如何解决，这个使用就得用到通配符了

![](https://cdn.nlark.com/yuque/0/2024/png/12477403/1720260443413-04c44fc5-0fe1-4f7b-b0ed-31ee022fa397.png)

## 类型通配符上限

语法：

```java
类/接口<? extends 实参类型>
```

要求该泛型的类型，只可以是实参类型或实参类型的子类类型

上面的案例showBox方法，使用了泛型但是还是用了Object去接受参数这样很不优雅，改造结果如下

![](https://cdn.nlark.com/yuque/0/2024/png/12477403/1720261556886-427e3aba-6490-4bf6-abf6-b7fcbda72b9a.png)



## 类型通配符下限

语法：

类/接口<? super 实参类型>

要求该泛型的类型，只能是实参类型，或者实参类型的父类类型

![](https://cdn.nlark.com/yuque/0/2024/png/12477403/1720262544227-9fad5124-a3c1-4e82-8127-42044c8b1ed0.png)

使用下限通配符，元素的接受只能是Object，因为所有元素的根就是Object
