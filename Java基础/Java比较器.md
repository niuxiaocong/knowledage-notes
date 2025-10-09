比较器类型两种分别是Comparable（自然排序） 和 Comparator（定制排序）

# Comparable 自然排序
Comparable是一个接口，如果想要实现比较需要实现这个接口并且重写里面的compareTo方法，实现两个对象的大小比较，即this当前对象和要比较的对象。

如果当前对象this大于形参对象obj返回正数，当前对象this小于形参对象obj返回负数，如果两个对象相等则返回0。

自定义类实现Comparable自然排序

```java
public class Person implements Comparable<Person>{
    private String name;
    private int age;
    public Person() {
    }
    public Person(String name, int age) {
        this.name = name;
        this.age = age;
    }
    public String getName() {
        return name;
    }
    public void setName(String name) {
        this.name = name;
    }
    public int getAge() {
        return age;
    }
    public void setAge(int age) {
        this.age = age;
    }
    @Override
    public String toString() {
        return "Person{" +
        "name='" + name + '\'' +
        ", age='" + age + '\'' +
        '}';
    }
    @Override
    public int compareTo(Person o) {

        if (this.age > o.age){
            return 1;
        }else if (this.age < o.age){
            return -1;
        }else{
            return 0;
        }
    }
}
```

客户端调用

```java
public class ComparebleDemo {
    public static void main(String[] args) {
        Person[] peoples = new Person[3];
        peoples[0] = new Person("AA",18);
        peoples[1] = new Person("CC",45);
        peoples[2] = new Person("JJ",7);
        Arrays.sort(peoples);
        System.out.println(Arrays.toString(peoples));

    }
}
```

# Comparator 定制排序
应用场景：当对象没有实现Comparable接口并且不方便修改代码，或者说是实现了Comparable接口的排序规则不适合当前的操作那么就可以使用Comparator重新进行排序



需要重写compare方法比较两个对象的大小o1和o2 ，o1大于o2返回正数，o1小于o2返回负数，相等返回0

```java
public class ComparatorDemo {
    public static void main(String[] args) {
        String[] strs = new String[]{"AA","CC","JJ","DD","BB"};
        Arrays.sort(strs, new Comparator<String>() {
            @Override
            public int compare(String o1, String o2) {
                return -o1.compareTo(o2);
            }
        });
        System.out.println(Arrays.toString(strs));
    }
}
```

