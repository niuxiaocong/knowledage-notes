# 介绍

Lambda 是JDK1.8中的新特性，使用Lambda 表达式可以使代码更加紧凑简洁、但是也增加了阅读、代码调试的难度。

Lambda 实际上是 由匿名函数改造而来，看下面例子，开启一个线程 使用传统写法和使用Lambda的写法有何不同。

传统写法：

```java
public class Lambda {
    public static void main(String[] args) {
        // 传统写法
        Runnable runnable = new Runnable() {
            @Override
            public void run() {
                System.out.println("Hello World!");
            }
        };
        // 调用
        runnable.run();
    }
}
```

使用Lambda

```java
public class Lambda {
    public static void main(String[] args) {
        Runnable lambdaRunnable = () -> System.out.println("Hello Lambda!");
        lambdaRunnable.run();
    }
}
```

在代码上减少了很多行，而且看起来非常简洁。

# 函数式接口

简单点说就是一个接口里面只能有一个未实现的方式，并且在接口类上标注了<font style="color:#000000;">@FunctionalInterface注解</font>

```java
@FunctionalInterface
interface NoParameterNoReturn {
    //注意：只能有一个方法
    void test();
}
```

**常用的几个函数式接口**

Consumer<T> 消费型接口   

Supplier<T> 供给型接口

Function<T,R> 函数型接口

Predicate<T> 断言型接口

# 使用语法方式

Lambda 的基础格式语法：（参数列表）-> { 方法体 }

（） 这里面是参数：括号中只有一个参数，括号可以省略，括号中不用写参数类型（编译器会自己推断）；

  {} 这个里面是写的方法体，如果方法体里面只有一句不管有没有返回值，都可以进行省略。

Lambda的基本使用

```java
public class LambdaBase {
    public static void main(String[] args) {
        NoParameterNoReturn n = () -> System.out.println("无参 无返回");
        n.call();
        OneParameterNoReturn o = (a) -> System.out.println("一个参数 无返回" + a);
        o.call(1);
        MoreParameterNoReturn m = (a,b) -> System.out.println("两个参数 无返回" + a + " " + b);
        m.call(1,2);


        NoParameterReturn n2 = () -> 1;
        System.out.println("无参数，有返回：" + n2.call());
        OneParameterReturn o2 = (a) -> a;
        System.out.println("一个参数，有返回：" + o2.call(1));
        MoreParameterReturn m2 = (a,b) -> a + b;
        System.out.println("两个参数，有返回：" + m2.call(1,2));

    }
}

// 无参数，无返回
@FunctionalInterface
interface NoParameterNoReturn{
    void call();
}
// 一个参数，无返回
@FunctionalInterface
interface OneParameterNoReturn {
    void call(int a);
}
// 两个参数，无返回
@FunctionalInterface
interface MoreParameterNoReturn {
    void call(int a,int b);
}
// 有返回值无参数
@FunctionalInterface
interface NoParameterReturn {
    int call();
}
// 有返回值一个参数
@FunctionalInterface
interface OneParameterReturn {
    int call(int a);
}
// 有返回值多个参数
@FunctionalInterface
interface MoreParameterReturn {
    int call(int a,int b);
}
```

# 并行流

可以利用CPU多核的特性来进行加速处理，当一个流并行处理时候，数据分成多个部分，然后进行多个线程进行处理，处理完成以后再将结果合并起来。

```java
public class JavaParalleStream {
    public static void main(String[] args) {
        List<Integer> list = Arrays.asList(1, 2, 3, 4, 5, 6);
        Stream<Integer> parallelStream = list.parallelStream();

        // forEach – 并行执行
        list.parallelStream().forEach(e -> System.out.print(e+ " "));
        System.out.println();
        // filter – 并行过滤
        list.parallelStream().filter(e -> e > 2).forEach(e -> System.out.print(e + " "));
        System.out.println();
        // map – 并行映射
        list.parallelStream().map(e -> e * 2).forEach(e -> System.out.print(e + " "));
        System.out.println();
        // reduce – 并行归约
        Integer sum = list.parallelStream().reduce(0, (a, b) -> a + b);
        // sorted – 并行排序
        list.parallelStream().sorted().forEach(e -> System.out.print(e + " "));
        System.out.println();
        System.out.println(sum);
    }
}
```

通过并行流可以让许多常见的流操作利用多核 CPU 进行并行计算，从而获得更高的性能。但是，并不意味着所有的流操作都适合并行化，如果流处理的程序本身不是 CPU 密集型的，那么并行化可能得不到太大的性能提升，甚至有所下降。

# Lambda 常用操作

## 并行求和计算

```java
public class LambdaSum {
    public static void main(String[] args) {
        List<String> values = Arrays.asList("1", "2", "3", "4", "5");
        int sum = values.parallelStream().mapToInt(item -> Integer.parseInt(item)).sum();
        System.out.println("求和计算的结果是："+sum);
    }
}
```

## ArrayList 排序

```java
public class LambdaArrayListSort {
    public static void main(String[] args) {
        List<Person> list = new ArrayList<>();
        list.add(new Person("小蓝",10));
        list.add(new Person("小红",11));
        list.add(new Person("小紫",18));
        list.add(new Person("小橙",20));
        list.sort((p1,p2) -> p1.getAge() - p2.getAge());
        for (Person person : list) {
            System.out.println(person.toString());
        }
    }
}

```

## TreeSet 排序

```java
public class LambdaTreeSetSort {
    public static void main(String[] args) {
        TreeSet<Person> set = new TreeSet<>((o1, o2) -> {
            if(o1.getAge() >= o2.getAge()){
                return -1;
            }else {
                return 1;
            }
        });
        set.add(new Person("小蓝",10));
        set.add(new Person("小红",11));
        set.add(new Person("小紫",18));
        set.add(new Person("小橙",20));
        System.out.println(set);
    }
}
```

## forEach 遍历输出

```java
public class LambdaForeach {
    public static void main(String[] args) {
        List<Integer> list = Arrays.asList(1, 2, 3, 4, 5, 6, 7, 8, 9);

        // 传统方式
        for (Integer integer : list) {
            System.out.println(integer);
        }

        // lambda
        list.forEach(System.out::println);
    }
}
```

## RemoveIf 删除元素

```java
public class LambdaRemoveIf {
    public static void main(String[] args) {
        List<Person> list = new ArrayList<>();
        list.add(new Person("小蓝",10));
        list.add(new Person("小红",11));
        list.add(new Person("小紫",10));
        list.add(new Person("小橙",20));

        // 这样写是不对滴  Exception in thread "main" java.util.ConcurrentModificationException
//        for (Person person : list) {
//            if (person.getAge() == 10){
//                list.remove(person);
//            }
//        }

        // 迭代器删除
        Iterator<Person> it = list.iterator();
        while(it.hasNext()){
            Person person = it.next();
            if (person.getAge() == 10){
                it.remove();
            }
        }

        //使用lambda实现
        //将集合中的每一个元素都带入到test方法中，如果返回值是true将删除
        list.removeIf(t -> {
            if (t.getAge() == 10){
                return true;
            }else {
                return false;
            }
        });
        //简化
        list.removeIf(t -> t.getAge()==10);
        System.out.println(list);
    }
}
```

## filter 过滤操作

```java
public class LambdaFilter {
    public static void main(String[] args) {
        List<Integer> list = Arrays.asList(1, 2, 3, 4, 5, 6, 7, 8, 9);
        // 使用Lambda表达式和Stream API过滤出偶数
        list.stream().filter(n -> n % 2 == 0).forEach(System.out::println);
    }
}
```

## distinct 去重

```java
public class LambdaDistinct {
    public static void main(String[] args) {
        List<Integer> list = Arrays.asList(1, 2, 3, 4, 5, 6, 7, 8, 9,1,2,3);
        list.stream().distinct().forEach(e -> System.out.println(e));

        // 简写
        list.stream().distinct().forEach(System.out::println);
    }
}
```

## map映射成一个新的元素

```java
public class LambdaMapRef {
    public static void main(String[] args) {
        List<Integer> list = Arrays.asList(1, 2, 3, 4, 5, 6, 7, 8, 9);
        // 使用Stream的map方法对列表中的每个元素进行操作
        List<Integer> newList = list.stream().map(item -> item * 2).collect(Collectors.toList());
        System.out.println(newList);
    }
}
```

## limit 取前几条数据

```java
public class LambdaLimit {
    public static void main(String[] args) {
        List<Integer> list = Arrays.asList(1, 2, 3, 4, 5, 6, 7, 8, 9);
        // 使用limit方法获取前3个元素
        list.stream().limit(3).forEach(System.out::println);
        System.out.println("---------------------");
        // 使用collect方法将limit操作后的结果收集到一个新的列表中
        List<Integer> newList = list.stream().limit(3).collect(Collectors.toList());
        System.out.println(newList);
    }
}
```

## skip跳过前几条数据

```java
public class LambdaSkip {
    public static void main(String[] args) {
        List<Integer> list = Arrays.asList(1, 2, 3, 4, 5, 6, 7, 8, 9);
        // 跳过前3个元素
        list.stream().skip(3).forEach(System.out::println);

        // 跳过前3个元素，并获取新的列表
        List<Integer> newList = list.stream().skip(3).collect(Collectors.toList());
        System.out.println(newList);

    }
}
```

## flaMap 合并流

吧元素中的每一个元素都映射成一个流，然后再把这些流连接成一个流

```java
public class LambdaFlatMap {
    public static void main(String[] args) {
        List<People> peopleList = new ArrayList<>();
        List<People> friendList = new ArrayList<>();
        friendList.add(new People("小红",11));
        friendList.add(new People("小紫",10));
        friendList.add(new People("小橙",20));
        peopleList.add(new People("小蓝",10,friendList));
        peopleList.stream().flatMap(people -> people.getFriendList().stream()).forEach(friend-> System.out.println(friend.getName()));
    }
}
```

## count 计算数量

```java
public class LambdaCount {
    public static void main(String[] args) {
        List<Integer> list = Arrays.asList(1, 2, 3, 4, 5, 6, 7, 8, 9);
        long count = list.stream().count();
        System.out.println(count);
    }
}
```

## max计算最大和min计算最小

```java
public class LambdaMinMax {
    public static void main(String[] args) {
        List<Integer> list = Arrays.asList(1, 2, 3, 4, 5, 6, 7, 8, 9);

        List<Person> personList = new ArrayList<>();

        personList.add(new Person("小蓝",10));
        personList.add(new Person("小红",11));
        personList.add(new Person("小紫",10));
        personList.add(new Person("小橙",20));


        Person personMax = personList.stream().max((o1, o2) -> o1.getAge() - o2.getAge()).get();
        Person personMin = personList.stream().min((o1, o2) -> o1.getAge() - o2.getAge()).get();

        System.out.println("最大年龄：" + personMax.getAge() + ", 最小年龄：" + personMin.getAge());


        // 简写
        Integer min = list.stream().min(Integer::compareTo).get();
        Integer max = list.stream().max(Integer::compareTo).get();
        System.out.println("min: " + min + ", max: " + max);
    }
}
```

## collect转换成新集合

## Collectors.toList() 转list

```java
public class LambdaToList {
    public static void main(String[] args) {
        List<Person> personList = new ArrayList<>();
        personList.add(new Person("小蓝",10));
        personList.add(new Person("小红",11));
        personList.add(new Person("小紫",10));
        personList.add(new Person("小橙",20));
        List<Person> newList = personList.stream().filter(item -> item.getAge() == 10).collect(Collectors.toList());
        newList.forEach(System.out::println);
    }
}
```

## Collectors.toMap() 转map

```java
public class LambdaToMap {
    public static void main(String[] args) {
        List<Person> personList = new ArrayList<>();
        personList.add(new Person("小蓝",10));
        personList.add(new Person("小红",11));
        personList.add(new Person("小紫",15));
        personList.add(new Person("小橙",20));
        Map<Integer, String> map = personList.stream().distinct().collect(Collectors.toMap(person -> person.getAge(), person -> person.getName()));
        System.out.println(map);
    }
}
```

## Collectors.toSet() 转set

```java
public class LambdaToSet {
    // 记得Person对象里面要重写 hashCode 和 equals 方法
    public static void main(String[] args) {
        List<Person> personList = new ArrayList<>();
        personList.add(new Person("小蓝",10));
        personList.add(new Person("小红",11));
        personList.add(new Person("小紫",10));
        personList.add(new Person("小紫",10));
        personList.add(new Person("小橙",20));
        Set<Person> personSet = personList.stream().filter(item -> item.getAge() == 10).collect(Collectors.toSet());
        personSet.forEach(System.out::println);
    }
}
```

## Collectors.groupingBy() 分组

```java
public class LambdaGroupingBy {
    public static void main(String[] args) {
        List<Person> personList = new ArrayList<>();
        personList.add(new Person("小蓝",10));
        personList.add(new Person("小蓝",50));
        personList.add(new Person("小红",11));
        personList.add(new Person("小紫",15));
        personList.add(new Person("小橙",20));


        // 使用Lambda表达式和Stream API对personList进行分组
        Map<String, List<Person>> listMap = personList.stream().collect(Collectors.groupingBy(person -> person.getName()));
        System.out.println(listMap);
    }
}
```

## Collectors.joining() 转字符串

```java
public class LambdaJoinging {
    public static void main(String[] args) {
        List<Person> personList = new ArrayList<>();
        personList.add(new Person("小蓝",10));
        personList.add(new Person("小红",11));
        personList.add(new Person("小紫",15));
        personList.add(new Person("小橙",20));
        String names = personList.stream().map(person -> person.getName()).collect(Collectors.joining(","));
        System.out.println(names);
    }
}
```

## anyMatch只要有一条数据满足条件即返回true

```java
public class LambdaAnyMatch {
    public static void main(String[] args) {
        List<Person> personList = new ArrayList<>();
        personList.add(new Person("小蓝",10));
        personList.add(new Person("小红",11));
        personList.add(new Person("小紫",15));
        personList.add(new Person("小橙",20));

        // 以people集合中是否有年龄大于15岁的，有一个结果就为true
        boolean flag = personList.stream().anyMatch(person -> person.getAge() > 15);
        System.out.println(flag);
    }
}
```

## allMatch必须全部都满足条件才会返回true

```java
public class LambdaAllMatch {
    public static void main(String[] args) {
        List<Person> personList = new ArrayList<>();
        personList.add(new Person("小蓝",10));
        personList.add(new Person("小红",11));
        personList.add(new Person("小紫",15));
        personList.add(new Person("小橙",20));

        // 以people集合中是否有年龄大于15岁的，有一个结果就为true
        boolean flag = personList.stream().allMatch(person -> person.getAge() > 5);
        System.out.println(flag);
    }
}
```

## noneMatch全都不满足条件才会返回true

```java
public class LambdaNoneMatch {
    public static void main(String[] args) {
        List<Person> personList = new ArrayList<>();
        personList.add(new Person("小蓝",10));
        personList.add(new Person("小红",11));
        personList.add(new Person("小紫",15));
        personList.add(new Person("小橙",20));

        // 以people集合中是否有年龄大于15岁的，有一个结果就为true
        boolean flag = personList.stream().noneMatch(person -> person.getAge() > 20);
        System.out.println(flag);
    }
}
```

## findFirst获取第一条数据

```java
public class LambdaFindFirst {
    public static void main(String[] args) {
        List<Person> personList = new ArrayList<>();
        personList.add(new Person("小蓝",10));
        personList.add(new Person("小红",11));
        personList.add(new Person("小紫",10));
        personList.add(new Person("小橙",20));
        Optional<Person> first = personList.stream().distinct().sorted((p1, p2) -> p1.getAge() - p2.getAge()).findFirst();
        first.ifPresent(System.out::println);
    }
}
```

## reduce根据指定的计算模型计算结果

```java
public class LambdaReduce {
    public static void main(String[] args) {
        List<Integer> list = Arrays.asList(1, 2, 3, 4, 5, 6, 7);

        // 带初始值的循环计算 第一个参数是每一次计算的结果，第二个参数是每一次输入进来的值 
        list.stream().reduce(0,(a, b) -> {
            System.out.println("a="+a+"  b="+b+"  a+b="+(a+b));
            return a + b;
        });
    }
}
```

# Optional

## Optional 创建空对象

```java
Person person = new Person();
Optional<Person> optional = Optional.ofNullable(person);
```

## 安全消费 ifPresent

消费对象为空，不执行

```java
public class LambdaIfPresent {
    public static void main(String[] args) {
        List<Person> personList = new ArrayList<>();
        personList.add(new Person("小蓝",10));
        personList.add(new Person("小红",11));
        personList.add(new Person("小紫",10));
        personList.add(new Person("小橙",20));
        Person person = null;

        // person 为空
        Optional<Person> optional = Optional.ofNullable(person);
        optional.ifPresent(p -> System.out.println("找到了：" + p.getName()));

        // person 不为空
        Person personNotNull = personList.get(0);
        Optional.ofNullable(personNotNull).ifPresent(p -> System.out.println("找到了：" + p.getName()));
    }
}
```

## 安全获取 OrElseGet

获取对象为空，不执行

```java
public class LambdaOrElseGet {
    public static void main(String[] args) {
        List<Person> list = new ArrayList<>();
        list.add(new Person("小蓝",10));
        list.add(new Person("小红",11));
        list.add(new Person("小紫",10));
        list.add(new Person("小橙",20));

        Person peopleNull = null;
            Optional<Person> optional = Optional.ofNullable(peopleNull);
        Person person = optional.orElseGet(() -> null);
        System.out.println(person);


        Person personNotNull = list.get(0);
        Optional<Person> optionalPerson = Optional.ofNullable(personNotNull);
        Person pp = optionalPerson.orElseGet(() -> null);
        System.out.println(pp);
    }
}
```

## 安全过滤 Filter

```java
public class LambdaFilterSafe {

    public static void main(String[] args) {
        List<Person> list = new ArrayList<>();
        list.add(new Person("小蓝",10));
        list.add(new Person("小红",11));
        list.add(new Person("小紫",10));
        list.add(new Person("小橙",20));

        Person personNull = null;
        Optional<Person> optional = Optional.ofNullable(personNull);
        optional.filter(person -> person.getAge() > 10).ifPresent(person -> System.out.println(person.getName()));

        Person personNotNull = list.get(0);
        Optional<Person> optionalList = Optional.ofNullable(personNotNull);
        optionalList.filter(person -> person.getAge() == 10).ifPresent(person -> System.out.println(person.getName()));

        // Optional 包装的 List对象如何变成流进行处理
        Optional<List<Person>> personList = Optional.ofNullable(list);
        List<Person> collect = personList.orElseGet(() -> null).stream().collect(Collectors.toList());
        System.out.println(collect);
    }
}
```

# 在什么情况下可以简写成双冒号

双冒号代表的是方法引用，相当于是一种代码的简写方式

双冒号有以下几种使用场景

## 静态方法引用

语法：类::静态方法

注意点：在引用的后面不用添加小括号、使用静态引用 方法的参数数量、类型和返回值，必须和接口定义的一致

![](https://cdn.nlark.com/yuque/0/2024/png/12477403/1719737060765-1c66082f-b6f1-4605-8ded-e5da2b77a52a.png)

## 非静态方法引用

语法：对象::方法名

注意点：在引用的后面不用添加小括号、使用静态引用 方法的参数数量、类型和返回值，必须和接口定义的一致

![](https://cdn.nlark.com/yuque/0/2024/png/12477403/1719737613619-7e4c1e45-0049-4aa5-8b17-7470db462cd8.png)

## 引用类构造方法

语法：类名::new

![](https://cdn.nlark.com/yuque/0/2024/png/12477403/1719738558706-4eb4ff71-20bc-46c3-9c29-963b3403115a.png)

## 引用数组构造方法

语法：数组类型[]::new

![](https://cdn.nlark.com/yuque/0/2024/png/12477403/1719739388100-bc4c4e4a-44a6-49ee-a276-19b9c606a30b.png)


