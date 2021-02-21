# Spring 基础

[URL结构](https://dmitripavlutin.com/parse-url-javascript/)

![](https://dmitripavlutin.com/static/ee16aae605fd81c56eb2c79191e76423/1ee40/url-constructor-components-10.png)

![SpringMVC vs. SpringBoot](https://pic1.zhimg.com/v2-1bb4075e46abe6b634ca6710d6456622_r.jpg?source=1940ef5c)

![](http://tooyi.gitee.io/picbox/img/SpringBoot%E5%88%86%E5%B1%82%E7%90%86%E8%A7%A3.png)

## MVC 结构

> 一般来说，Controller是Handler，但Handler不一定是Controller。
>
> 例如：HttpRequestHandler、WebRequestHandler、MessageHandler都是可以使用DispatcherServlet的处理程序。（Controller是执行Web请求并返回视图的处理程序。）
>
> 简而言之，Handler只是一个术语，它既不是类也不是接口它负责执行映射。

## 配置文件

[加载顺序](https://docs.spring.io/spring-boot/docs/2.3.3.RELEASE/reference/html/spring-boot-features.html#boot-features-external-config)

Application: application.yml > appliction.properties > application-xxx.yml > application-xxx.properties

若 application.yml 和 bootstrap.yml 在同一目录下，bootstrap.yml 先加载 application.yml 后加载。bootstrap.yml 用于应用程序上下文的引导阶段。bootstrap.yml 由父 SpringApplicationContext 加载。bootstrap.yml 有高优先级不会被覆盖。

> In Spring Boot, YAML files can be overridden by other YAML properties files.
>
> Prior to version 2.4.0, YAML properties were overridden by properties files in the following locations, in order of highest precedence first:
>
> - Profiles' properties placed outside the packaged jar
> - Profiles' properties packaged inside the packaged jar
> - Application properties placed outside the packaged jar
> - Application properties packaged inside the packaged jar
>
> As of Spring Boot 2.4, external file always override packaged files, regardless of whether its profile-specific or not.

[bootstrap.yml configuration not processed anymore with Spring Cloud 2020.0](https://cloudstack.ninja/pavan-jadda/bootstrap-yml-configuration-not-processed-anymore-with-spring-cloud-2020-0/)

> properties in YAML files will be replaced by HashiCorp Vault properties during startup. For this, I use Spring Cloud Vault library. Everything works as expected in Spring Boot 2.3.x
>
> When I try to upgrade the project to Spring Boot 2.4.0 with Spring Cloud Vault 3.0.0-SNAPSHOT version, the properties are not being replaced

