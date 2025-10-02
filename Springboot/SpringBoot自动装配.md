在了解自动装配之前，先看一下Spring的发展，Spring的发展大致上可以分成三个阶段，第一个阶段是基于配置文件<font style="color:rgb(51, 51, 51);">也就是在该版本中我们必须要提供xml的配置文件，在该文件中我们通过 </bean> 标签来配置需要被IoC容器管理的Bean，第二个阶段是基于注解和配置文件的形式，这个时候并没有完全脱离配置文件，第三个阶段是基于注解驱动的形式，可以完全脱离配置文件的形式。

![](https://cdn.nlark.com/yuque/0/2025/png/12477403/1758599181581-6458ff0c-2773-420f-981c-f3d189aa9659.png)

# 三个阶段

## 第一阶段（基于配置文件）

主要是Spring1.0版本，这个版本的注解比较少，例如@Transation注解

**演示案例**

<font style="color:rgb(51, 51, 51);">spring xml 配置</font>

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE beans PUBLIC "-//SPRING//DTD BEAN//EN" "http://www.springframework.org/dtd/spring-beans.dtd">
<beans>
  <bean id="userService" class="com.nyc.boot.UserService"></bean>
</beans>
```

pom.xml

packaging 标签不要写成pom，会导致resource目录编译不成功

```xml
<dependencies>
  <!-- spring-context -->
  <dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-context</artifactId>
    <version>1.2.9</version>
  </dependency>
</dependencies>
```

spring获取容器里面的内容

```java
public class AppStat {
    public static void main(String[] args) {
        ClassPathXmlApplicationContext ac = new ClassPathXmlApplicationContext("application.xml");
        System.out.println("ac.getBean(\"userService\"):=======" + ac.getBean("userService"));
    }
}
```

## 第二阶段（基于配置文件+注解）

主要是Spring2.0版本，在这个版本里面衍生出了新的注解@Required,@Repository,@Aspect,@Autowired,@Qualifier,@Component,@Service,@Controller,@RequestMapping等

**演示案例**

Service类

```java
@Service
public class UserService {}
```

Spring配置文件，通过component-scan标签来扫描标记了@Component注解的类

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
  xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xmlns:context="http://www.springframework.org/schema/context"
  xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context.xsd">

  <context:component-scan base-package="com.nyc.boot"></context:component-scan>
</beans>
```

获取spring容器里面的内容

```java
public class ApplicationMain {
    public static void main(String[] args) {
        ApplicationContext context = new ClassPathXmlApplicationContext("applicationContext.xml");
        System.out.println("context.getBean(\"userService\") === " + context.getBean("userService"));
    }
}
```

## 第三阶段（基于注解）

Spring3.0 ~ 至今，这个阶段衍生出来比较重要的几个注解** @Import、@configuration、@ComponentScan、@ImportResource、@Conditional**

**重要注解解释说明**

+ @configuration：这个注解的作用主要就是代替XML配置文件

![](https://cdn.nlark.com/yuque/0/2025/png/12477403/1758623764204-8319232a-3d8d-4885-baa3-0d2669515efc.png)

+ @ComponentScan：这个注解的作用主要是为了替换掉<component-scan></component-scan> 标签，默认扫描的是当前注解所修饰的Java类所在的包及其子包下所有的 @Component注解修饰的Java类

![](https://cdn.nlark.com/yuque/0/2025/png/12477403/1758623848825-af881e03-b56a-483a-b3ac-4d61b91702d7.png)

**演示案例**

配置类

```java
@Configuration //Configuration 这个注解相当于 spring的配置文
@ComponentScan(basePackages = "com.nyc.boot.service")  // 默认扫描的是，当前注解所修饰的java类所在的包及其子包下所有的 @Component注解修饰的Java类,3.1版本的这个注解必须指明最少一个包路径
public class JavaConfig {
}
```

实体类

```java
@Service
public class UserService {}
```

从spring容器中读取内容

```java
public class ApplicationMain {
    public static void main(String[] args) {
        ApplicationContext ac = new AnnotationConfigApplicationContext(JavaConfig.class);
        UserService bean = ac.getBean(UserService.class);
        System.out.println("ac.getBean(UserService.class); =======" + bean);
    }
}
```

+ @ImportResource：主要用来加载XML配置，在使用完全基于注解的Spring项目时，仍然可以复用之前写好的XML配置。比如项目原来用XML配置了很多的Bean，现在需要迁移到SpringBoot或者要完全基于注解的方式，如果不想一次性全部改成@Bean或者@Component注解标注的形式，可以使用@ImportResourse注解用来过渡。<font style="color:rgb(51, 51, 51);">3.0 版本并没有完全脱离xml，提供@ImportResource，3.1版本提供了@ComponentScan，在这个版本才可以完全脱离了xml</font>

**演示案例**

配置类

```java
@Configuration //Configuration 这个注解相当于 spring的配置文件
@ImportResource("application.xml") //这个注解可以关联到对应的配置文件
public class JavaConfig {}
```

实体类

```java
@Service
public class UserService {}
```

Spring配置文件

```java
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">
    <bean id="userService" class="com.nyc.boot.service.UserService"></bean>
</beans>
```

从容器中读取

```java
public class ApplicationMain {
    public static void main(String[] args) {
        ApplicationContext ac = new AnnotationConfigApplicationContext(JavaConfig.class);
        UserService bean = ac.getBean(UserService.class);
        System.out.println("ac.getBean(UserService.class); =======" + bean);
    }
}
```

+ @Import：@Import注解的目的是为了替换掉XML配置文件里面的Import标签，作用就是导入第三方配置类。

**演示案例**

配置类

```java
@Configuration
public class MyBatisConfig {
    @Bean
    public OrderServcie orderServcie(){
        return new OrderServcie();
    }
}
```

```java
@Configuration
public class RedisConfig {
    @Bean
    public StudentService studentService(){
        return new StudentService();
    }
}
```

通过Import注解导入上面的两个配置类

```java
@Configuration //Configuration 这个注解相当于 spring的配置文件
@Import({MyBatisConfig.class, RedisConfig.class})
public class JavaConfig {
    @Bean
    public UserService userService(){
        return new UserService();
    }
}
```

从容器中获取内容

```java
public class ImportApplicationMain {
    public static void main(String[] args) {
        ApplicationContext ac = new AnnotationConfigApplicationContext(JavaConfig.class);
        OrderServcie bean = ac.getBean(OrderServcie.class);
        StudentService bean2 = ac.getBean(StudentService.class);
        System.out.println("ac.getBean(OrderServcie.class):=====" + bean);
        System.out.println("ac.getBean(StudentService.class):======" + bean2);
    }
}
```



扩展：导入三方的配置类，可以使用importSelector和<font style="color:rgb(51, 51, 51);">ImportBeanDefinitionRegistrar</font>

**使用****importSelector方式导入三方的配置类**

**演示案例**

实体类

```java
public class Cache {
}
```

```java
public class Logger {
}
```

实现importSelector接口

```java
public class MyImportSelector implements ImportSelector {
    @Override
    public String[] selectImports(AnnotationMetadata importingClassMetadata) {
        return new String[]{Logger.class.getName(),Cache.class.getName()};
    }
}
```

查看这个Bean的注入情况

```java
@Configuration
@Import(MyImportSelector.class)
public class JavaConfig2 {
    public static void main(String[] args) {
        ApplicationContext ac = new AnnotationConfigApplicationContext(JavaConfig2.class);
        for (String beanDefinitionName : ac.getBeanDefinitionNames()) {
            System.out.println(beanDefinitionName);
        }
    }
}
```

![](https://cdn.nlark.com/yuque/0/2025/png/12477403/1740497593581-984a457b-0013-4834-8c3f-491ca37ffc96.png)

**使用****<font style="color:rgb(51, 51, 51);">ImportBeanDefinitionRegistrar</font>****方式导入三方的配置类**

**演示案例**

实现ImportBeanDefinitionRegistrar接口

```java
public class MyImportBeanDefinitionRegistrar implements ImportBeanDefinitionRegistrar {
    @Override
    public void registerBeanDefinitions(AnnotationMetadata importingClassMetadata
            , BeanDefinitionRegistry registry) {
        RootBeanDefinition logger =new RootBeanDefinition(Logger.class);
        registry.registerBeanDefinition("logger666",logger);

        RootBeanDefinition cache = new RootBeanDefinition(Cache.class);
        registry.registerBeanDefinition("cache111",cache);
    }
}
```

从Spring容器中获取内容

```java
@Configuration
@Import(MyImportBeanDefinitionRegistrar.class)
public class JavaConfig2 {

    public static void main(String[] args) {
        ApplicationContext ac = new AnnotationConfigApplicationContext(JavaConfig2.class);
        for (String beanDefinitionName : ac.getBeanDefinitionNames()) {
            System.out.println(beanDefinitionName);
        }
    }
}
```

**ImportSelector和****<font style="color:rgb(51, 51, 51);">ImportBeanDefinitionRegistrar这两种注入Bean的方式有什么优势和缺点？</font>**

ImportSelector：返回的是一组类的全限定名称，好处是简单高效不需要直接操作BeanDefinition，经常被用在条件装配，组合@Condition注解可以做到按需引入。缺点就是不太灵活，只能导入已经存在的配置类， 不能直接操作 BeanDefinition（比如动态生成 Bean、设置属性）  

<font style="color:rgb(51, 51, 51);">ImportBeanDefinitionRegistrar：好处是灵活，可以动态的操作BeanDefinition，可以动态决定Bean的定义方式（包括作用域、懒加载、构造参数等），缺点是操作起来复杂，维护成本高。</font>



+ @Condition： 条件注入，通过条件来判断是否需要注入Bean对象

**<font style="color:rgb(51, 51, 51);">演示案例</font>**

实体类

```java
public class DeptService {}
```

Condition条件判断

```java
public class ConditionOnClass implements Condition {
    @Override
    public boolean matches(ConditionContext context, AnnotatedTypeMetadata metadata) {
        // 条件判断，可以通过context的上下文来判断
        return false;
    }
}
```

配置类

```java
@Configuration
public class Demo04JavaConfig {
    @Conditional({ConditionOnClass.class})  // 判断条件返回true则注入，返回false则不进行注入
    @Bean
    public DeptService deptService(){
        return new DeptService();
    }

    public static void main(String[] args) {
        ApplicationContext ac = new AnnotationConfigApplicationContext(Demo04JavaConfig.class);
        String[] beanDefinitionNames = ac.getBeanDefinitionNames();
        for (String beanDefinitionName : beanDefinitionNames) {
            System.out.println(beanDefinitionName);
        }
    }
}
```

# Enable模块

在Spring/Spring Boot 里面，经常会看到一些 @EnableXXX注解，例如：@EnableScheduling、@EnableAsync、@EnableCaching、@EnableWebMvc、@EnableRedisRepositories等注解。这些个注解就使用到了Enable模块机制。Enable模块机制的本质是一个开关注解，用来启用某一个模块的功能，原理是使用到了@Import注解把一些配置类和Bean注册进容器开启对应的功能。可以理解成一个快捷开关，帮助我们快速导入一些配置和组件。

以@EnableAsync注解为例

![](https://cdn.nlark.com/yuque/0/2025/png/12477403/1758642931514-5ffe1954-1250-4752-ad33-cacb0cca76a7.png)



定义一个Enable模块的注解

```java
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.TYPE)
@Documented
@Inherited
@Import(RedisAutoConfiguration.class)
public @interface EnableAutoRedisConfiguration {
}
```

写一个reids的配置类

```java
@Configuration
public class RedisAutoConfiguration {
    @Bean
    public StudentService studentService(){
        return new StudentService();
    }
}
```

配置自定义的Enable

```java
@Configuration
@EnableAutoRedisConfiguration
public class Demo03JavaConfig {

}
```

读取spring容器中的内容

```java
public class Demo03StartApp {
    public static void main(String[] args) {
        ApplicationContext ac = new AnnotationConfigApplicationContext(Demo03JavaConfig.class);
        StudentService bean = ac.getBean(StudentService.class);
        System.out.println("ac.getBean(StudentService.class):=====" + bean);
    }
}
```

# SPI机制

SPI（Service Provider Interface）是一种服务发现机制，它允许框架或模块定义接口，然后由第三方实现接口，并通过配置的方式被框架自动发现和加载。

可以通俗理解成：框架只是负责约定“需要的功能（接口）”，具体实现交给外部，加载的时候框架会扫描配置文件，找到实现类并实例化。

JDK有自己的一套SPI机制，在java.util包下面的ServiceLoader问题，Spring采用的就是JDK默认的这一套SPI机制，SpringBoot对其进行了扩展。在SpringBoot中会从META-INF/spring.factories里面读取配置，然后按需加载，需要注意的是在SpringBoot2.7/3.x版本引入了新的实现方式用spring/org.springframework.boot.autoconfigure.AutoConfiguration.imports 文件替代了 spring.factories



## 实现Spring中的SPI

JDBC驱动案例，定义JDBC的驱动规范（“接口”），不同的数据库厂商只需要基于这套规范实现接口即可

![](https://cdn.nlark.com/yuque/0/2025/png/12477403/1740579959879-6b6f858f-41ea-4ba0-a6f6-f932f1d01b84.png)

新建服务spring-database-spi

```java
public interface Connection {
    String getUrl();
}
```

新建服务spring-mysql-database-api 并实现Connection接口

```java
public class MySQLConnection implements Connection{
    @Override
    public String getUrl() {
        System.out.println("MySQLConnection连接....");
        return null;
    }
}
```

新建服务spring-oracle-database-api 并实现Connection接口

```java
public class OracleConnection implements Connection {
    @Override
    public String getUrl() {
        System.out.println("OracleConnection连接......");
        return null;
    }
}
```

<font style="color:rgb(51, 51, 51);">新建服务spring-spi来应用不同数据库厂商的实现</font>

```java
public class StartAppSPI {
    public static void main(String[] args) {
        ServiceLoader<Connection> load = ServiceLoader.load(Connection.class);
        Iterator<Connection> iterator = load.iterator();
        while(iterator.hasNext()){
            Connection next = iterator.next();
            next.getUrl();
        }
    }
}
```

## 实现SpringBoot的SPI（封装自定义的Starter）

新建服务 redis-spring-boot-starter

从配置文件读取配置的类

```java
@Data
@ConfigurationProperties(prefix = "my.redisson")
public class RedissonProperties {
    private String password = "123456";
    private String host = "localhost";
    private Integer port = 6379;
    private int timeout = 1000;
    private boolean ssl = false;
}
```

redis连接配置类

```java
@ConditionalOnClass(Redisson.class)
@EnableConfigurationProperties(RedissonProperties.class)
@Configuration
public class RedissonAutoConfiguration {
    @Bean
    public RedissonClient redissonClient(RedissonProperties redissonProperties){
        Config config = new Config();
        String prefix = "redis://";
        if(redissonProperties.isSsl()){
            prefix = "rediss://";
        }
        config.useSingleServer()
                .setAddress(prefix+redissonProperties.getHost()+":"+redissonProperties.getPort())
                .setTimeout(redissonProperties.getTimeout())
                .setPassword(redissonProperties.getPassword());
        return Redisson.create(config);
    }
}
```

在META-INF.spring.factories里面指定要被加载的类

```java
org.springframework.boot.autoconfigure.EnableAutoConfiguration=\
com.nyc.boot.RedissonAutoConfiguration
```

在META-INF.additional-spring-configuration-metadata.json文件里面指定yml配置redis信息的文件格式

```java
{
  "properties": [{
    "name": "my.redisson.host",
    "type": "java.lang.String",
    "description": "Redis的服务地址",
    "defaultValue": "localhost"
  },{
    "name": "my.redisson.port",
    "type": "java.lang.Integer",
    "description": "Redis的服务端口",
    "defaultValue": 6379
  },
    {
      "name": "my.redisson.password",
      "type": "java.lang.String",
      "description": "Redis密码",
      "defaultValue": "123456"
    }
  ]
}
```



新建服务redis-spring-boot-starter-demo开始应用上面的这个组件

新建yaml配置文件

```yaml
server:
  port: 8888

my:
  redisson:
    host: xxx.xxx.xxx.xxx
    port: xxx
    ssl: false
    timeout: 1000
    password: xxxx
```

新建控制层类

```java
@RestController
public class TestController {
    @Autowired
    private RedissonClient redissonClient;


    @GetMapping("/query")
    public String test(){
        long count = redissonClient.getKeys().count();
        return "key的个数" + count;
    }
}
```

新建启动类

```java
@SpringBootApplication
public class SpringbootStarterDemoApp {
    public static void main(String[] args) {
        SpringApplication.run(SpringbootStarterDemoApp.class,args);
    }
}
```

项目启动完成以后，测试[http://localhost:8888/query](http://localhost:8888/query)这个请求即可。

# 自动装配分析

<font style="color:rgb(51, 51, 51);">Spring 一直在致力于解决一个问题，就是如何让bean的管理变得更简单，如何让开发者尽可能的少关注一些基础化的bean的配置，从而实现自动装配。所以，自动装配，实际上就是如何自动将bean装载到Ioc容器中来。在Enable模块驱动注解的出现，已经有了一定的自动装配的雏形，而真正能够实现这一机制，还是在conditional条件注解的出现。</font>

项目启动入口是带有@SpringBootApplication注解的类，这个注解组成如下

![](https://cdn.nlark.com/yuque/0/2025/png/12477403/1758681642890-0c1913d1-425c-4bb0-a146-8c9da72d0dde.png)

@SpringBootConfiguration 代表是SpringBoot项目的一个配置类

@ComponentScan 包扫描注解

@EnableAutoConfiguration 注解是由两个部分组成分别是@AutoConfigurationPackage和@Import注解。

![](https://cdn.nlark.com/yuque/0/2025/png/12477403/1758681951992-3b1f25e2-0b57-42e9-ad8b-c321a33bb934.png)

+ @AutoConfigurationPackage注解下面又套了一个@Import注解

![](https://cdn.nlark.com/yuque/0/2025/png/12477403/1758682032761-cc025a1e-e759-4e9f-b9de-ad2943e6f045.png)

+ @Import 注解导入第三方的类

既然这一块到最后的一个注解都是@Import注解，可以先理解成这一块的作用都是导入外部第三方类的。

**AutoConfigurationImportSelector** 这个类的类图如下

![](https://cdn.nlark.com/yuque/0/2025/png/12477403/1740586204346-79142536-6681-4d5f-90f8-5e185904c026.png)

生成的uml关系图可以看到AutoConfigurationImportSelector实现这个类里面的方法，而这个类又继承了ImportSelector

所以可以知道下面的这段代码就是负责自动装配的

```java
public String[] selectImports(AnnotationMetadata annotationMetadata) {
    if (!this.isEnabled(annotationMetadata)) {
        return NO_IMPORTS;
    } else {
        // 这一行代码是负责获取bean的信息，然后是放在了configurations里面
        AutoConfigurationEntry autoConfigurationEntry = this.getAutoConfigurationEntry(annotationMetadata);
        return StringUtils.toStringArray(autoConfigurationEntry.getConfigurations());
    }
}
```

![](https://cdn.nlark.com/yuque/0/2025/png/12477403/1740587707163-37cffe08-5e2b-488c-ade6-f65c61bdf761.png)

![](https://cdn.nlark.com/yuque/0/2025/png/12477403/1740587919942-bea020e2-77af-4144-8ba1-e9c42f60ddef.png)

```java
protected AutoConfigurationEntry getAutoConfigurationEntry(AnnotationMetadata annotationMetadata) {
    if (!this.isEnabled(annotationMetadata)) {
        return EMPTY_ENTRY;
    } else {
        // 获取注解的元数据 
        AnnotationAttributes attributes = this.getAttributes(annotationMetadata);
        // 加载当前系统下 META-INF/spring.factories 文件中声明的配置类
        List<String> configurations = this.getCandidateConfigurations(annotationMetadata, attributes);
        // 去重
        configurations = this.removeDuplicates(configurations);
        // 排除掉某些依赖
        Set<String> exclusions = this.getExclusions(annotationMetadata, attributes);
        this.checkExcludedClasses(configurations, exclusions);
        configurations.removeAll(exclusions);
        // 过滤掉不需要的配置
        configurations = this.getConfigurationClassFilter().filter(configurations);
        this.fireAutoConfigurationImportEvents(configurations, exclusions);
        return new AutoConfigurationEntry(configurations, exclusions);
    }
}
```

针对getAutoConfigurationEntry方法进行步骤总结

+ 首先获取注解的元数据信息
+ 加载当前系统下的META-INF/spring.factories文件中声明的配置类
+ 针对加载完的配置类进行去重操作
+ 排除掉某些不需要的依赖
+ 过滤掉不需要的配置信息
  
  

selectImports方法调用流程

![](https://cdn.nlark.com/yuque/0/2025/png/12477403/1758974529020-fe5c8bb1-c47d-4949-b980-ca9f426b12cd.png)

最终loadSpringFactories方法会从META-INF.factories文件里面根据类的全路径名称将类加载到Spring容器里面。

**Debug这个加载流程**

先不看getCandidateConfigurations这个方法里面的实现细节，先说结论这个方法就是加载spring.factories文件里面key为org.springframework.boot.autoconfigure.EnableAutoConfiguration=\下面的全路径类名。

![](https://cdn.nlark.com/yuque/0/2025/png/12477403/1758984722760-32b8939f-5d31-4aec-806f-30bf2be04103.png)

![](https://cdn.nlark.com/yuque/0/2025/png/12477403/1758984783045-f59b55df-5375-4b34-8029-4a156bba1d91.png)

按需加载当执行到filter方法的时候会把不需要进行加载的类进行过滤掉。同样是先不看这个filter方法里面的实现细节。

![](https://cdn.nlark.com/yuque/0/2025/png/12477403/1758985515471-f77ab351-af28-4e2a-8198-9a9d8c4d7a9d.png)

**getCandidateConfigurations 实现细节**

```java
private static Map<String, List<String>> loadSpringFactories(@Nullable ClassLoader classLoader) {
    // 先看缓存里是否已经加载过，（避免重复 I/O 读取），有则直接返回。cache 以 ClassLoader 为 key 做缓存
    MultiValueMap<String, String> result = cache.get(classLoader);
    if (result != null) {
        return result;
    }

    try {
        // 去找所有 META-INF/spring.factories 文件
        Enumeration<URL> urls = (classLoader != null ?
                                 classLoader.getResources(FACTORIES_RESOURCE_LOCATION) :
                                 ClassLoader.getSystemResources(FACTORIES_RESOURCE_LOCATION));
        result = new LinkedMultiValueMap<>();
        // 遍历所有找到的spring.factories文件：
        // 把URL转成UrlResource交给PropertiesLoaderUtils读取，得到一个 Properties，即 key-value 配置
        while (urls.hasMoreElements()) {
            URL url = urls.nextElement();
            UrlResource resource = new UrlResource(url);
            // 把spring.factories文件里面的key进行加载
            Properties properties = PropertiesLoaderUtils.loadProperties(resource);
            // 遍历 properties 里的配置项：
            // key = 接口/抽象类的名字，例如 org.springframework.boot.autoconfigure.EnableAutoConfiguration
            // value = 逗号分隔的实现类全限定名，例如 com.example.FooAutoConfiguration,
            for (Map.Entry<?, ?> entry : properties.entrySet()) {
                String factoryTypeName = ((String) entry.getKey()).trim();
                for (String factoryImplementationName : StringUtils.commaDelimitedListToStringArray((String) entry.getValue())) {
                    result.add(factoryTypeName, factoryImplementationName.trim());
                }
            }
        }
        // 加载完成后，放入缓存，下次直接用缓存。最后返回结果
        cache.put(classLoader, result);
        return result;
    }
    catch (IOException ex) {
        throw new IllegalArgumentException("Unable to load factories from location [" +
                                           FACTORIES_RESOURCE_LOCATION + "]", ex);
    }
}
```

遍历所有找到的Spring.factpries文件

![](https://cdn.nlark.com/yuque/0/2025/png/12477403/1758989062212-4b4a501c-c410-4e71-8293-6181ef1637be.png)

把URL转成UrlResource

![](https://cdn.nlark.com/yuque/0/2025/png/12477403/1758989176050-51682d8b-8fb2-4d46-b8e5-3bd0bd373bf3.png)

把UrlResource通过loadProperties方法加载成Properties

+ 可以看出来加载的就是spring.factories这个文件里面的key（接口/抽象类）

![](https://cdn.nlark.com/yuque/0/2025/png/12477403/1758989381731-0be6dc1d-cb19-468e-8bf7-da2a3f17f2af.png)

![](https://cdn.nlark.com/yuque/0/2025/png/12477403/1758989375194-f3a4cbc1-a6d2-47c7-9209-c31c0d034b46.png)

遍历 properties 里的配置项并加入到Map集合里面

key = 接口/抽象类的名字，例如 org.springframework.boot.autoconfigure.EnableAutoConfiguration

value = 逗号分隔的实现类全限定名，例如 com.example.FooAutoConfiguration,

![](https://cdn.nlark.com/yuque/0/2025/png/12477403/1758989592474-0cd18668-2d24-4288-bd88-7dd472181085.png)

最后放入缓存里面并返，到此自动配置加载完成



**filter方法实现细节**

在加载完成以后，springboot不可能把所有的在spring.factories里面读取到的类全部放入到容器中，因为有的类可能暂时用不到。例如，如果我没有引入数据库相关的依赖，他就不会把和datasource的类资源进行加载。

```java
private List<String> filter(List<String> configurations, AutoConfigurationMetadata autoConfigurationMetadata) {
    long startTime = System.nanoTime();
    String[] candidates = StringUtils.toStringArray(configurations);
    boolean[] skip = new boolean[candidates.length];
    boolean skipped = false;
    for (AutoConfigurationImportFilter filter : getAutoConfigurationImportFilters()) {
        invokeAwareMethods(filter);
        boolean[] match = filter.match(candidates, autoConfigurationMetadata);
        for (int i = 0; i < match.length; i++) {
            if (!match[i]) {
                skip[i] = true;
                candidates[i] = null;
                skipped = true;
            }
        }
    }
    if (!skipped) {
        return configurations;
    }
    List<String> result = new ArrayList<>(candidates.length);
    for (int i = 0; i < candidates.length; i++) {
        if (!skip[i]) {
            result.add(candidates[i]);
        }
    }
    if (logger.isTraceEnabled()) {
        int numberFiltered = configurations.size() - result.size();
        logger.trace("Filtered " + numberFiltered + " auto configuration class in "
                     + TimeUnit.NANOSECONDS.toMillis(System.nanoTime() - startTime) + " ms");
    }
    return new ArrayList<>(result);
}
```

在这个方法里面有一个参数AutoConfigurationMetadata，他是从**loadMetadata**方法里面加载而来的

```plain
AutoConfigurationMetadata autoConfigurationMetadata = AutoConfigurationMetadataLoader.loadMetadata(this.beanClassLoader);
```

![](https://cdn.nlark.com/yuque/0/2025/png/12477403/1759127037345-28c69173-fa55-4b50-89ee-6f0cbc6e88da.png)

![](https://cdn.nlark.com/yuque/0/2025/png/12477403/1759127331705-3e2564c7-7acf-4957-833e-267450f73246.png)

spring-autoconfigure-metadata.properties文件里面内容示例

```properties
org.springframework.boot.autoconfigure.web.servlet.WebMvcAutoConfiguration.ConditionalOnClass=javax.servlet.Servlet,org.springframework.web.servlet.DispatcherServlet
org.springframework.boot.autoconfigure.jdbc.DataSourceAutoConfiguration.ConditionalOnClass=javax.sql.DataSource
```

它的含义是

+ WebMvcAutoConfiguration 这个配置类只在 Servlet 和 DispatcherServlet 存在时才生效。  

+ DataSourceAutoConfiguration 只在 javax.sql.DataSource 存在时才生效。
  spring-autoconfigure-metadata.properties的作用是为这些自动配置类提供**快速条件匹配信息。**

**filter 方法的程序流程**

![](https://cdn.nlark.com/yuque/0/2025/png/12477403/1759131114133-e1b171a7-0890-46eb-a8a1-4398031df6cb.png)

到这里整个过滤方法结束。


