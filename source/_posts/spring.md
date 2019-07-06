---
title: spring全家桶
date: 2019-07-06 11:40:13
tags: 
    - spring
---

![](https://bucketyy.oss-cn-beijing.aliyuncs.com/image/spring/spring1.jpg)
### spring框架集合

spring系列包含非常多的项目，可以满足java开发中的方方面面。先看spring框架集合：
![](https://bucketyy.oss-cn-beijing.aliyuncs.com/image/spring/spring2.png)
### 五个常用的spring框架
####  1.spring framework

也就是我们经常说的spring框架，包括了ioc依赖注入，Context上下文、bean管理、springmvc等众多功能模块，其它spring项目比如spring boot也会依赖spring框架。

#### 2.spring boot

它的目标是简化Spring应用和服务的创建、开发与部署，简化了配置文件，使用嵌入式web服务器，含有诸多开箱即用的微服务功能，可以和spring cloud联合部署。

Spring Boot的核心思想是约定大于配置，应用只需要很少的配置即可，简化了应用开发模式。

#### 3.Spring Data

是一个数据访问及操作的工具集，封装了多种数据源的操作能力，包括：jdbc、Redis、MongoDB等。

#### 4.Spring Cloud

是一套完整的微服务解决方案，是一系列不同功能的微服务框架的集合。Spring Cloud基于Spring Boot，简化了分布式系统的开发，集成了服务发现、配置管理、消息总线、负载均衡、断路器、数据监控等各种服务治理能力。比如sleuth提供了全链路追踪能力，Netflix套件提供了hystrix熔断器、zuul网关等众多的治理组件。config组件提供了动态配置能力，bus组件支持使用RabbitMQ、kafka、Activemq等消息队列，实现分布式服务之间的事件通信。

#### 5. Spring Security

主要用于快速构建安全的应用程序和服务，在Spring Boot和Spring Security OAuth2的基础上，可以快速实现常见安全模型，如单点登录，令牌中继和令牌交换。你可以了解一下oauth2授权机制和jwt认证方式。oauth2是一种授权机制，规定了完备的授权、认证流程。JWT全称是JSON Web Token，是一种把认证信息包含在token中的认证实现，oauth2授权机制中就可以应用jwt来作为认证的具体实现方法。
### spring 基本概念
![](https://bucketyy.oss-cn-beijing.aliyuncs.com/image/spring/spring3.jpg)
#### 1.IOC

IOC，就是控制反转，如最左边，拿公司招聘岗位来举例：

假设一个公司有产品、研发、测试等岗位。如果是公司根据岗位要求，逐个安排人选，如图中向下的箭头，这是正向流程。如果反过来，不用公司来安排候选人，而是由第三方猎头来匹配岗位和候选人，然后进行推荐，如图中向上的箭头，这就是控制反转。

在spring中，对象的属性是由对象自己创建的，就是正向流程；如果属性不是对象创建，而是由spring来自动进行装配，就是控制反转。这里的DI也就是依赖注入，就是实现控制反转的方式。正向流程导致了对象于对象之间的高耦合，IOC可以解决对象耦合的问题，有利于功能的复用，能够使程序的结构变得非常灵活。

#### 2.Context上下文和Bean

spring进行IOC实现时使用的有两个概念：context上下文和bean。

如中间图所示，所有被spring管理的、由spring创建的、用于依赖注入的对象，就叫做一个bean。Spring创建并完成依赖注入后，所有bean统一放在一个叫做context的上下文中进行管理。

#### 3.AOP

AOP就是面向切面编程。如上图，一般程序执行流程是从controller层调用service层、然后service层调用DAO层访问数据，最后在逐层返回结果。
这个是图中向下箭头所示的按程序执行顺序的纵向处理。但是，一个系统中会有多个不同的服务，例如用户服务、商品信息服务等等，每个服务的controller层都需要验证参数，都需要处理异常，如果按照图中红色的部分，对不同服务的纵向处理流程进行横切，在每个切面上完成通用的功能，例如身份认证、验证参数、处理异常等等、这样就不用在每个服务中都写相同的逻辑了，这就是AOP思想解决的问题。

AOP以功能进行划分，对服务顺序执行流程中的不同位置进行横切，完成各服务共同需要实现的功能。

### spring框架组件
![](https://bucketyy.oss-cn-beijing.aliyuncs.com/image/spring/spring4.jpg)

上图列出了spring框架主要包含的组件。这张图来自spring4.x的文档。目前最新的5.x版本中右面的portlet组件已经被废弃掉，同时增加了用于异步响应式处理的WebFlux组件。

并不需要对所有的组件都详细了解，只需重点了解最常用的几个组件实现，以及知道每个组件用来实现哪一类功能。

图中红框是比较重要的组件，core组件是spring所有组件的核心；bean组件和context组件我刚才提到了，是实现IOC和依赖注入的基础；AOP组件用来实现面向切面编程；web组件包括springmvc是web服务的控制层实现。

### spring中机制和实现
![](https://bucketyy.oss-cn-beijing.aliyuncs.com/image/spring/spring5.jpg)

#### 1.AOP

AOP的实现是通过代理模式，在调用对象的某个方法时，执行插入的切面逻辑。实现的方式有动态代理也叫运行时增强，比如jdk代理、CGLIB；静态代理是在编译时进行织入或类加载时进行织入，比如AspectJ。

关于AOP还需要了解一下对应的Aspect、pointcut、advice等注解和具体使用方式。

#### 2.placeHolder动态替换

主要需要了解替换发生的时间，是在bean definition创建完成后，bean初始化之前，是通过实现BeanFactoryPostProcessor接口实现的。主要实现方式有PropertyPlaceholderConfigurer和PropertySourcesPlaceholderConfigurer。这两个类实现逻辑不一样，spring boot使用PropertySourcesPlaceholderConfigurer实现。

#### 3.事务

需要了解spring 中对事务规定的隔离类型和事务传播类型。要知道事务的隔离级别是由具体的数据库来实现的，在数据库部分我会详细介绍。

事务的传播类型，可以重点了解最常用的REQUIRED和SUPPORTS类型。

#### 4.核心接口类

ApplicationContext保存了ioc的整个应用上下文，可以通过其中的beanfactory获取到任意到bean；
BeanFactory主要的作用是根据bean definition来创建具体的bean；
BeanWrapper是对Bean的包装，一般情况下是在spring ioc内部使用，提供了访问bean的属性值、属性编辑器注册、类型转换等功能，方便ioc容器用统一的方式来访问bean的属性；
FactoryBean通过getObject方法返回实际的bean对象，例如motan框架中referer对service的动态代理就是通过FactoryBean来实现的。
#### 5.Scope

bean的scope是指bean的作用域，默认情况下是单例模式，这也是使用最多的一种方式；多例模式，即每次从beanFactory中获取bean都会创建一个新的bean。

request、session、global-session是在web服务中使用的scope，request每次请求都创建一个实例，session是在一个会话周期内保证只有一个实例。

global-session在5.x版本中已经不在使用，同时增加了Application和Websocket两种scope，分别保证在一个ServletContext与一个WebSocket中只创建一个实例。

#### 6.事件机制

spring的事件机制需要知道spring定义的五种标准事件，具体事件可见上图，了解如何自定义事件和实现对应的applicationListener来处理自定义事件。

### spring应用相关
![](https://bucketyy.oss-cn-beijing.aliyuncs.com/image/spring/spring6.jpg)

#### 1.常用注释

* 类型类注释：

类型类注释包括controller、service等，需要重点了解

其中component和bean注解的区别如下：

@Component注解在类上使用表明这个类是个组件类，需要Spring为这个类创建bean。
@Bean注解使用在方法上，告诉Spring这个方法将会返回一个Bean对象，需要把返回的对象注册到Spring的应用上下文中。
* 设置类注解

重点了解@Autowire和@Qualifier以及bytype、byname等不同的自动装配机制。

* web类注解

主要以了解为主，关注@RequestMapping、@GetMapping、@PostMapping等路径匹配注解，以及@PathVariable、@RequestParam 等参数获取注解。

* 功能类注解

包括@ImportResource引用配置、@ComponentScan注解自动扫描、@Transactional事务注解等等，这里不一一介绍了。

#### 2.配置方式

需要了解配置spring的几种方式，xml文件配置、注解配置和使用api进行配置。

自动装配机制需要了解按类型匹配进行自动装配，按bean名称进行自动装配，构造器中的自动装配和自动检测等主要的四种方式。

还需要了解一下list、set、map等集合类属性的配置方式以及内部bean的使用。

### Spring的Context的初始化流程
![](https://bucketyy.oss-cn-beijing.aliyuncs.com/image/spring/spring7.jpg)

图中左上角是三种类型的context，xml配置方式的context、springboot的context和web服务的context。不论哪种context，创建后都会调用到AbstractApplicationContext类的refresh方法，这个方法是我们要重点分析的。

refresh方法中，操作共分13步：

第1步：对刷新进行准备，包括设置开始时间、设置激活状态、初始化context环境中的占位符，这个动作根据子类的需求由子类来执行，然后验证是否缺失必要的properties；

第2步：刷新并获得内部的bean factory；

第3步：对bean factory进行准备工作，比如设置类加载器和后置处理器、配置不进行自动装配的类型、注册默认的环境bean；

第4步：为context的子类提供后置处理bean factory的扩展能力。如果子类想在bean定义加载完成后，开始初始化上下文之前做一些特殊逻辑，可以复写这个方法；

第5步，执行context中注册的bean factory后缀处理器；

注：这里有两种后置处理器，一种是可以注册bean的后缀处理器，另一种是针对bean factory进行处理的后置处理器。执行的顺序是，先按优先级执行可注册bean的处理器，在按优先级执行针对beanfactory的处理器。

对springboot来说，这一步会进行注解bean definition的解析。流程如右面小框中所示，由ConfigurationClassPostProcessor触发、由ClassPathBeanDefinitionScanner解析并注册到bean factory。

第6步：按优先级顺序在beanfactory中注册bean的后缀处理器，bean后置处理器可以在bean初始化前、后执行处理；

第7步：初始化消息源，消息源用来支持消息的国际化；

第8步：初始化应用事件广播器。事件广播器用来向applicationListener通知各种应用产生的事件，是一个标准的观察者模式；

第9步，是留给子类的扩展步骤，用来让特定的context子类初始化其他的bean；

第10步，把实现了ApplicationListener的bean注册到事件广播器，并对广播器中的早期未广播事件进行通知；

第11步，冻结所有bean描述信息的修改，实例化非延迟加载的单例bean；

第12步，完成上下文的刷新工作，调用LifecycleProcessor的onFresh()方法以及发布ContextRefreshedEvent事件；

第13步：在finally中，执行第十三步，重置公共的缓存，比如ReflectionUtils中的缓存、AnnotationUtils中的缓存等等；

至此，spring的context初始化主流程完成。

### Spring中bean的生命周期
![](https://bucketyy.oss-cn-beijing.aliyuncs.com/image/spring/spring8.jpg)

Spring中bean的生命周期，先看绿色的部分，bean的创建过程：

第1步：调用bean的构造方法创建bean；

第2步：通过反射调用setter方法进行属性的依赖注入；

第3步：如果实现BeanNameAware接口的话，会设置bean的name；

第4步：如果实现了BeanFactoryAware，会把bean factory设置给bean；

第5步：如果实现了ApplicationContextAware，会给bean设置ApplictionContext；

第6步：如果实现了BeanPostProcessor接口，则执行前置处理方法；

第7步：实现了InitializingBean接口的话，执行afterPropertiesSet方法；

第8步：执行自定义的init方法；

第9步：执行BeanPostProcessor接口的后置处理方法。

这时，就完成了bean的创建过程。

在使用完bean需要销毁时，会先执行DisposableBean接口的destroy方法，然后在执行自定义的destroy方法。


### Spring扩展接口
![](https://bucketyy.oss-cn-beijing.aliyuncs.com/image/spring/spring9.jpg)

对spring进行定制化功能扩展时，可以选择如下一些扩展点：

#### 1.BeanFactoryPostProcessor

是beanFactory后置处理器，支持在bean factory标准初始化完成后，对bean factory进行一些额外处理。在讲context初始化流程时介绍过，这时所有的bean的描述信息已经加载完毕，但是还没有进行bean初始化。例如前面提到的PropertyPlaceholderConfigurer，就是在这个扩展点上对bean属性中的占位符进行替换。

#### 2.BeanDefinitionRegistryPostProcessor

它扩展自BeanFactoryPostProcessor，在执行BeanFactoryPostProcessor的功能前，提供了可以添加bean definition的能力，允许在初始化一般bean前，注册额外的bean。例如可以在这里根据bean的scope创建一个新的代理bean。

#### 3.BeanPostProcessor

提供了在bean初始化之前和之后插入自定义逻辑的能力。与BeanFactoryPostProcessor的区别是处理的对象不同，BeanFactoryPostProcessor是对beanfactory进行处理，BeanPostProcessor是对bean进行处理。

*** 注：上面这三个扩展点，可以通过实现Ordered和PriorityOrdered接口来指定执行顺序。实现PriorityOrdered接口的processor会先于实现Ordered接口的执行。***

#### 4.ApplicationContextAware

可以获得ApplicationContext及其中的bean，当需要在代码中动态获取bean时，可以通过实现这个接口来实现。

#### 5.InitializingBean

可以在bean初始化完成，所有属性设置完成后执行特定逻辑，例如对自动装配对属性进行验证等等。

#### 6.DisposableBean

用于在bean被销毁前执行特定的逻辑，例如做一些回收工作等。

#### 7.ApplicationListener

用来监听spring的标准应用事件或者自定义事件。


七、springboot相关的知识点
![](https://bucketyy.oss-cn-beijing.aliyuncs.com/image/spring/spring10.jpg)

#### 1.启动流程

主要步骤首先要配置environment，然后准备context上下文，包括执行applicationContext的后置处理、初始化initializer、通知listener处理contextPrepared和contextLoaded事件。最后执行refreshContext，也就是前面介绍过的AbstractApplicationContext类的refresh方法。

#### 2.配置文件

然后要知道在Spring Boot中有两种上下文，一种是bootstrap, 另外一种是application。

bootstrap是应用程序的父上下文，也就是说bootstrap会先于applicaton加载。bootstrap主要用于从额外的资源来加载配置信息，还可以在本地外部配置文件中解密属性。bootstrap里面的属性会优先加载，默认也不能被本地相同配置覆盖。

#### 3.注解

@SpringBootApplication包含了@ComponentScan、@EnableAutoConfiguration、@SpringBootConfiguration三个注解

而@SpringBootConfiguration注解包含了@Configuration注解。也就是springboot的自动配置功能。

@Conditional注解就是控制自动配置的生效条件的注解，例如bean或class存在、不存在时进行配置，当满足条件时进行配置等等。

#### 4.特色模块

starter是springboot提供的无缝集成功能的一种方式，使用某个功能时开发者不需要关注各种依赖库的处理，不需要具体的配置信息，由Spring Boot自动配置进行bean的创建。例如需要使用web功能时，只需要在依赖中引入spring-boot-starter-web即可。
actuator是用来对应用程序进行监视和管理，通过restful api请求来监管、审计、收集应用的运行情况。
devtools提供了一系列开发工具的支持，来提高开发效率。例如热部署能力等。
CLI就是命令行接口，是一个命令行工具，支持使用Groovy脚本，可以快速搭建spring原型项目。

>以上内容摘取自《32个Java面试必考点》 第07讲：必会框架-Spring全家桶

