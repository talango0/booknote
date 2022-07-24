## Springboot 原理

Spring 原理（Spring注解）、SpringMVC原理、自动配置原理、SpringBoot原理。

### 1. Springboot启动过程

`new SpringApplication(primarySources).run(args);`

#### 1.1初始化SpringApplication

```java
public SpringApplication(ResourceLoader resourceLoader, Class<?>... primarySources) {
    this.resourceLoader = resourceLoader;
    Assert.notNull(primarySources, "PrimarySources must not be null");
    this.primarySources = new LinkedHashSet<>(Arrays.asList(primarySources));
    this.webApplicationType = WebApplicationType.deduceFromClasspath();
    this.bootstrappers = new ArrayList<>(getSpringFactoriesInstances(Bootstrapper.class));
    setInitializers((Collection) getSpringFactoriesInstances(ApplicationContextInitializer.class));
    setListeners((Collection) getSpringFactoriesInstances(ApplicationListener.class));
    this.mainApplicationClass = deduceMainApplicationClass();
}
```



#### 1.2 调用SpringApplication的run方法

