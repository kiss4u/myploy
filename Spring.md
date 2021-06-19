# Spring

[TOC]

## 控制反转和依赖注入

> 控制反转（IoC）对象的创建和销毁都交由框架负责，控制对象生存周期的不再是引用它的对象，而是spring框架。IOC容器本质上是一个map，存放的是各种对象；
>
> 依赖注入（DI）：在系统运行中，动态的向某个对象提供他所需要的的对象，向构造函数传递参数，或者通过使用 setter 方法。（通过反射实现）

## IoC容器

### IOC的好处

> 减少代码量
>
> 减少入侵松耦合
>
> 支持即时实例化及懒加载

### 容器类型

| 类型                  |                                         |                                |
| --------------------- | --------------------------------------- | ------------------------------ |
| BeanFactory           | 适合轻量级应用                          | 调用getBean方法时才实例化bean  |
| ApplicationContext    | 是 BeanFactory 的子接口，功能更强大一些 | 初始化时会实例化所有的单例bean |
| WebApplicationContext | 专门为web应用准备的                     |                                |

### Bean的作用域（scope）

> ```text
> @Scope(ConfigurableBeanFactory.SCOPE_PROTOTYPE)
> ```

| 作用域         | 说明                                                         | 备注                            |
| -------------- | ------------------------------------------------------------ | ------------------------------- |
| singleton      | 仅存在一个bean实例，默认                                     |                                 |
| prototype      | 每次从容器中调用bean都返回一个新的实例，即每次调用getBean()，相当于执行newBean() |                                 |
| request        | 每次http请求都创建一个新的bean                               | 仅适用WebApplicationContext环境 |
| session        | 同一个http session共享一个bean，不同session使用不同的bean    | 仅适用WebApplicationContext环境 |
| global session |                                                              | 仅适用WebApplicationContext环境 |

### Bean的生命周期

> 1、根据配置文件实例化bean
>
> 2、bean的属性注入
>
> 3、如果bean对应的类实现了BeanNameAware接口，调用setBeanName方法；如果实现了BeanFactoryAware接口，调用setBeanFactory方法
>
> 4、如果配置了后置处理器，执行方法
>
> 5、如果bean对应的类中有@PostConstruct，且开启了注解扫描，执行该初始化方法
>
> 6、

![img](https://images2015.cnblogs.com/blog/1071792/201703/1071792-20170319001837916-2023744929.png)

![img](https://img2018.cnblogs.com/blog/842514/201906/842514-20190611220552139-1446928813.png)

### 后置处理器

实现BeanPostProcessor接口，多个后置处理器可实现Order接口调整顺序（默认按xml配置顺序执行）

**ApplicationContext** 会自动检测由 **BeanPostProcessor** 接口的实现定义的 bean，注册这些 bean 为后置处理器

```java
public class ProcessorOne implements BeanPostProcessor, Ordered {
    
    public Object postProcessBeforeInitialization(Object bean, String beanName) throws BeansException {
        System.out.println("bean初始化之前");
        return bean;
    }

    public Object postProcessAfterInitialization(Object bean, String beanName) throws BeansException {
        System.out.println("bean初始化之后");
        return bean;
    }

    /**
     * 多个后置处理器的调用顺序，值越大优先级越低
     *
     * @return
     */
    public int getOrder() {
        return 0;
    }
}
```

## 依赖注入

### 注入方式

#### 构造方法

```java
public class TestMy {
   public TestMy(Test1 test1, Test2 test2) {
      // ...
   }
}
```

#### setter方法

```java
public class TestMy {
   private Test1 test1;
   public void setTest1(Test1 test1) {
      this.test1 = test1;
   }
}
```

## 基于注解配置

| 注解                        | 说明                                 |      |
| --------------------------- | ------------------------------------ | ---- |
| @Required                   | 参数必须配置                         |      |
| @Autowired                  | byType自动连接                       |      |
| @Qualifier                  | 装载指定的bean                       |      |
| @PostConstruct、@PreDestroy | 指定init、destory方法                |      |
| @Resource                   | 如果指定了name，相当于byName自动连接 |      |
| @Configuration              | 类可以作为Bean的来源被Spring装载     |      |
| @Bean                       |                                      |      |

## 事件

通过ApplicationListener接口发布事件
```java
public class CStartEventHandler 
   implements ApplicationListener<ContextStartedEvent>{
   public void onApplicationEvent(ContextStartedEvent event) {
      System.out.println("ContextStartedEvent Received");
   }
}

public class MainApp {
   public static void main(String[] args) {
      ConfigurableApplicationContext context = 
      new ClassPathXmlApplicationContext("Beans.xml");

      // Let us raise a start event.
      context.start();

      HelloWorld obj = (HelloWorld) context.getBean("helloWorld");
      obj.getMessage();

   }
}
```

| 类型                  |                                                              |
| --------------------- | ------------------------------------------------------------ |
| ContextRefreshedEvent | ApplicationContext 被初始化或刷新时，该事件被发布。也可以使用 ConfigurableApplicationContext 接口中 refresh() 方法来发生。 |
| ContextStartedEvent   | 使用 ConfigurableApplicationContext 接口中的 start() 方法启动 ApplicationContext 时，该事件被发布 |
| ContextStoppedEvent   | 使用 ConfigurableApplicationContext 接口中的 stop() 方法停止 ApplicationContext 时，该事件被发布 |
| ContextClosedEvent    | 使用 ConfigurableApplicationContext 接口中的 close() 方法关闭 ApplicationContext 时，该事件被发布 |
| RequestHandledEvent   | 告诉所有 bean HTTP 请求已经被服务                            |

>由于 Spring 的事件处理是单线程的，所以如果一个事件被发布，直至并且除非所有的接收者得到的该消息，该进程被阻塞并且流程将不会继续。因此，如果事件处理被使用，在设计应用程序时应注意。

### 自定义事件





## AOP

面向切面编程

将那些与业务无关的，被业务模块共同调用的逻辑或责任封装起来，减少代码的重复，提高扩展性和维护性

![img](https://img-blog.csdnimg.cn/20200520191206508.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80NDM5NTcwNw==,size_16,color_FFFFFF,t_70)

### 实现方式

#### 静态代理

##### 编译时编织

> 特殊的编译器

##### 类加载期间- LoadTimeWeaving

> 在类的class文件未被jvm加载之前，通过自定义的类加载器或文件转换器，将功能织入目标类的class文件，再交给jvm加载

#### 动态代理

> 在运行时在内存中“临时”生成 AOP 动态代理类

##### JDK动态代理

##### CGLIB

|               |                                    |      |
| ------------- | ---------------------------------- | ---- |
| aspect        | 切面 |  一组横切需求的执行方法    |
| joinpoint     | 切点               | 程序运行中的一些时间点, 例如一个方法的执行, 或者是一个异常的处理 |
| advice        | 通知                 | 特定 JoinPoint 处的 Aspect 所采取的动作 |
| pointcut      |  | 切入一个类，所有方法都按需求走通知 |
| introduction  |                                    |      |
| target object |                                    |      |
| weaving       | 增强添加到目标对象                 |      |

### 有哪些通知类型（advice）

|                |                 |                                |
| -------------- | --------------- | ------------------------------ |
| 前置通知       | @Before("XXX")  | 在方法执行前通知               |
| 后置通知       | @After          | 在方法执行后通知，不考虑结果   |
| 返回后通知     | @AfterReturning | 在方法执行成功之后，才执行通知 |
| 抛出异常后通知 | @AfterThrowing  | 方法抛出异常时通知             |
| 环绕通知       | @Around         | 方法执行前后都通知             |

```java
@Aspect
public class Logging {

   @Pointcut("execution(* com.my.Student.*(..))")
   private void selectAll(){}

   @Before("selectAll()")
   public void beforeAdvice(){
      System.out.println("Going to setup student profile.");
   }

   @After("selectAll()")
   public void afterAdvice(){
      System.out.println("Student profile has been setup.");
   }

   @AfterReturning(pointcut = "selectAll()", returning="retVal")
   public void afterReturningAdvice(Object retVal){
      System.out.println("Returning:" + retVal.toString() );
   }

   @AfterThrowing(pointcut = "selectAll()", throwing = "ex")
   public void AfterThrowingAdvice(IllegalArgumentException ex){
      System.out.println("There has been an exception: " + ex.toString());   
   }  
}

public class MainApp {
   public static void main(String[] args) {
      ApplicationContext context = 
             new ClassPathXmlApplicationContext("Beans.xml");
      Student student = (Student) context.getBean("student");
      student.getName();
      student.getAge();     
      student.printThrowException();
   }
}
```

## spring事务管理

### 事务管理方式

> 编程式事务：写入代码（不推荐）
>
> 声明式事务：使用注释或基于配置xml

### @Transaction注解可以放在哪些地方

#### 类

> 1、表明该类下所有public方法都配置事务属性
>
> 2、protected、private修饰的方法上使用 @Transactional 注解，事务无效但不会有报错

#### 方法

> 1、表明该方法配置事务属性
>
> 2、会覆盖类上配置的事务属性

### spring事务隔离级别

基本同mysql，多了一个默认使用数据库

#### isolation_default

> 使用数据库默认隔离级别

#### isolation_read_uncommitted

> 读未提交

#### isolation_read_committed

> 读已提交

#### isolation_repeatable_read

> 可重复度

#### isolation_serializable

> 串行

### spring事务传播机制

#### 支持当前事务情况

propagation_required

> 如果当前存在事务，则加入该事务，如果不存在则创建一个

propagation_supports

>如果当前存在事务，则加入该事务，如果不存在则以非事务方式运行

propagation_mandatory

>如果当前存在事务，则加入该事务，如果不存在则抛异常

#### 不支持当前事务情况

propagation_requires_new

> 创建一个事务，如果当前存在事务，则把当前事务挂起

propagation_not_support

>以非事务方式运行，如果当前存在事务，则把当前事务挂起

propagation_never

> 以非事务方式运行，如果当前存在事务，则抛出异常

#### 其他情况

propagation_nested

> 如果当前存在事务，则创建一个事务作为当前事务的嵌套事务运行，如果不存在，则等价于propagation_required

### @Transaction失效的场景

#### 配置在了非public修饰的方法上

> 

#### 注解属性 propagation 设置错误

>TransactionDefinition.PROPAGATION_SUPPORTS：如果当前存在事务，则加入该事务；如果当前没有事务，则以非事务的方式继续运行。
>
>TransactionDefinition.PROPAGATION_NOT_SUPPORTED：以非事务方式运行，如果当前存在事务，则把当前事务挂起。
>
>TransactionDefinition.PROPAGATION_NEVER：以非事务方式运行，如果当前存在事务，则抛出异常。

#### rollbackFor指定异常类型，但是抛出的是别的异常

>

#### 同一个类中方法调用

> 1、比如说一个类中A，B两个方法，A未配置事务，B配置了事务；外部调用A方法，A方法调用B方法，此时B的事务不会生效
>
> 2、事务方法被当前类以外的代码调用时，才会aop生成代理对象来管理，A方法虽然被外部调用，但是A方法没配置事务，也就无法传播到B

#### 方法加了trycache，不抛出异常导致事务失效

>

#### 数据库引擎不支持事务

>比如表设置了myisam引擎

## 三大器

### 监听器

```java
@Service
public class HeartbeatEventListener {

	@EventListener(condition = "#event.userInfo.isMale()")
    public void doSomething(HeartbeatEvent event) {
       
        System.out.println("username：" + userInfo.getUserId());
    }
}
```

### 拦截器

#### 实现HandlerInterceptor接口

>preHandle 在controller方法调用之前执行
>
>postHandle 在controller方法调用之后执行
>
>afterCompletion 在整个请求结束之后执行

```java
public class LogCostInterceptor implements HandlerInterceptor {
    long start = System.currentTimeMillis();
    @Override
    public boolean preHandle(HttpServletRequest httpServletRequest, HttpServletResponse httpServletResponse, Object o) throws Exception {
        start = System.currentTimeMillis();
        return true;
    }
 
    @Override
    public void postHandle(HttpServletRequest httpServletRequest, HttpServletResponse httpServletResponse, Object o, ModelAndView modelAndView) throws Exception {
        System.out.println("Interceptor cost="+(System.currentTimeMillis()-start));
    }
 
    @Override
    public void afterCompletion(HttpServletRequest httpServletRequest, HttpServletResponse httpServletResponse, Object o, Exception e) throws Exception {
    }
}


@Configuration
public class InterceptorConfig extends WebMvcConfigurerAdapter {
 
    @Override
    public void addInterceptors(InterceptorRegistry registry) {
        registry.addInterceptor(new LogCostInterceptor()).addPathPatterns("/**");
        super.addInterceptors(registry);
    }
}
```



### 过滤器

application类需加@ServletComponetScan扫描

#### 实现filter接口

```java
public class LogCostFilter implements Filter {
    @Override
    public void init(FilterConfig filterConfig) throws ServletException {
 
    }
 
    @Override
    public void doFilter(ServletRequest servletRequest, ServletResponse servletResponse, FilterChain filterChain) throws IOException, ServletException {
        long start = System.currentTimeMillis();
        filterChain.doFilter(servletRequest,servletResponse);
        System.out.println("Execute cost="+(System.currentTimeMillis()-start));
    }
 
    @Override
    public void destroy() {
 
    }
}


@Configuration
public class FilterConfig {
 
    @Bean
    public FilterRegistrationBean registFilter() {
        FilterRegistrationBean registration = new FilterRegistrationBean();
        registration.setFilter(new LogCostFilter());
        registration.addUrlPatterns("/*");
        registration.setName("LogCostFilter");
        registration.setOrder(1);
        return registration;
    }
 
}
```

#### 通过@WebFilter配置

```java
@WebFilter(urlPatterns = "/*", filterName = "logFilter2")
public class LogCostFilter2 implements Filter {
    @Override
    public void init(FilterConfig filterConfig) throws ServletException {
 
    }
 
    @Override
    public void doFilter(ServletRequest servletRequest, ServletResponse servletResponse, FilterChain filterChain) throws IOException, ServletException {
        long start = System.currentTimeMillis();
        filterChain.doFilter(servletRequest, servletResponse);
        System.out.println("LogFilter2 Execute cost=" + (System.currentTimeMillis() - start));
    }
 
    @Override
    public void destroy() {
 
    }
}
```

### 拦截器和过滤器主要区别

> 过滤器能做的，拦截器都能做，功能更强大，可以在请求前请求后执行，比较灵活
>
> 拦截器只能对action（也就是controller）请求起作用，而过滤器则可以对几乎所有的请求起作用，并且可以对请求的资源进行起作用
>
> 拦截器在action的生命周期可以被多次调用，过滤器只能在最开始调用一次

## spring是如何解决循环依赖的

>1、通过ApplicationContext.getBean，递归方式获取bean及其依赖的bean
>
>2、实例化bean时，分两步进行，首先实例化bean放入二级缓存，完成注入属性后再放入一级缓存
>
>
>
>对于依赖的class，会先从二级缓存中获取对象给递归上层，真正使用时从一级缓存中获取
>
>(待确认)



![img](https://pic1.zhimg.com/80/v2-6fd43ded717a9c31b0a9abae0234db9b_720w.jpg?source=1940ef5c)

## spring内部有三级缓存

### singletonObjects

> 一级缓存，用于保存实例化、注入、初始化完成的bean实例

### earlySingletonObjects 

> 二级缓存，用于保存实例化完成的bean实例

### singletonFactories 

> 三级缓存，用于保存bean创建工厂，以便于后面扩展有机会创建代理对象

#### 单例setter注入

```java
@Service
public class TestService1 {

    @Autowired
    private TestService2 testService2;

    public void test1() {
    }
}

@Service
public class TestService2 {

    @Autowired
    private TestService1 testService1;

    public void test2() {
    }
}
```

![img](https://pic2.zhimg.com/80/v2-1e7bd042df73e47bb951e70b298c96ca_720w.jpg?source=1940ef5c)



![img](https://pic2.zhimg.com/80/v2-85b44d9fdb16b78e9af3690b929cbff2_720w.jpg?source=1940ef5c)

### 生成代理对象产生的循环依赖

> 使用`@Lazy`注解延迟加载
>
> 使用`@DependsOn`注解，指定加载先后顺序
>
> 修改文件名称，改变循环依赖类的加载顺序

### 使用@DependsOn产生的循环依赖

> 找到`@DependsOn`注解循环依赖的地方调整

#### 多例循环依赖

> 改为单例

#### 构造器循环依赖

> 使用`@Lazy`

### 第三级缓存中为什么要添加`ObjectFactory`对象，直接保存实例对象不行吗？

> 不行，因为假如你想对添加到三级缓存中的实例对象进行增强，直接用实例对象是行不通的

## spring框架中用到了哪些设计模式

> 工厂模式 		通过BeanFactory和ApplicationContext创建bean
>
> 单例模式 		bean默认都是单例
>
> 代理模式 		AOP的实现
>
> 适配器模式	

## 将一个类声明为bean的注解有哪些？

> @Compmant 通用注解，用于可标注任意类，如果bean不知道归属于那一层，可以使用
>
> @Service		对应服务层
>
> @Controller	对应控制层层
>
> @Repostory	对应持久层

## Spring中的Singleton bean线程安全吗？

> 不是

## 哪些是重要的bean生命周期方法？可以覆盖它们吗

> init-method 				@PostConstruct
>
> destroy-method		@PreDestroy