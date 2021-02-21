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
## @Bean
[@Bean 注解全解析](https://www.cnblogs.com/cxuanBlog/p/11179439.html)

[Spring的@Bean只能放在@Configuration里面吗？](https://blog.csdn.net/z69183787/article/details/108105329)

[Spring注解驱动开发（一）](https://blog.csdn.net/weixin_37778801/article/details/86233124)

被`@Bean`注解的**方法**相当于XML配置文件中的`<bean>`标记。Bean类型必须有一个**无参构造器**。

[Spring面试题（2020最新版）](https://thinkwon.blog.csdn.net/article/details/104397516#Springbean_526)

![`Bean`的生命周期](https://img-blog.csdnimg.cn/201911012343410.png)

[Spring中bean的生命周期（最详细）](https://blog.csdn.net/qq_35634181/article/details/104473308#t3)

![`Bean`的生命周期](https://img-blog.csdnimg.cn/2020072612595369.png)

### [Full 模式和 Lite 模式](https://my.oschina.net/u/4412457/blog/4296258)

官方定义为：在没有标注`@Configuration`的类里面有`@Bean`方法就称为Lite模式的配置。透过源码再看这个定义是不完全正确的，而应该是有如下情况均认为是Lite模式的配置类：

- 类上标注有`@Component`注解
- 类上标注有`@ComponentScan`注解
- 类上标注有`@Import`注解
- 类上标注有`@ImportResource`注解
- 若类上没有任何注解，但类内存在`@Bean`方法

这种模式产生的Bean不为单例模式，没有经过CGLIB动态代理，也就无法对其进行AOP增强，所以调用构造方法时，为普通Java调用方式。容器上下文中只会保留第一个生成的实例，作为容器管理的Bean。

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

### Bean的四种构造方式

#### constructor-arg

#### property

#### 注入集合

##### list

##### set

##### map

#### props

#### 注入内部 Bean

---
## @Component

泛指组件，当组件不好归类的时候，我们可以使用这个注解进行标注。

---
## @ComponentScan 和 TypeFilter
[Spring注解——使用@ComponentScan自动扫描组件](https://www.jianshu.com/p/64aac6461d5b)

[Spring注解驱动开发（一）](https://blog.csdn.net/weixin_37778801/article/details/86233124)

`@ComponentScan`可以扫描到所有注解了`@Component`的类，并加载到Spring容器中。例如：`@Configuration`, `@Controller`, `@Repository`, `@Service`。其自身也必须放在可被扫描到的类上，一般为配置类。


---
## @Scope
[Spring注解驱动开发（一）](https://blog.csdn.net/weixin_37778801/article/details/86233124)

---
## @Import
注解在类上，可以在以该类配置的容器中，引入一个名字为全类名的`Bean`，如：`com.example.BeanClass`。可以搭配`ImportSelector`和`ImportBeanDefinitionRegistrar`使用。

### ImportSelector
实现`ImportSelector`接口并搭配`@Import`，返回需要的组件的全类名的数组。

### ImportBeanDefinitionRegistrar
实现`ImportBeanDefinitionRegistrar`接口并搭配`@Import`，可以手动注册`Bean`到容器中。

## 读取配置的方式

### @Value

将全局配置文件application[-profile].properties|yml中的值赋给被注解的成员变量上。

### @PropertySource

读取配置文件.properties或.yml并其中对应的字段注入给被注解的类的成员变量上。

### @ImportSource

读取配置文件<beans-config>.xml，并将配置的Bean全部交给容器管理。必须注解在配置类上。

---

## @Controller

[springmvc Controller接收前端参数的几种方式总结](https://www.cnblogs.com/mjs154/p/11667796.html)

[controller 的三种返回方式](https://www.cnblogs.com/gagazhao/p/6735803.html)

> 一般来说，Controller是Handler，但Handler不一定是Controller。
>
> 例如：HttpRequestHandler、WebRequestHandler、MessageHandler都是可以使用DispatcherServlet的处理程序。（Controller是执行Web请求并返回视图的处理程序。）
>
> 简而言之，Handler只是一个术语，它既不是类也不是接口它负责执行映射。

---

## @Repository

使用`@Repository`注解可以确保DAO或者Repositories提供异常转译，这个注解修饰的DAO或者Repositories类会被`@ComponetScan`发现并配置，同时也不需要为它们提供XML配置项。

---

## @Service

---

## @Configuration

相当于XML配置文件中的<beans>

[Spring注解驱动开发（一）](https://blog.csdn.net/weixin_37778801/article/details/86233124)

[@Component和@Configuration作为配置类的差别](https://blog.csdn.net/long476964/article/details/80626930)

与`@Component`的区别是，`@Configuration`注解的类是**单例**，而`@Component`注解的类是**多例**。

## @Autowired

> Field injection is not recommended Inspection info: Spring Team recommends: "Always use constructor based dependency injection in your beans. Always use assertions for mandatory dependencies". 
>
> 这段是Spring工作组的建议，大致翻译一下： 属性字段注入的方式不推荐，检查到的问题是：Spring团队建议:"始终在bean中使用基于构造函数的依赖项注入， 始终对强制性依赖项使用断言"