# Spring Bean Lifecycle

Spring 的核心内容之一就是 Bean 的生命周期, 而这部分逻辑大部分内容都封装在 `refresh` 中,  外部看上去风平浪静，其实内部则是一片惊涛骇浪. Spring Boot更是封装了 Spring，遵循**约定大于配置**，加上**自动装配**的机制, 很多时候我们只要引用了一个依赖，**几乎是零配置**就能完成一个功能的装配。

了解 Spring Bean的生命周期的同时也是在了解 Spring 的一些拓展点, 加深对 Spring 的理解, 有助于我们基于拓展点使用正确姿势去实现一些中间件或者横切业务逻辑的实现.

Spring Bean的生命周期只有这四个阶段。实例化对应构造方法，属性赋值对应setter方法的注入，初始化和销毁是用户能自定义扩展的两个阶段。

**实例化 -> 属性赋值 -> 初始化 -> 销毁**

![](https://image.cdn.yangbingdong.com/image/spring-bean-lifecycle/72abf511a3d055170c0db33b134663b7-8b1ab2.png)

* **实例化** 
  * 通过反射调用构造器构造 Bean
    * 不建议多个构造器
* **属性赋值/依赖注入**, 三级缓存
  * `@Autowired`
  * setter 方法
* 初始化
  * `*Aware` 接口填充
  * `@PostConstruct`
  * `InitializingBean#afterPropertiesSet`
  * `@Bean` 中定义的 `initMethod`
* 所有 Bean 初始化后回调 `SmartInitializingSingleton#afterSingletonsInstantiated`
* 销毁
  * `@PreDestroy`
  * `DisposableBean#destroy`
  * `@Bean` 中定义的 `destroyMethod`

相关源码在 `org.springframework.beans.factory.support.AbstractAutowireCapableBeanFactory#doCreateBean`:

```
// 忽略了无关代码
protected Object doCreateBean(final String beanName, final RootBeanDefinition mbd, final @Nullable Object[] args)
      throws BeanCreationException {

   // Instantiate the bean.
   BeanWrapper instanceWrapper = null;
   if (instanceWrapper == null) {
       // 实例化阶段！
      instanceWrapper = createBeanInstance(beanName, mbd, args);
   }

   // Initialize the bean instance.
   Object exposedObject = bean;
   try {
       // 属性赋值阶段！
      populateBean(beanName, mbd, instanceWrapper);
       // 初始化阶段！
      exposedObject = initializeBean(beanName, exposedObject, mbd);
   }
}
```

而 Bean 的依赖注入, 初始化都依赖了一个重要的接口: **`BeanPostProcessor`**, 这也是一个常用的拓展点, 另外还有一个  `InstantiationAwareBeanPostProcessor`, 对应 Bean 的生命周期:

![](https://image.cdn.yangbingdong.com/image/spring-bean-lifecycle/b7dba1b3dc796855dc34458f126104b5-3d4afe.png)

另外 BeanPostProcessor 其中一部分继承关系:

![](https://image.cdn.yangbingdong.com/image/spring-bean-lifecycle/84888768dc9b165f50a1fa10d43cef8e-6adc55.png)

* `AutowiredAnnotationBeanPostProcessor`: 处理 `@Autowired`, setter 等依赖注入
* `InitDestroyAnnotationBeanPostProcessor`: 处理初始化/销毁方法
* `AsyncAnnotationBeanPostProcessor`: 处理 `@Async` 注解, 生成异步代理对象
* `ApplicationContextAwareProcessor`: 处理大部分 Aware 接口注入对应资源, 比如 `ApplicationContextAware`, `EnvironmentAware`
  * 除了 `BeanNameAware`, `BeanClassLoaderAware` 以及 `BeanFactoryAware`, 这几个 Aware 是在 `org.springframework.beans.factory.support.AbstractAutowireCapableBeanFactory#invokeAwareMethods` 处理的.
* `ConfigurationPropertiesBindingPostProcessor`: 处理 `@ConfigurationProperties` 注解, 给配置类注入对应的 Properties

> `BeanPostProcessor` 有众多实现, 它们的顺序由  `PriorityOrdered` 以及 `Ordered` 接口决定.
>
> *  PiorityOrdered是一等公民，首先被执行，PriorityOrdered公民之间通过接口返回值排序 
> *  Ordered是二等公民，然后执行，Ordered公民之间通过接口返回值排序 
> * 都没有实现是三等公民，最后执行

# 一图流

![](https://image.cdn.yangbingdong.com/image/spring-bean-lifecycle/9fff6c6b3d0f606db623618c53c0661a-27cad7.jpg)

![](https://image.cdn.yangbingdong.com/image/spring-bean-lifecycle/e4ed012bf754ebb79471c3aeb03d75b0-8d6a05.png)

![](https://image.cdn.yangbingdong.com/image/spring-bean-lifecycle/2cec2917c40b79127c0c4a8f46d3f7c0-50fb29.webp)

![](https://image.cdn.yangbingdong.com/image/spring-bean-lifecycle/0e7cd07d7953200bcb6f48648d6779b4-c94671.png)

# 详解

* `org.springframework.context.ApplicationContextInitializer`:  用于在 `ApplicationContext` 被刷新（`refresh`）之前对其进行编程初始化。这种机制允许在上下文刷新之前插入一些自定义的逻辑或配置.
  * `ApplicationContextInitializer` 的一个经典实现是 `PropertySourceBootstrapConfiguration`, 该实现通过拿到 `PropertySourceLocator` 的实现向 `ConfigurableEnvironment` 注入外部配置, 比如 Nacos 配置中心就是这么实现的, 参考 `NacosPropertySourceLocator`.
* ` org.springframework.beans.factory.config.BeanFactoryPostProcessor`:  是 Spring 框架中的一个重要接口，它允许我们在 Spring 容器的标准初始化过程之后、实际的 bean 实例化之前，对 bean 的定义（`BeanDefinition`）进行修改。这种机制在 Spring 框架中用于各种配置和自定义初始化逻辑。常见的 BeanFactoryPostProcessor 实现如下
  * ` PropertySourcesPlaceholderConfigurer `: 用于处理 `@Value` 注解中的占位符比如 `${spring.application.name}`。它从外部资源文件（如 properties 文件）中读取值，并将这些值注入到 Spring bean 中。
  * `ConfigurationClassPostProcessor` : 它主要负责解析使用 `@Configuration` 注解的配置类，以及通过 `@ComponentScan` 和 `@Import` 等注解注册的类, 然后注册到 Spirng 容器中.



# Spring Boot Starter 的推荐实现方式

* 声明配置类, 并在类上贴 `@AutoConfiguration` (2.7.0 以前的版本使用 `@Configuration`)注解, Bean 在配置类中声明
* 在 `spring.factories` 或者 `org.springframework.boot.autoconfigure.AutoConfiguration.imports` (Spring Boot 3.0 之后的方式) 加上配置类

Example:

```
@AutoConfiguration
@EnableConfigurationProperties(RabbitProperties.class)
public class MyAutoConfiguration {

    @Bean
    @ConditionalOnMissingBean
    public MyAbility myAbility() {
        return new MyAbility();
    }
}

public class MyAbility {

    public String sayHello() {
        return "Hello World!";
    }
}

```

` META-INF/spring.factories `:

```
org.springframework.boot.autoconfigure.EnableAutoConfiguration=\
com.example.autoconfig.MyAutoConfiguration
```

如果是 Spring Boot 3.0+, 则需要声明在 `META-INF/spring/org.springframework.boot.autoconfigure.AutoConfiguration.imports`

```
com.example.autoconfig.MyAutoConfiguration
```

**注意点**

对于配置 Properties 类, 尽量使用 `@ConfigurationProperties` + `@EnableConfigurationProperties` 使用.