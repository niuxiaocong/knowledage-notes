SpringBoot配置文件的加载是依靠监听机制来实现的，是通过ConfigFileApplicationListener这个监听器来实现的。

# 加载属性文件的一个流程图
![](https://cdn.nlark.com/yuque/0/2025/png/12477403/1756957104520-87e468c8-56b5-484a-936a-e542ae514319.png)

# 源码分析
在SpringApplication(primarySources)这个构造方法里面加载Springboot内置的监听器，包含我们的配置文件处理器监听器ConfigFileApplicationListener，如果是Cloud项目还会加载BootstrapApplicationListener。

在run通过getRunListeners(args)方法加载事件发布器EventPublishingRunListener，然后发布starting事件，在prepareEnvironment(listeners, applicationArguments)环境准备阶段，去发布environmentPrepared事件，在这个事件发布的时候会触发ConfigFileApplicationListener这个监听器。在ConfigFileApplicationListener这个监听器里面就负责加载 .properties、.xml、.yml、.yaml 配置文件

## 环境准备prepareEnvironment
```java
private ConfigurableEnvironment prepareEnvironment(SpringApplicationRunListeners listeners,
                                                   ApplicationArguments applicationArguments) {
    // Create and configure the environment
    // 创建并配置Environment
    ConfigurableEnvironment environment = getOrCreateEnvironment();
    // 配置PropertySources和activeProfiles
    configureEnvironment(environment, applicationArguments.getSourceArgs());
    ConfigurationPropertySources.attach(environment);
    // 在配置环境信息之前发布事件
    listeners.environmentPrepared(environment);
    // 把相关的配置信息绑定到Spring容器中
    bindToSpringApplication(environment);
    if (!this.isCustomEnvironment) {
        environment = new EnvironmentConverter(getClassLoader()).convertEnvironmentIfNecessary(environment,
                                                                                               deduceEnvironmentClass());
    }
    // 配置PropertySources对它自己的递归依赖
    ConfigurationPropertySources.attach(environment);
    return environment;
}
```

发布事件environmentPrepared以后触发**ConfigFileApplicationListener**里面的onApplicationEvent(ApplicationEvent event)方法

```java
public void onApplicationEvent(ApplicationEvent event) {
    //  如果是ApplicationEnvironmentPreparedEvent 则处理
    if (event instanceof ApplicationEnvironmentPreparedEvent) {
        onApplicationEnvironmentPreparedEvent((ApplicationEnvironmentPreparedEvent) event);
    }
    // 如果是 ApplicationPreparedEvent 则处理
    if (event instanceof ApplicationPreparedEvent) {
        onApplicationPreparedEvent(event);
    }
}
```

## 配置文件内容加载onApplicationEnvironmentPreparedEvent


```java
private void onApplicationEnvironmentPreparedEvent(ApplicationEnvironmentPreparedEvent event) {
    // 加载系统提供的环境配置的后置处理器
    List<EnvironmentPostProcessor> postProcessors = loadPostProcessors();
    // 添加自身即 ConfigFileApplicationListener 为后置处理器
    postProcessors.add(this);
    // 原来有4个，现在加了一个需要重新排序
    AnnotationAwareOrderComparator.sort(postProcessors);
    for (EnvironmentPostProcessor postProcessor : postProcessors) {
        // 系统提供的4个不是重点，重点是ConfigFileApplicationListener 中的这个方法处理
        postProcessor.postProcessEnvironment(event.getEnvironment(), event.getSpringApplication());
    }
}
```

后置处理方法

```java
@Override
public void postProcessEnvironment(ConfigurableEnvironment environment, SpringApplication application) {
    // 属性文件的加载解析在这个方法里面
    addPropertySources(environment, application.getResourceLoader());
}
```

![](https://cdn.nlark.com/yuque/0/2025/png/12477403/1756970938495-8d03ebb6-c440-4fe6-bb12-c89a957446a2.png)

addPropertySources 将配置文件属性源添加到指定环境

```java
protected void addPropertySources(ConfigurableEnvironment environment, ResourceLoader resourceLoader) {
    RandomValuePropertySource.addToEnvironment(environment);
    new Loader(environment, resourceLoader).load();
}
```

## Loader 加载候选属性源并配置活动配置文件
### Loader构造方法
```java
Loader(ConfigurableEnvironment environment, ResourceLoader resourceLoader) {
    // environment 对象的赋值
    this.environment = environment;
    // 占位符处理
    this.placeholdersResolver = new PropertySourcesPlaceholdersResolver(this.environment);
    // 资源加载器
    this.resourceLoader = (resourceLoader != null) ? resourceLoader : new DefaultResourceLoader();
    // 重点：获取spring.factories 中配置的属性加载器，用来加载解析properties文件或者是yml文件
    this.propertySourceLoaders = SpringFactoriesLoader.loadFactories(PropertySourceLoader.class,
            getClass().getClassLoader());
}
```

![](https://cdn.nlark.com/yuque/0/2025/png/12477403/1756972290047-8a93712b-1343-4c90-ac61-c7711adf440c.png)

![](https://cdn.nlark.com/yuque/0/2025/png/12477403/1756972322363-8382c16f-af5c-4286-995b-0ea4c582e4db.png)

#### PropertiesPropertySourceLoader 类
负责处理.properties、.xml配置文件

```java
public class PropertiesPropertySourceLoader implements PropertySourceLoader {
    private static final String XML_FILE_EXTENSION = ".xml";
    @Override
    public String[] getFileExtensions() {
        return new String[] { "properties", "xml" };
    }
    @Override
    public List<PropertySource<?>> load(String name, Resource resource) throws IOException {
        // 获取属性文件中的信息
        Map<String, ?> properties = loadProperties(resource);
        if (properties.isEmpty()) {
            return Collections.emptyList();
        }
        return Collections
        .singletonList(new OriginTrackedMapPropertySource(name, Collections.unmodifiableMap(properties), true));
    }
    @SuppressWarnings({ "unchecked", "rawtypes" })
    private Map<String, ?> loadProperties(Resource resource) throws IOException {
        String filename = resource.getFilename();
        if (filename != null && filename.endsWith(XML_FILE_EXTENSION)) {
            // 处理application.xml
            return (Map) PropertiesLoaderUtils.loadProperties(resource);
        }
        // 处理application.properties
        return new OriginTrackedPropertiesLoader(resource).load();
    }
}
```

#### YamlPropertySourceLoader 类
负责处理yml、yaml配置文件

```java
public class YamlPropertySourceLoader implements PropertySourceLoader {

    @Override
    public String[] getFileExtensions() {
        return new String[] { "yml", "yaml" };
    }

    @Override
    public List<PropertySource<?>> load(String name, Resource resource) throws IOException {
        if (!ClassUtils.isPresent("org.yaml.snakeyaml.Yaml", null)) {
            throw new IllegalStateException(
                "Attempted to load " + name + " but snakeyaml was not found on the classpath");
        }
        List<Map<String, Object>> loaded = new OriginTrackedYamlLoader(resource).load();
        if (loaded.isEmpty()) {
            return Collections.emptyList();
        }
        List<PropertySource<?>> propertySources = new ArrayList<>(loaded.size());
        for (int i = 0; i < loaded.size(); i++) {
            String documentNumber = (loaded.size() != 1) ? " (document #" + i + ")" : "";
            propertySources.add(new OriginTrackedMapPropertySource(name + documentNumber,
                                                                   Collections.unmodifiableMap(loaded.get(i)), true));
        }
        return propertySources;
    }

}
```

### void load()方法
```java
void load() {
    FilteredPropertySource.apply(this.environment, DEFAULT_PROPERTIES, LOAD_FILTERED_PROPERTY,
            (defaultProperties) -> {
                // 创建默认的profile链表
                this.profiles = new LinkedList<>();
                // 创建已经处理过的profile类别
                this.processedProfiles = new LinkedList<>();
                // 默认设置为未激活
                this.activatedProfiles = false;
                // 创建load对象
                this.loaded = new LinkedHashMap<>();
                // 加载配置profile信息，默认未default
                initializeProfiles();
                // 遍历Profiles，并加载解析
                while (!this.profiles.isEmpty()) {
                    // 从双向链表中获取一个profile对象
                    Profile profile = this.profiles.poll();
                    // 非默认的就加入，进去看源码即可
                    if (isDefaultProfile(profile)) {
                        addProfileToEnvironment(profile.getName());
                    }
                    load(profile, this::getPositiveProfileFilter,
                            addToLoaded(MutablePropertySources::addLast, false));
                    this.processedProfiles.add(profile);
                }
                // 解析profile
                load(null, this::getNegativeProfileFilter, addToLoaded(MutablePropertySources::addFirst, true));
                // 加载默认的属性文件application.properties
                addLoadedPropertySources();
                applyActiveProfiles(defaultProperties);
            });
}
```

![](https://cdn.nlark.com/yuque/0/2025/png/12477403/1756974651763-c104ab60-b02f-4fb3-9bf0-917059454fb3.png)

```java
private void load(Profile profile, DocumentFilterFactory filterFactory, DocumentConsumer consumer) {
    // 获得默认的扫描路径，如果没有特殊指定
    // 就采用DEFAULT_SEARCH_LOCATIONS中定义的4个路径
    // 而getSearchNames方法获得的就是application这个默认的配置文件名
    // 然后逐一遍历加载目录路径及其指定文件名的文件
    // file:./config/file:./classpath:/config/ classpath:/ 默认四个路径
    getSearchLocations().forEach((location) -> {
        boolean isFolder = location.endsWith("/");
        // 去对应的路径下获取属性文件，默认的文件名称是application
        Set<String> names = isFolder ? getSearchNames() : NO_SEARCH_NAMES;
        names.forEach((name) -> load(location, name, profile, filterFactory, consumer));
    });
}
```

```java
private void load(String location, String name, Profile profile, DocumentFilterFactory filterFactory,
        DocumentConsumer consumer) {
    if (!StringUtils.hasText(name)) {
        for (PropertySourceLoader loader : this.propertySourceLoaders) {
            if (canLoadFileExtension(loader, location)) {
                // 去对应的文件夹下加载属性文件，application.properties application.yml
                load(loader, location, profile, filterFactory.getDocumentFilter(profile), consumer);
                return;
            }
        }
        throw new IllegalStateException("File extension of config file location '" + location
                + "' is not known to any PropertySourceLoader. If the location is meant to reference "
                + "a directory, it must end in '/'");
    }
    Set<String> processed = new HashSet<>();
    // 获取properties和yml的资源加载器
    for (PropertySourceLoader loader : this.propertySourceLoaders) {
        // 获取对于应的加载器的后缀 properties、xml、yml、yaml
        for (String fileExtension : loader.getFileExtensions()) {
            if (processed.add(fileExtension)) {
                // 加载文件 application.properties application.yml
                // application.yml application.yaml
                loadForFileExtension(loader, location + name, "." + fileExtension, profile, filterFactory,
                        consumer);
            }
        }
    }
}
```

![](https://cdn.nlark.com/yuque/0/2025/png/12477403/1756974737183-8ce7e4a3-cf53-4615-983a-f7c84ccc4742.png)

```java
private void loadForFileExtension(PropertySourceLoader loader, String prefix, String fileExtension,
        Profile profile, DocumentFilterFactory filterFactory, DocumentConsumer consumer) {
    DocumentFilter defaultFilter = filterFactory.getDocumentFilter(null);
    DocumentFilter profileFilter = filterFactory.getDocumentFilter(profile);
    if (profile != null) {
        // 如果有profile的情况比如dev --> application-dev.properties
        // Try profile-specific file & profile section in profile file (gh-340)
        String profileSpecificFile = prefix + "-" + profile + fileExtension;
        load(loader, profileSpecificFile, profile, defaultFilter, consumer);
        load(loader, profileSpecificFile, profile, profileFilter, consumer);
        // Try profile specific sections in files we've already processed
        for (Profile processedProfile : this.processedProfiles) {
            if (processedProfile != null) {
                String previouslyLoaded = prefix + "-" + processedProfile + fileExtension;
                load(loader, previouslyLoaded, profile, profileFilter, consumer);
            }
        }
    }
    // 加载正常的情况的属性文件 application.properties
    // Also try the profile-specific section (if any) of the normal file
    load(loader, prefix + fileExtension, profile, profileFilter, consumer);
}
```

![](https://cdn.nlark.com/yuque/0/2025/png/12477403/1756974788270-eec8d537-b995-4e60-b14b-6c8cabd254e6.png)

```java
private void load(PropertySourceLoader loader, String location, Profile profile, DocumentFilter filter,
        DocumentConsumer consumer) {
    try {
        Resource resource = this.resourceLoader.getResource(location);
        if (resource == null || !resource.exists()) {
            if (this.logger.isTraceEnabled()) {
                StringBuilder description = getDescription("Skipped missing config ", location, resource,
                        profile);
                this.logger.trace(description);
            }
            return;
        }
        if (!StringUtils.hasText(StringUtils.getFilenameExtension(resource.getFilename()))) {
            if (this.logger.isTraceEnabled()) {
                StringBuilder description = getDescription("Skipped empty config extension ", location,
                        resource, profile);
                this.logger.trace(description);
            }
            return;
        }
        String name = "applicationConfig: [" + location + "]";
        // 加载属性文件信息
        List<Document> documents = loadDocuments(loader, name, resource);
        if (CollectionUtils.isEmpty(documents)) {
            if (this.logger.isTraceEnabled()) {
                StringBuilder description = getDescription("Skipped unloaded config ", location, resource,
                        profile);
                this.logger.trace(description);
            }
            return;
        }
        List<Document> loaded = new ArrayList<>();
        for (Document document : documents) {
            if (filter.match(document)) {
                addActiveProfiles(document.getActiveProfiles());
                addIncludedProfiles(document.getIncludeProfiles());
                loaded.add(document);
            }
        }
        Collections.reverse(loaded);
        if (!loaded.isEmpty()) {
            loaded.forEach((document) -> consumer.accept(profile, document));
            if (this.logger.isDebugEnabled()) {
                StringBuilder description = getDescription("Loaded config file ", location, resource, profile);
                this.logger.debug(description);
            }
        }
    }
    catch (Exception ex) {
        throw new IllegalStateException("Failed to load property source from location '" + location + "'", ex);
    }
}
```

![](https://cdn.nlark.com/yuque/0/2025/png/12477403/1756974839570-c471edb9-c1e7-4367-80bc-ee498e202f2d.png)

```java
private List<Document> loadDocuments(PropertySourceLoader loader, String name, Resource resource)
        throws IOException {
    // 文档缓存的主键对象
    DocumentsCacheKey cacheKey = new DocumentsCacheKey(loader, resource);
    // 获取缓存数据
    List<Document> documents = this.loadDocumentsCache.get(cacheKey);
    if (documents == null) {
        // 加载属性文件信息
        List<PropertySource<?>> loaded = loader.load(name, resource);
        documents = asDocuments(loaded);
        this.loadDocumentsCache.put(cacheKey, documents);
    }
    return documents;
}
```

![](https://cdn.nlark.com/yuque/0/2025/png/12477403/1756974889013-dda3e9e5-7ef8-4225-aa53-eb9015dd2493.png)

到此位置loader.load(name, resource);这个方法就会触发到PropertiesPropertySourceLoader或者YamlPropertySourceLoader的load方法，去加载配置文件的内容。

![](https://cdn.nlark.com/yuque/0/2025/png/12477403/1756973813266-2a101e83-990a-4379-b9c1-7cd48c2220d0.png)





