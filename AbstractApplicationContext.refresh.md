# 彻底搞懂Spring ApplicationContext refresh方法

`org.springframework.context.support.AbstractApplicationContext#refresh`

⚠️ 对refresh方法的内容加上了synchronized锁，refresh线程安全。

## 1. 创建记录启动器 startupStep

## 2. 为启动刷新准备上下文，包括
    2.1 设置启动时间
    2.2 激活启动状态
    2.3 初始化并校验上下文环境Environment的属性值。
    2.4 初始化 applicationListeners
    2.5 初始化 earlyApplicationEvents

## 3. 通知子类刷新内部的BeanFactory，并获取到对应的BeanFctory ，obtainFreshBeanFactory()，这里是DefaultListBeanFactory。
GenericApplicationContext 的field beanFactory = new DefaultListableBeanFactory()

DafaultListableBeanFatory 实现了org.springframework.beans.factory.config.ConfigurableListableBeanFactory， 
ConfigurableListableBeanFactory 实现了 ListableBeanFactory, AutowireCapableBeanFactory, ConfigurableBeanFactory.


## 4. 调用prepareBeanFactory（beanFactory）方法，为上下文准备BeanFactory，配置工厂标准的上下文属性，包括ClassLoader和 post-processors。
    4.1 给BeanFactory设置类加载器，ClassUtils.getDefaultClassLoader()
    4.2 指定beanFactory中值的表达式解析器。
    4.3 添加属性编辑注册器。ResourceEditorRegistrar
    4.4 配置beanFactory 上下文回调函数。并设置ignoredDependencyInterfaces，默认情况下值忽略BeanFactory接口。
    4.5 设置resolvableDependencies（从依赖类型到相关自动注入

问题: Spring在初始化的过程中，构造函数什么时候调用？

## 5. invokeBeanFactoryPostProcessors(beanFactory)

## 6. registerBeanPostProcessors(beanFactory)

## 7. initMessageSource()

## 8. initApplicationEventMulticaster()  //

## 9. onrefresh() //在子类中实例化特殊的类

## 10. registerListeners()  //

## 11. finishBeanFactoryInitialization(beanFactory) //实例化非懒加载的单例

## 12. finishRefresh() //发布相关事件

## 13. restCommonCache() //重置Spring core 中常见的introspection cache(自省缓存)

## 14. 如果发生异常，则需要清理已经创建的单例。（destroyBeans()），并调用cancelRefresh 取消刷新。

