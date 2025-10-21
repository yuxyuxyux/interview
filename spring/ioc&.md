非常好 👍 你已经把 Spring IOC 的 **refresh() 全链路** 梳理得非常清晰了。
 下面我帮你把这段 **层层调用的过程** 用专业、详尽的语言再写一遍，带上**每层的职责、类层级、调用关系、设计模式背景、源码细节、关键代码段**。
 这是一个**最贴近 Spring 源码运行机制的完整 refresh() 深度分析版**。

------

# 🌱 `refresh()` 方法详细解析（核心启动链）

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/31f406d2064a1b188162c5536ef0c290.png)

`refresh()` 是 **Spring IoC 容器启动的核心方法**，定义在
 `org.springframework.context.support.AbstractApplicationContext` 中。
 它采用 **模板方法模式（Template Method Pattern）**，由子类按阶段完成容器初始化的不同步骤。

------

## 一、`refresh()` 总体结构

```java
@Override
public void refresh() throws BeansException, IllegalStateException {
    synchronized (this.startupShutdownMonitor) {
        prepareRefresh();                               // 1. 刷新前准备工作
        ConfigurableListableBeanFactory beanFactory = obtainFreshBeanFactory(); // 2. 创建 BeanFactory
        prepareBeanFactory(beanFactory);                // 3. 配置 BeanFactory
        postProcessBeanFactory(beanFactory);            // 4. 子类扩展点
        invokeBeanFactoryPostProcessors(beanFactory);   // 5. 执行 BeanFactoryPostProcessor
        registerBeanPostProcessors(beanFactory);        // 6. 注册 BeanPostProcessor
        initMessageSource();                            // 7. 国际化资源初始化
        initApplicationEventMulticaster();              // 8. 初始化事件广播器
        onRefresh();                                    // 9. 子类特定扩展（例如 Web 容器初始化）
        registerListeners();                            // 10. 注册事件监听器
        finishBeanFactoryInitialization(beanFactory);   // 11. 实例化非懒加载单例 Bean
        finishRefresh();                                // 12. 通知容器刷新完成
    }
}
```

------

## 二、创建 BeanFactory（`obtainFreshBeanFactory()`）

### 实现类：`AbstractApplicationContext`

```java
protected ConfigurableListableBeanFactory obtainFreshBeanFactory() {
    refreshBeanFactory();      // 创建或刷新 BeanFactory
    return getBeanFactory();   // 获取 BeanFactory 实例
}
```

该方法分为两步：

------

### (1) 刷新 BeanFactory：`refreshBeanFactory()`

定义在：`AbstractRefreshableApplicationContext`

```java
@Override
protected final void refreshBeanFactory() throws BeansException {
    if (hasBeanFactory()) {
        destroyBeans();        // 销毁旧 Bean
        closeBeanFactory();    // 关闭旧 BeanFactory
    }
    DefaultListableBeanFactory beanFactory = createBeanFactory(); // 新建 BeanFactory
    customizeBeanFactory(beanFactory);
    loadBeanDefinitions(beanFactory);  // 加载 Bean 定义（重点）
    this.beanFactory = beanFactory;
}
```

#### 过程说明：

1. **销毁旧容器**（如果存在）
2. **创建新的 BeanFactory**
   - 实际类型为 `DefaultListableBeanFactory`
   - 具备 `BeanDefinitionRegistry` 能力（可以注册 Bean 定义）
3. **加载 Bean 定义** — 进入本次分析的核心链条。

------

## 三、加载 Bean 定义（`loadBeanDefinitions(beanFactory)`）

定义在：`AbstractXmlApplicationContext`

```java
@Override
protected void loadBeanDefinitions(DefaultListableBeanFactory beanFactory) throws BeansException, IOException {
    XmlBeanDefinitionReader beanDefinitionReader = new XmlBeanDefinitionReader(beanFactory);
    beanDefinitionReader.setEnvironment(getEnvironment());
    beanDefinitionReader.setResourceLoader(this);
    beanDefinitionReader.setEntityResolver(new ResourceEntityResolver(this));
    loadBeanDefinitions(beanDefinitionReader);
}
```

> **核心类**：`XmlBeanDefinitionReader`
>  它是 `AbstractBeanDefinitionReader` 的子类。

------

### 1️⃣ `loadBeanDefinitions(XmlBeanDefinitionReader reader)`

该方法会尝试加载多个配置位置：

```java
protected void loadBeanDefinitions(XmlBeanDefinitionReader reader) throws IOException {
    Resource[] configResources = getConfigResources();
    if (configResources != null) {
        reader.loadBeanDefinitions(configResources);
    }
    String[] configLocations = getConfigLocations();
    if (configLocations != null) {
        reader.loadBeanDefinitions(configLocations);
    }
}
```

------

### 2️⃣ `XmlBeanDefinitionReader.loadBeanDefinitions(String... locations)`

来自 `AbstractBeanDefinitionReader`：

```java
public int loadBeanDefinitions(String... locations) throws BeanDefinitionStoreException {
    int count = 0;
    for (String location : locations) {
        count += loadBeanDefinitions(location);
    }
    return count;
}
```

继续深入：

------

### 3️⃣ `loadBeanDefinitions(String location, @Nullable Set<Resource> actualResources)`

解析资源路径，获取 `Resource[]`，再调用：

```java
loadBeanDefinitions(Resource resource);
```

------

### 4️⃣ `XmlBeanDefinitionReader.loadBeanDefinitions(Resource resource)`

最终会调用：

```java
public int loadBeanDefinitions(Resource resource) throws BeanDefinitionStoreException {
    EncodedResource encodedResource = new EncodedResource(resource);
    return loadBeanDefinitions(encodedResource);
}
```

------

### 5️⃣ `loadBeanDefinitions(EncodedResource encodedResource)`

该方法以流的形式读取配置文件，并进入核心方法：

```java
InputStream inputStream = encodedResource.getResource().getInputStream();
InputSource inputSource = new InputSource(inputStream);
return doLoadBeanDefinitions(inputSource, encodedResource.getResource());
```

------

### 6️⃣ 核心载入方法：`doLoadBeanDefinitions()`

```java
protected int doLoadBeanDefinitions(InputSource inputSource, Resource resource)
        throws BeanDefinitionStoreException {
    Document doc = doLoadDocument(inputSource, resource);     // 解析为 XML DOM 树
    return registerBeanDefinitions(doc, resource);            // 注册 BeanDefinition
}
```

------

### 7️⃣ 注册 BeanDefinition：`registerBeanDefinitions(Document doc, Resource resource)`

```java
public int registerBeanDefinitions(Document doc, Resource resource)
        throws BeanDefinitionStoreException {
    BeanDefinitionDocumentReader documentReader = createBeanDefinitionDocumentReader();
    return documentReader.registerBeanDefinitions(doc, createReaderContext(resource));
}
```

------

### 8️⃣ `DefaultBeanDefinitionDocumentReader.registerBeanDefinitions()`

```java
@Override
public void registerBeanDefinitions(Document doc, XmlReaderContext readerContext) {
    this.readerContext = readerContext;
    Element root = doc.getDocumentElement();
    doRegisterBeanDefinitions(root);
}
```

------

### 9️⃣ `doRegisterBeanDefinitions(Element root)`

```java
protected void doRegisterBeanDefinitions(Element root) {
    BeanDefinitionParserDelegate delegate = createDelegate(getReaderContext(), root, null);
    parseBeanDefinitions(root, delegate);
}
```

------

### 🔟 `parseBeanDefinitions(Element root, BeanDefinitionParserDelegate delegate)`

判断元素是默认标签（`<bean>`、`<alias>` 等）还是自定义命名空间：

```java
if (delegate.isDefaultNamespace(root)) {
    parseDefaultElement(root, delegate);
}
```

------

### (11) `parseDefaultElement(Element ele, BeanDefinitionParserDelegate delegate)`

当是 `<bean>` 标签时：

```java
if ("bean".equals(ele.getNodeName())) {
    processBeanDefinition(ele, delegate);
}
```

------

### (12) `processBeanDefinition(Element ele, BeanDefinitionParserDelegate delegate)`

```java
BeanDefinitionHolder bdHolder = delegate.parseBeanDefinitionElement(ele);
BeanDefinitionReaderUtils.registerBeanDefinition(bdHolder, getReaderContext().getRegistry());
```

------

### (13) `BeanDefinitionReaderUtils.registerBeanDefinition()`

```java
public static void registerBeanDefinition(
        BeanDefinitionHolder definitionHolder, BeanDefinitionRegistry registry)
        throws BeanDefinitionStoreException {
    String beanName = definitionHolder.getBeanName();
    registry.registerBeanDefinition(beanName, definitionHolder.getBeanDefinition());
}
```

------

### (14) `DefaultListableBeanFactory.registerBeanDefinition()`

最终注册 Bean：

```java
public void registerBeanDefinition(String beanName, BeanDefinition beanDefinition)
        throws BeanDefinitionStoreException {
    BeanDefinition existingDefinition = this.beanDefinitionMap.get(beanName);
    if (existingDefinition != null) {
        throw new BeanDefinitionStoreException("Duplicate beanName " + beanName);
    }
    this.beanDefinitionMap.put(beanName, beanDefinition);   // 注册
    this.beanDefinitionNames.add(beanName);                 // 记录名称
}
```

------

✅ **总结：**

> 所有 XML `<bean>` 标签最终都被转换成 `BeanDefinition` 对象，
>  并存放在 `DefaultListableBeanFactory` 的 `beanDefinitionMap` 中。
>  到此为止，容器**完成了配置解析与元数据注册阶段**。

------

## 四、获取 BeanFactory — `getBeanFactory()`

```java
@Override
public final ConfigurableListableBeanFactory getBeanFactory() {
    return this.beanFactory;
}
```

> 返回在 `refreshBeanFactory()` 中创建的 `DefaultListableBeanFactory`，
>  该对象已包含所有注册的 BeanDefinition，但此时 **Bean 还未被实例化**。

------

## 五、实例化 Bean — `finishBeanFactoryInitialization(beanFactory)`

定义在：`AbstractApplicationContext`

```java
protected void finishBeanFactoryInitialization(ConfigurableListableBeanFactory beanFactory) {
    // 初始化 ConversionService
    if (beanFactory.containsBean(CONVERSION_SERVICE_BEAN_NAME)) {
        beanFactory.setConversionService(beanFactory.getBean(CONVERSION_SERVICE_BEAN_NAME, ConversionService.class));
    }

    // 初始化所有非懒加载单例 Bean
    beanFactory.preInstantiateSingletons();
}
```

------

### `preInstantiateSingletons()` （DefaultListableBeanFactory）

```java
@Override
public void preInstantiateSingletons() throws BeansException {
    for (String beanName : this.beanDefinitionNames) {
        RootBeanDefinition bd = getMergedLocalBeanDefinition(beanName);
        if (bd.isSingleton() && !bd.isLazyInit()) {
            getBean(beanName);
        }
    }
}
```

此时调用链进入核心的 **Bean 创建流程：**

------

### Bean 创建主流程（`getBean` → `doGetBean`）

```java
protected <T> T doGetBean(String name, @Nullable Class<T> requiredType,
                          @Nullable Object[] args, boolean typeCheckOnly) {
    Object sharedInstance = getSingleton(beanName);
    if (sharedInstance == null) {
        RootBeanDefinition mbd = getMergedLocalBeanDefinition(beanName);
        Object beanInstance = createBean(beanName, mbd, args);
        sharedInstance = getSingleton(beanName, beanInstance);
    }
    return (T) sharedInstance;
}
```

#### Bean 的生命周期在这里开始：

1. **createBeanInstance()** → 构造方法实例化
2. **populateBean()** → 属性依赖注入
3. **initializeBean()** → 调用初始化方法、AOP、后置处理器
4. **registerDisposableBeanIfNecessary()** → 注册销毁回调

------

## 🌟 最终总结图（refresh 核心主线）

```plaintext
refresh()
 ├─ obtainFreshBeanFactory()
 │   ├─ refreshBeanFactory()
 │   │   ├─ createBeanFactory()
 │   │   ├─ loadBeanDefinitions()
 │   │   │   └─ XmlBeanDefinitionReader → doLoadBeanDefinitions()
 │   │   │       └─ DefaultListableBeanFactory.registerBeanDefinition()
 │   └─ getBeanFactory()
 └─ finishBeanFactoryInitialization(beanFactory)
     └─ beanFactory.preInstantiateSingletons()
         └─ getBean() → createBean() → populateBean() → initializeBean()
```

------

是否希望我帮你画出这一整条链的**时序图（方法调用层级 + 类归属 + 调用顺序）**？
 能直观看到从 `ApplicationContext.refresh()` 到 `BeanDefinition 注册` 再到 `Bean 实例化` 的整个运行路线图。