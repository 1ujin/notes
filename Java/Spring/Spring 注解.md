# Spring 注解

@(Java)[Spring, 注解]

---
- [`@Bean`](#bean)
	- [`InstantiationAwareBeanPostProcessor`接口](#instantiationawarebeanpostprocessor接口)
	- [`SmartInstantiationAwareBeanPostProcessor`接口](#smartinstantiationawarebeanpostprocessor接口)
	- [`MergedBeanDefinitionPostProcessor`接口](#mergedbeandefinitionpostprocessor接口)
	- [`BeanNameAware`接口](#beannameaware接口)
	- [`BeanFactoryAware`接口](#beanfactoryaware接口)
	- [`ApplicationContextAware`接口](#applicationcontextaware接口)
	- [`BeanPostProcessor`接口](#beanpostprocessor接口)
	- [`@PostConstruct`和`@PreDestroy`](#postconstruct和predestroy)
	- [`InitializingBean`和`DisposableBean`接口](#initializingbean和disposablebean接口)
	- [`initMethod`和`destroyMethod`](#initmethod和destroymethod)

- [`@Configuration`](#configuration)

- [`@ComponentScan`和`TypeFilter`](#componentscan和typefilter)

- [`@Scope`](#scope)

- [`@Import`](#import)
	- [`ImportSelector`](#importselector)
	- [`ImportBeanDefinitionRegistrar`](#importbeandefinitionregistrar)

---
## [`@Bean`](#spring-注解)
[@Bean 注解全解析](https://www.cnblogs.com/cxuanBlog/p/11179439.html)

[Spring的@Bean只能放在@Configuration里面吗？](https://blog.csdn.net/z69183787/article/details/108105329)

[Spring注解驱动开发（一）](https://blog.csdn.net/weixin_37778801/article/details/86233124)

`Bean`类型必须有一个无参构造器。

[Spring面试题（2020最新版）](https://thinkwon.blog.csdn.net/article/details/104397516#Springbean_526)

![`Bean`的生命周期](https://img-blog.csdnimg.cn/201911012343410.png)

[Spring中bean的生命周期（最详细）](https://blog.csdn.net/qq_35634181/article/details/104473308#t3)

![`Bean`的生命周期](https://img-blog.csdnimg.cn/2020072612595369.png)

### [`InstantiationAwareBeanPostProcessor`接口](#bean)

### [`SmartInstantiationAwareBeanPostProcessor`接口](#bean)

### [`MergedBeanDefinitionPostProcessor`接口](#bean)

### [`BeanNameAware`接口](#bean)
在`Bean`类自己的无参构造器调用之后首先调用的方法。

### [`BeanFactoryAware`接口](#bean)

### [`ApplicationContextAware`接口](#bean)

### [`BeanPostProcessor`接口](#bean)

### [`@PostConstruct`和`@PreDestroy`](#bean)

### [`InitializingBean`和`DisposableBean`接口](#bean)

### [`initMethod`和`destroyMethod`](#bean)

---
## [`@Configuration`](#spring-注解)
[Spring注解驱动开发（一）](https://blog.csdn.net/weixin_37778801/article/details/86233124)

---
## [`@ComponentScan`和`TypeFilter`](#spring-注解)
[Spring注解——使用@ComponentScan自动扫描组件](https://www.jianshu.com/p/64aac6461d5b)

[Spring注解驱动开发（一）](https://blog.csdn.net/weixin_37778801/article/details/86233124)

`@ComponentScan`可以扫描到所有注解了`@Component`的类，并加载到Spring容器中。例如：`@Configuration`, `@Controller`, `@Repository`, `@Service`。


---
## [`@Scope`](#spring-注解)
[Spring注解驱动开发（一）](https://blog.csdn.net/weixin_37778801/article/details/86233124)

---
## [`@Import`](#spring-注解)
注解在类上，可以在以该类配置的容器中，引入一个名字为全类名的`Bean`，如：`com.example.BeanClass`。可以搭配`ImportSelector`和`ImportBeanDefinitionRegistrar`使用。

### [`ImportSelector`](#import)
实现`ImportSelector`接口并搭配`@Import`，返回需要的组件的全类名的数组。

### [`ImportBeanDefinitionRegistrar`](#import)
实现`ImportBeanDefinitionRegistrar`接口并搭配`@Import`，可以手动注册`Bean`到容器中。