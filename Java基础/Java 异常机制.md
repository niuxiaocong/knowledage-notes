介绍

Java 中的异常处理机制用来处理程序中出现异常的代码，让异常处理和正常业务逻辑分离。异常机制通过5个关键字**try**、**catch**、 **finally**、**throw**和**throws**完成。

# 异常的体系结构

![](https://cdn.nlark.com/yuque/0/2024/png/12477403/1724395887518-590f2d81-188d-4257-bcf2-e3383087b134.png)

所有的异常类都继承自Throwable

Throwable 下面分成了两个子类分别是 Error 和 Exception。

Error：表示系统级别的错误，是严重的、不可恢复的错误，Error错误必然会导致程序中断。所以，没有必要捕获Error对象。

Exception：是指在程序正常运行过程中，可以预料的意外情况，可以事先捕获并且处理。



Exception 异常又分成了两类：分别是**RunTime**异常和**Checked**异常。

所有RuntimeException类及其下面的子类被称之为Runtime异常，非RuntimeException类及其子类的异常实例被称之为**Checked**（编译时异常 或 检查时异常）异常。



**Checked**（编译时异常）：这种异常类型是必须进行处理的，处理的方式可以使用try-catch 或者 throws。

**RuntimeException**（运行时异常）：Runtime 不需要显示的进行处理，如果不进行处理的话就相当于隐式的带有throws声明，程序会自动的把异常抛给上一级调用方。



# Checked 异常出现的意义

Runtime异常的出现很多时候是无法预料的，因为使用者很多时候不会按照程序员预期的方式操作程序。程序员很难提前通过try-catch的方式提供足够完备的异常处理。最好的办法就是当异常出现后直接抛给上一级。所以，java默认Runtime异常带有隐式的throws声明。



而Checked异常java认为是应该被处理(或修复)的，所以java要求程序员必须显式的处理Checked异常。比如直接修改源代码防止Checked异常的产生，或者即使没办法完全避免Checked异常的产生，也要做到完全清楚程序可能产生哪些Checked异常，并且将这些Checked异常通过try-catch或者throws语句清楚的处理。

这样做会让代码更繁琐，但是常常可以帮助程序员在编译阶段就避免Checked异常的产生。



# 异常产生的方式

java的异常本质是一个个的对象，不同类型的异常本质上来说就是不同的异常类，代码出现问题的时候，就会产生对应的异常对象。

**1、JVM自己生成的异常对象。**

如果出现异常的代码，JVM会根据代码的错误类型自动生成对应的异常对象。比如数组下标越界访问的话，会自动生成**ArrayIndexOutOfBoundsException**异常对象。

**2、程序员主动通过throw语句生成的异常对象。**

有的异常属于不符合[业务逻辑](https://so.csdn.net/so/search?q=%E4%B8%9A%E5%8A%A1%E9%80%BB%E8%BE%91&spm=1001.2101.3001.7020)，但代码本身不会自动抛异常，比如age的值小于0，这种情况需要程序员主动通过throw语句生成异常对象。最好是根据异常的类型自己定义一个异常类，让这个异常类继承Exception类或者最好是继承RuntimeException类。



# 异常的处理方式

异常的处理有两种：try-catch和throws

1、如果用try-catch块包住可能出现异常的代码，就是主动处理异常，异常对象会被传入对应的catch块，按照catch块中设定的逻辑处理该异常。

2、如果不想主动处理该异常，可以通过throws语句把异常对象抛给方法的调用者，让方法的调用者处理。如果方法的调用者是JVM，JVM就会直接打印异常信息并终止程序。注意：如果异常类型是RuntimeException类及其子类，如果不try-catch处理，系统会自动调用throws语句向上抛出，程序员可以不主动写throws语句。

也就是说，异常被抛出后，除了用try-catch块主动处理，就只能抛给上一级方法调用者。每个方法都有这2个选择，如果一直不用try-catch块主动处理，异常对象就会被抛给JVM处理。



# Try Catch Finally

try 块，捕捉可能会出现异常的代码

catch块，出现异常的代码进入到对应的catch块里面进行处理

finally块，用于回收try块打开的物理资源。



try-catch块用于处理代码中可能产生的异常，只要异常对象传入对应的catch块，程序就会执行相应的逻辑，程序就不会中断，会继续正常执行。

没有进行异常捕捉导致执行中断

![](https://cdn.nlark.com/yuque/0/2024/png/12477403/1724597395619-4a28db84-265a-4b17-b148-183fbffe92c7.png)

进行异常捕捉代码继续执行

![](https://cdn.nlark.com/yuque/0/2024/png/12477403/1724597492013-f9fdb252-a0c5-4a21-89ff-d8e7d365235b.png)

Finall块是一定会执行的，并且Finall块需要放到try-catch后面也就是整个块的后面

即便是在try块或者catch块里面执行了return，也会先跳到Finally块下面然后执行完以后在执行最后一条return语句。

(特殊情况：System.exit(1); 这行语句可以强制退出虚拟机。只有在try块或者catch块中调用了这个方法，finally块才不会得到执行。)

因此，尽量避免在finally块中使用return或throw等导致方法终止的语句，否则可能会导致try块或者catch块的return语句或者throw语句失效。

不要被这个结果误导，这个实际的打印结果是10；

因为会首先执行try块里面的代码，并且记录下来这个返回结果，但是不会立马返回，执行完finally块里面的代码以后，然后在返回这个被记录下来的结果。

![](https://cdn.nlark.com/yuque/0/2024/png/12477403/1724598289846-73280abf-6685-4694-91ad-0412ef01b73e.png)

# Throw 和 Throws

**Throw**异常的业务逻辑。这种异常系统没有办法抛出，需要开发者自己定义。

语法格式：throw new xxxException。

注意：throw是程序员自己抛出异常，throws是声明方法可能会抛出的异常，两者的应用场合完全不一样。



**Throws**很多时候当前方法并不知道如何明确处理可能抛出的异常代码，就只能把可能出现的异常抛给上一级调用者。此时需要通过throws声明该方法可能会抛出的异常，如果异常对象产生，就把异常抛给上一级。

throws语法格式：throws ExceptionClass1、ExceptionClass2。

throws可以声明多个可能抛出的异常类型，throws只能出现在方法签名中，在这个方法里面产生对应的异常对象，就直接抛给上一级的调用者。</font>
