# Spring

[TOC]

## 控制反转和依赖注入

> 控制反转（IoC）对象的创建和销毁都交由框架负则，控制对象生存周期的不再是引用它的对象，而是spring框架。
>
> 依赖注入（DI）：在系统运行中，动态的向某个对象提供他所需要的的对象，向构造函数传递参数，或者通过使用 setter 方法。（通过反射实现）

## IoC容器

### 容器类型

| 类型               |                                         |
| ------------------ | --------------------------------------- |
| BeanFactory        | 适合轻量级应用                          |
| ApplicationContext | 是 BeanFactory 的子接口，功能更强大一些 |

### Bean的作用域

| 作用域         | 说明                                                         | 备注                            |
| -------------- | ------------------------------------------------------------ | ------------------------------- |
| singleton      | 仅存在一个bean实例，默认                                     |                                 |
| prototype      | 每次从容器中调用bean都返回一个新的实例，即每次调用getBean()，相当于执行newBean() |                                 |
| request        | 每次http请求都创建一个新的bean                               | 仅适用WebApplicationContext环境 |
| session        | 同一个http session共享一个bean，不同session使用不同的bean    | 仅适用WebApplicationContext环境 |
| global session |                                                              | 仅适用WebApplicationContext环境 |

单独配置
```xml
<bean id="helloWorld" class="com.my.HelloWorld" scope="singleton" init-method="init" destroy-method="destroy">
	<property name="message" value="Hello World!"/>
</bean>
```
通用配置

```xml
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://www.springframework.org/schema/beans
    http://www.springframework.org/schema/beans/spring-beans-3.0.xsd"
    default-init-method="init" 
    default-destroy-method="destroy">
```

### Bean的生命周期

1、实例化、

2、设置对象属性（依赖注入）

3、初始化 init-method

4、使用

5、销毁 destory-method

```xml
<bean id="helloWorld" class="com.my.HelloWorld" init-method="init" destroy-method="destroy">
	<property name="message" value="Hello World!"/>
</bean>
```

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

### 定义继承

子bean继承父bean的属性（子类中也要定义message1变量，否则xml会提示异常）

```xml
<bean id="helloWorld" class="com.my.HelloWorld">
    <property name="message1" value="Hello World 1"/>
    <property name="message2" value="Hello World 2"/>
</bean>

<bean id="helloChina" class="com.my.HelloChina" parent="helloWorld">
    <property name="message2" value="Hello China 1"/>
    <property name="message3" value="Hello China 2"/>
</bean>
```

#### bean模板

父bean被定义为抽象，不能被实例化

```xml
 <bean id="teamplate" abstract="true">
    <property name="message1" value="Hello World 1"/>
    <property name="message2" value="Hello World 2"/>
    <property name="message3" value="Hello World 3"/>
</bean>

<bean id="helloChina" class="com.my.HelloChina" parent="teamplate">
    <property name="message2" value="Hello China 1"/>
    <property name="message3" value="Hello China 2"/>
</bean>
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

按顺序

```xml
<beans>
   <bean id="TestBean" class="com.my.TestMy">
      <constructor-arg ref="test1"/>
      <constructor-arg ref="test2"/>
   </bean>

   <bean id="test1" class="com.my.Test1"/>
   <bean id="test2" class="com.my.Test2"/>
</beans>
```
按类型
```xml
<beans>
   <bean id="exampleBean" class="examples.ExampleBean">
      <constructor-arg type="int" value="111"/>
      <constructor-arg type="java.lang.String" value="hello"/>
   </bean>
</beans>
```
按位置
```xml
<beans>
   <bean id="exampleBean" class="examples.ExampleBean">
      <constructor-arg index="0" value="111"/>
      <constructor-arg index="1" value="222"/>
   </bean>
</beans>
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



```xml
<beans>
   <bean id="TestBean" class="com.my.TestMy">
      <property name="test1" ref="test1"/>
   </bean>

   <bean id="test1" class="com.my.Test1"/>
</beans>
```

```xml
<beans>
   <bean id="TestBean" class="com.my.TestMy">
      <property name="test1">
          <bean id="test1" class="com.my.Test1"/>
       </property>
   </bean>
</beans>
```

#### 注入集合

```xml
<bean id="javaCollection" class="com.my.JavaCollection">

      <!-- results in a setAddressList(java.util.List) call -->
      <property name="addressList">
         <list>
            <value>INDIA</value>
            <value>Pakistan</value>
            <value>USA</value>
            <value>USA</value>
         </list>
      </property>

      <!-- results in a setAddressSet(java.util.Set) call -->
      <property name="addressSet">
         <set>
            <value>INDIA</value>
            <value>Pakistan</value>
            <value>USA</value>
            <value>USA</value>
        </set>
      </property>

      <!-- results in a setAddressMap(java.util.Map) call -->
      <property name="addressMap">
         <map>
            <entry key="1" value="INDIA"/>
            <entry key="2" value="Pakistan"/>
            <entry key="3" value="USA"/>
            <entry key="4" value="USA"/>
         </map>
      </property>

      <!-- results in a setAddressProp(java.util.Properties) call -->
      <property name="addressProp">
         <props>
            <prop key="one">INDIA</prop>
            <prop key="two">Pakistan</prop>
            <prop key="three">USA</prop>
            <prop key="four">USA</prop>
         </props>
      </property>

   </bean>

</beans>
```

### 自动装配

#### 默认

使用<constructor-arg>和<property>

```xml
<beans>
   <bean id="TestBean" class="com.my.TestMy">
      <property name="test1" ref="test1"/>
      <property name="message" value="hello"/>
   </bean>

   <bean id="test1" class="com.my.Test1"/>
</beans>
```

#### byName或byType

```xml
<beans>
   <bean id="TestBean" class="com.my.TestMy" autowire="byName/byType">
      <property name="message" value="hello"/>
   </bean>

   <bean id="test1" class="com.my.Test1"/>
</beans>
```

#### 构造函数

```xml
<beans>
   <bean id="TestBean" class="com.my.TestMy" autowire="constructor">
      <constructor-arg value="hello"/>
   </bean>

   <bean id="test1" class="com.my.Test1"/>
</beans>
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

|               |                                    |
| ------------- | ---------------------------------- |
| aspect        | 一组横切需求的执行方法             |
| join point    |                                    |
| advice        | AOP切面具体执行的方法              |
| pointcut      | 切入一个类，所有方法都按需求走通知 |
| introduction  |                                    |
| target object |                                    |
| weaving       |                                    |

通知类型

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

### 编程式事务

> 写入代码

### 声明式事务

> 使用注释或基于配置xml

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