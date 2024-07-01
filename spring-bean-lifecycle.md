# Spring Bean Lifecycle

了解 Spring Bean的生命周期的同时也是在了解 Spring 的一些拓展点, 有助于我们基于拓展点正确的进行一些中间件或者横切业务逻辑的实现.

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

# Spring Boot Lifecycle

一图流:

![](https://image.cdn.yangbingdong.com/image/spring-bean-lifecycle/0e7cd07d7953200bcb6f48648d6779b4-c94671.png)

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