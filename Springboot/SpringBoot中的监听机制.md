# SpringBoot中内置的监听器

内置监听器都定义在在spring.factories文件中

![](https://cdn.nlark.com/yuque/0/2025/png/12477403/1756741352246-9091c4b6-00e7-4aa4-a521-bca928853204.png)

| 监听器                                        | 监听事件                                                                                                                                      | 说明                                                                                                                                                              |
| ------------------------------------------ | ----------------------------------------------------------------------------------------------------------------------------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| ClearCachesApplicationListener             | ContextRefreshedEvent                                                                                                                     | 当触发ContextRefreshedEvent事件会清空应用的缓存                                                                                                                              |
| ParentContextCloserApplicationListener     | ParentContextAvailableEvent                                                                                                               | 触发ParentContextAvailableEvent事件会完成父容器关闭的监听器                                                                                                                     |
| CloudFoundryVcapEnvironmentPostProcessor   | ApplicationPreparedEvent                                                                                                                  | 判断环境中是否存在VCAP_APPLICATION或者VCAP_SERVICES。如果有就添加Cloud Foundry的配置；没有就不执行任何操作。                                                                                     |
| FileEncodingApplicationListener            | ApplicationEnvironmentPreparedEvent                                                                                                       | 文件编码的监听器                                                                                                                                                        |
| AnsiOutputApplicationListener              | ApplicationEnvironmentPreparedEvent                                                                                                       | 根据 `spring.output.ansi.enabled`参数配置 `AnsiOutput`                                                                                                                |
| ConfigFileApplicationListener              | ApplicationEnvironmentPreparedEvent<br/>ApplicationPreparedEvent                                                                          | 完成相关属性文件的加载，application.properties、 application.yml                                                                                                             |
| DelegatingApplicationListener              | ApplicationEnvironmentPreparedEvent                                                                                                       | 监听到事件后转发给环境变量 context.listener.classes指定的那些事件监听器                                                                                                                |
| ClasspathLoggingApplicationListener        | ApplicationEnvironmentPrepared<br/>EventApplicationFailedEvent                                                                            | 一个SmartApplicationListener,对环境就绪事件ApplicationEnvironmentPreparedEvent/应用失败事件ApplicationFailedEvent做出响应，往日志DEBUG级别输出TCCL(thread context class loader)的classpath。 |
| LoggingApplicationListener                 | ApplicationStartingEvent ApplicationEnvironmentPreparedEvent ApplicationPreparedEvent <br/>ContextClosedEvent <br/>ApplicationFailedEvent | 配置 `LoggingSystem`。使用 `logging.config`环境变量指定的配置或者缺省配置                                                                                                           |
| LiquibaseServiceLocatorApplicationListener | ApplicationStartingEvent                                                                                                                  | 使用一个可以和Spring Boot可执行jar包配合工作的版本替换liquibase ServiceLocator                                                                                                      |
| BackgroundPreinitializer                   | ApplicationStartingEvent<br/>ApplicationReadyEvent <br/>ApplicationFailedEvent                                                            | 尽早触发一些耗时的初始化任务，使用一个后台线程                                                                                                                                         |



# 自定义监听器和监听事件

自定义监听事件

需要继承ApplicationEvent

```java
public class MyEvent extends ApplicationEvent {
    /**
     * Create a new {@code ApplicationEvent}.
     *
     * @param source the object on which the event initially occurred or with
     *               which the event is associated (never {@code null})
     */
    public MyEvent(Object source) {
        super(source);
    }
}
```

![](https://cdn.nlark.com/yuque/0/2025/png/12477403/1756742107215-44053456-72a5-4ee5-9e64-ed505ee39fce.png)

自定义监听器

需要实现ApplicationListener接口

```java
/**
 * @author 26917
 * 自定义监听器
 *  监听自定义的事件
 */
public class MyCustomerEventListener implements ApplicationListener<MyEvent> {
    @Override
    public void onApplicationEvent(MyEvent event) {
        System.out.println("MyCustomerEventListener ----》 自定义事件触发" + event);
    }
}
```

ApplicationListener类图

![](https://cdn.nlark.com/yuque/0/2025/png/12477403/1756742061139-08f78319-e1f5-481d-9579-d4c7579e0538.png)

将这个自定义的监听器加入到spring.factories文件中

```java
org.springframework.context.ApplicationListener=\
com.nyc.listener.MyCustomerEventListener
```

使用这个监听器

```java
@RestController
public class UserController {
    @Autowired
    private ApplicationContext context;
    @GetMapping("/hello")
    public String hello(){
        context.publishEvent(new MyEvent(new Object()));
        return "hello";
    }
}
```

# 监听器代码分析

## 类图分析

![](https://cdn.nlark.com/yuque/0/2025/png/12477403/1756954613501-0263dd9b-fd9d-424a-a960-44889d39a328.png)

## getRunListeners(args)方法

![](https://cdn.nlark.com/yuque/0/2025/png/12477403/1756796599890-6d90168f-214e-479a-873a-1353e84b16c4.png)

这个方法获取到事件发布器

![](https://cdn.nlark.com/yuque/0/2025/png/12477403/1756796705826-e2f6ef56-1e00-4ca4-be16-56cd80cd587b.png)

EventPublishingRunListener 构造方法

```java
public EventPublishingRunListener(SpringApplication application, String[] args) {
    this.application = application;
    this.args = args;
    // 初始化多个监听器，其实就是我们前面加载的spring.factories文件中的11个监听器
    this.initialMulticaster = new SimpleApplicationEventMulticaster();
    // application.getListeners() 获取11个监听器
    for (ApplicationListener<?> listener : application.getListeners()) {
        // 绑定初始的11个监听器
        this.initialMulticaster.addApplicationListener(listener);
    }
}
```

## listeners.starting()方法

触发启动事件 -》发布starting事件，那么监听starting事件的监听器就会触发

![](https://cdn.nlark.com/yuque/0/2025/png/12477403/1756798264521-721f42e9-3d27-421f-a649-ccbcf374224e.png)

```java
class SpringApplicationRunListeners {

    private final Log log;

    private final List<SpringApplicationRunListener> listeners;

    SpringApplicationRunListeners(Log log, Collection<? extends SpringApplicationRunListener> listeners) {
        this.log = log;
        this.listeners = new ArrayList<>(listeners);
    }

    void starting() {
        // 发布器 EventPublishingRunListener
        for (SpringApplicationRunListener listener : this.listeners) {
            listener.starting();
        }
    }

    void environmentPrepared(ConfigurableEnvironment environment) {
        for (SpringApplicationRunListener listener : this.listeners) {
            listener.environmentPrepared(environment);
        }
    }

    void contextPrepared(ConfigurableApplicationContext context) {
        for (SpringApplicationRunListener listener : this.listeners) {
            listener.contextPrepared(context);
        }
    }

    void contextLoaded(ConfigurableApplicationContext context) {
        for (SpringApplicationRunListener listener : this.listeners) {
            listener.contextLoaded(context);
        }
    }

    void started(ConfigurableApplicationContext context) {
        for (SpringApplicationRunListener listener : this.listeners) {
            listener.started(context);
        }
    }

    void running(ConfigurableApplicationContext context) {
        for (SpringApplicationRunListener listener : this.listeners) {
            listener.running(context);
        }
    }

    void failed(ConfigurableApplicationContext context, Throwable exception) {
        for (SpringApplicationRunListener listener : this.listeners) {
            callFailedListener(listener, context, exception);
        }
    }

    private void callFailedListener(SpringApplicationRunListener listener, ConfigurableApplicationContext context,
            Throwable exception) {
        try {
            listener.failed(context, exception);
        }
        catch (Throwable ex) {
            if (exception == null) {
                ReflectionUtils.rethrowRuntimeException(ex);
            }
            if (this.log.isDebugEnabled()) {
                this.log.error("Error handling failed", ex);
            }
            else {
                String message = ex.getMessage();
                message = (message != null) ? message : "no error message";
                this.log.warn("Error handling failed (" + message + ")");
            }
        }
    }

}
```

SpringApplicationRunListeners类的作用

`SpringApplicationRunListener` 的集合代理类，用来统一调度并触发所有 RunListener，整体监视 Spring Boot 启动过程的各个阶段。

在源码中SpringApplicationRunListener接口的默认实现就只有，EventPublishingRunListener

![](https://cdn.nlark.com/yuque/0/2025/png/12477403/1756798764298-369ded0c-029f-4afb-8521-cdd56a7bb964.png)

EventPublishingRunListener类的starting方法。

```java
@Override
public void starting() {
    this.initialMulticaster.multicastEvent(new ApplicationStartingEvent(this.application, this.args));
}
```

在springboot的启动的时候会在以下几个阶段进行监听器发布，分别是启动阶段、环境配置准备阶段、容器上下文准备阶段、启动完成阶段、启动失败阶段以及运行阶段。

![](https://cdn.nlark.com/yuque/0/2025/png/12477403/1756800155769-33d2da00-e869-4f8c-b498-bef53aa207fb.png)




























