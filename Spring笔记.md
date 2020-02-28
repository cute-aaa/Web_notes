# Spring

特点：

1. spring是一个设计良好的webMVC框架
2. 使用依赖注入（DI）实现了控制反转（IoC）依赖注入见印象笔记
3. 面向方面的程序设计（AOP）。OOP中模块化的关键单元是类，而AOP中模块化的关键单元是方面。             一个程序中跨越多个点的功能被称为横切关注点，比如日志记录、声明性事务、安全性、缓存等（辣鸡翻译）

![Spring 体系结构](https://7n.w3cschool.cn/attachments/image/wk/wkspring/arch1.png)

## Application接口

ApplicationContext是BeanFactory的子接口，也叫Spring上下文。

ApplicationContext包含了BeanFactory的所有功能，如：可以加载配置文件中定义的Bean，将所有的bean集中在一起，当有请求的时候分配bean。另外还增加了企业所需要的功能，如：从属性文件中解析文本信息、将事件传递给指定的监听器。

当然BeanFactory仍然可以在轻量级应用中使用，如移动设备或基于applet的应用。

##### 常用实现类：

- **FileSystemXmlApplicationContext**：从XML文件中加载已被定义的bean，需要提供给构造器XML文件的完整路径。即盘符：路径
- **ClassPathXmlApplicationContext** ：从XML文件中加载已被定义的bean，无需提供XML文件的完整路径，只需正确配置classpath环境变量即可。因为容器会从classpath中搜索bean配置文件
- **WebXmlApplicationContext**：该容器会在一个web应用程序的范围内加载在XML文件中已被定义的bean

## Bean

被称作bean的对象是构成应用程序的支柱，由Spring IoC容器管理。

bean是一个被实例化、组装、并通过Spring IoC容器所管理的对象。这些bean是由用容器提供的配置元数据创建的，如在XML表单中的定义。

bean的生命周期可以表达为：Bean的定义——初始化——使用——销毁![img](https://images2015.cnblogs.com/blog/332898/201606/332898-20160603162144914-1873567192.jpg)

**属性**：

class		指定用来创建bean的Java类

name		为class指定唯一的bean标识符，也可以使用id来指定

scope		指定bean对象的作用域

constructor-arg	用来注入依赖关系（构造注入）

properties	用来注入依赖关系（set注入）

auto-wiring mode	用来注入依赖关系（自动注入）

lazy-initialization方法	在bean的所有必须属性被容器设置后，调用回调方法

destruction方法		当包含该fean的容器被销毁时使用回调方法

##### 生存周期（scope的值）：

| 作用域             | 描述                                                         |
| ------------------ | ------------------------------------------------------------ |
| singleton （默认） | 在spring IoC容器仅存在一个Bean实例，Bean以单例方式存在，默认值 |
| prototype          | 每次从容器中调用Bean时，都返回一个新的实例，即每次调用getBean()时，相当于执行newXxxBean() |

Web应用中：

| 作用域         | 描述                                                         |
| -------------- | ------------------------------------------------------------ |
| request        | 每次HTTP请求都会创建一个新的Bean，该作用域仅适用于WebApplicationContext环境 |
| session        | 同一个HTTP Session共享一个Bean，不同Session使用不同的Bean，仅适用于WebApplicationContext环境 |
| global-session | 一般用于Portlet应用环境，该运用域仅适用于WebApplicationContext环境 |

其中singleton单例类型是在创建容器时就自动创建了一个bean的对象，用或不用，他都在那里。

而prototype原型类型的bean在创建容器时并没有被实例化，而是在每次请求该bean的时候（注入另一个bean，或调用容器的getBean方法时）才会去创建对象，并且每次都是不同的对象。

**构造与销毁：**

使用 **init-method** 和 **destroy-method** 来**指定**初始化和销毁时执行的函数。

- 接口实现：<br>实现org.springframework.beans.factory.InitializingBean接口中的afterPropertiesSet方法来初始化，<br>实现org.springframework.beans.factory.DisposableBean接口中的destroy方法
- XML配置实现：**（推荐）**<br>在bean中指定bean的类中的一个方法，如example包中的ExampleBean类中的init方法和destroy方法：

```xml
<bean id="exampleBean" 
         class="examples.ExampleBean" 
      init-method="init" destroy-method="destroy" />
```

定义一个默认的初始化和销毁方法：

```xml
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://www.springframework.org/schema/beans
    http://www.springframework.org/schema/beans/spring-beans-3.0.xsd"
    default-init-method="init" 
    default-destroy-method="destroy">

   <bean id="..." class="...">
       <!-- collaborators and configuration for this bean go here -->
   </bean>

</beans>
```

这样就会用每个bean里的init和destroy方法作为默认的构造析构方法。（注意不是使用某一个类同一个方法）

**Bean后置处理器：**

Bean后置处理器允许在调用初始化方法前后对Bean进行额外的处理

org.springframework.beans.factory.config.BeanPostProcessor接口中：

public Object postProcessBeforeInitialization(Object bean, String beanName) throws BeansException{};定义初始化之前的方法

public Object postProcessAfterInitialization(Object bean, String beanName) throws BeansException{};定义初始化之后的方法。

如果是非web应用，还需要在JVM中注册一个“关闭钩子”，来确保Spring IoC容器被恰当关闭并释放所有由单例持有的资源，只需调用：ApplicationContext接口的实现类中的registerShutdownHook()方法即可。

在XML文件中将实现类添加进去即可。



##### Bean继承：

在bean属性中添加parent属性来指定父类，同时将拥有父类的属性。并且可以重写父类的属性，引入自己的属性。

**Bean模板（抽象属性）**

将bean的abstract设为true来定义一个模板，模板不需要class属性。使用其他类来继承模板就可以使用模板里的值。如：

```xml
   <bean id="beanTeamplate" abstract="true">
      <property name="message1" value="Hello World!"/>
      <property name="message2" value="Hello Second World!"/>
      <property name="message3" value="Namaste India!"/>
   </bean>

   <bean id="helloIndia" class="com.tutorialspoint.HelloIndia" parent="beanTeamplate">
      <property name="message1" value="Hello India!"/>
      <property name="message3" value="Namaste India!"/>
   </bean>
```

当一个定义为抽象时，它仅仅作为一个纯粹的模板bean来使用，充当子定义的父定义。

模板自身不能被实例化，因为他是不完整且抽象的。

**DI**

如果要用引用来赋值就使用ref，使用普通变量就用value

- 构造注入：用于有强制依存关系的类。<br>在bean中使用< constructor-arg   ref="">标签来指定构造函数中要传的对象，赋值则在构造函数中完成。如：

```java
public class Foo {
   public Foo(int year, String name) {
      // ...
   }
}
```

```xml
<beans>
   <bean id="foo" class="x.y.Foo">
      <constructor-arg ref="bar"/>
      <constructor-arg ref="baz"/>
   </bean>

   <bean id="bar" class="x.y.Bar"/>
   <bean id="baz" class="x.y.Baz"/>
</beans>
```

也可以使用ype属性通过类型来赋值：

```xml
      <constructor-arg type="int" value="2001"/>
      <constructor-arg type="java.lang.String" value="Zara"/>
```

或者通过下标来赋值：

```xml
<constructor-arg index="0" value="2001"/>
      <constructor-arg index="1" value="Zara"/>
```

- set注入：给要赋值的变量添加set方法，并在bean中添加标签来赋值。

```xml
<property name="spellChecker" ref="com.dao.spellChecker"/>
```

或者直接使用：

```xml
<property name="name" value="John Doe"/>
<property name="spouse" ref="jane"/>
可以变为：
p:name="John Doe"
p:spouse-ref="jane"/>
```

可以在bean中定义内部bean用来赋值。如：

```xml
<bean id="textEditor" class="com.tutorialspoint.TextEditor">
      <property name="spellChecker">
         <bean id="spellChecker" 
               class="com.tutorialspoint.SpellChecker"/>
       </property>
</bean>
```

传递多个值：

list

set

map：键值对，名称和值可以是任何类型

props（properties）：键值对，名称和值都是字符串

```xml
   <bean id="javaCollection" class="com.tutorialspoint.JavaCollection">

      <!-- results in a setAddressList(java.util.List) call -->
      <property name="addressList">
         <list>
            <value>INDIA</value>
            <value>Pakistan</value>
            <ref bean = "helloworld" />
         </list>
      </property>

      <!-- results in a setAddressSet(java.util.Set) call -->
      <property name="addressSet">
         <set>
            <value>INDIA</value>
            <value>USA</value>
            <ref bean = "helloworld"/>
        </set>
      </property>

      <!-- results in a setAddressMap(java.util.Map) call -->
      <property name="addressMap">
         <map>
            <entry key="1" value="INDIA"/>
            <entry key="2" value-ref="helloworld"/>
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
       
	    注入 null 和空字符串的值：
        <bean id="..." class="exampleBean">
           <property name="email" value=""/>
        </bean>
        相当于：exampleBean.setEmail("")。

        <bean id="..." class="exampleBean">
           <property name="email"><null/></property>
        </bean>
        相当于：exampleBean.setEmail(null)。
   </bean>
```

- 自动注入：

byName：

条件：

xml文件里需要装配的属性名必须与已存在的id名相同

在bean里声明autowire属性为byName或在全局声明default-autowire为byName

如：hello.java里有String string1

则xml里必须使<bean id="string1"(与java文件里的string1相同) class="java.util.String";

byType：

条件：

在xml中指定autowire为byType或指定全局配置

根据元素类型来进行赋值，如果没有找到同类型的或找到多个同类型的则会抛出异常。也可以给接口或父类定义的对象赋值

constructor：

在容器中寻找与选哟自动装配的bean的构造参数一直的一个或多个bean，没有则抛异常

autodetect：

首先尝试使用constructor来自动装配，不行再使用byType方式

**使用注解来配置依赖注入：**

使用相关类、方法、字段声明的注解将bean配置移动到组件本身而不是xml文档

在spring配置文件中启用注解：

```xml
<?xml version="1.0" encoding="UTF-8"?>

<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:context="http://www.springframework.org/schema/context"
    xsi:schemaLocation="http://www.springframework.org/schema/beans
    http://www.springframework.org/schema/beans/spring-beans-3.0.xsd
    http://www.springframework.org/schema/context
    http://www.springframework.org/schema/context/spring-context-3.0.xsd">

   <context:annotation-config/>
   <!-- bean definitions go here -->

</beans>
```

几个重要的注解：

**@Required**

@Required注解应用于bean属性的setter方法，表明bean在配置时必须在xml配置，如果少了几个属性就会抛出BeanInitializationException 异常。比如有name和age属性，但在xml中只对name赋值，就会抛出异常。

**@Autowired**

@Autowired注解对在哪里和如何完成自动连接提供了更多的细微的控制，总之就是自动使用byType进行赋值。

- 在setter方法中：

在set方法前加一个@Autowired，他就会使用byType自动连接

- 在属性中：

在属性前加一个@Autowired，他就会找到bean中的同类型id自动赋值

- 在构造方法中：

会找到所需参数中的同类型id进行赋值

- - @Autowired(required=false)选项：

  默认情况下，@Autowired注释意味着依赖是必须的（类似于@Required），可以使用required=false关闭默认行为，即不把参数赋完整也可以运行了。

**@Qualifier**

如果使用了@Autowired注解，但bean中有多个同类型的id时，且想要指定使用哪一个来装配，就可以使用@Qualifier("id名")来指定。如xml中有两个id为stu1和stu2，想要通过stu1来赋值，使用：

@Autowired

@Qualifier("stu1")

就可以使用stu1赋值了

**其他**

- （因为有了xml，所以这些不常用了）
- @PostConstruct：相当于init-method
- @PreDestroy：相当于destroy-method
- @Resource：相当于byName注入，如@Resource(name="id名")就在xml中查找id名并赋值。如果没有指定name，默认字段名或属性名

#### 基于Java的配置

使用Java配置可以在不配置xml的情况下编写大多数的spring

**@Configuration**

带有@Configuration的注解类表示这个类可以作为bean来使用，与**@Bean**一同使用，带有@Bean注解的方法返回一个对象，该对象被注册为spring中的bean。如：

```java
package com.tutorialspoint;
import org.springframework.context.annotation.*;
@Configuration
public class HelloWorldConfig {
   @Bean 
   public HelloWorld helloWorld(){
      return new HelloWorld();
   }
}
```

就相当于：

```xml
<beans>
   <bean id="helloWorld" class="com.tutorialspoint.HelloWorld" />
</beans>
```

这样在使用配置时，就可以使用：

```java
ApplicationContext ctx = 
new AnnotationConfigApplicationContext(HelloWorldConfig.class);
ctx.getBean(HelloWorld.class);
```



-------

```java
public static void main(String[] args) {
   AnnotationConfigApplicationContext ctx = 
   new AnnotationConfigApplicationContext();
   ctx.register(AppConfig.class, OtherConfig.class);
   ctx.register(AdditionalConfig.class);
   ctx.refresh();
   MyService myService = ctx.getBean(MyService.class);
   myService.doStuff();
}
```

-----



带有@Bean注解的方法名作为bean的id，它创建并返回实际的bean，配置类可以声明多个@Bean。

例：

```java
package com.tutorialspoint;
import org.springframework.context.annotation.*;
@Configuration
public class AppConfig {
   @Bean
   public Foo foo() {
      return new Foo(bar());
   }
   @Bean
   public Bar bar() {
      return new Bar();
   }
}
```

一个比较完整的：

这里是 **TextEditorConfig.java** 文件的内容：

```java
package com.tutorialspoint;
import org.springframework.context.annotation.*;
@Configuration
public class TextEditorConfig {
   @Bean 
   public TextEditor textEditor(){
      return new TextEditor( spellChecker() );
   }
   @Bean 
   public SpellChecker spellChecker(){
      return new SpellChecker( );
   }
}
```

这里是 TextEditor.java 文件的内容：

```java
package com.tutorialspoint;
public class TextEditor {
   private SpellChecker spellChecker;
   public TextEditor(SpellChecker spellChecker){
      System.out.println("Inside TextEditor constructor." );
      this.spellChecker = spellChecker;
   }
   public void spellCheck(){
      spellChecker.checkSpelling();
   }
}
```

下面是另一个依赖的类文件 **SpellChecker.java** 的内容：

```java
package com.tutorialspoint;
public class SpellChecker {
   public SpellChecker(){
      System.out.println("Inside SpellChecker constructor." );
   }
   public void checkSpelling(){
      System.out.println("Inside checkSpelling." );
   }

}
```

下面是 MainApp.java文件的内容：

```java
package com.tutorialspoint;

import org.springframework.context.ApplicationContext;
import org.springframework.context.annotation.*;

public class MainApp {
   public static void main(String[] args) {
      ApplicationContext ctx = 
      new AnnotationConfigApplicationContext(TextEditorConfig.class);

      TextEditor te = ctx.getBean(TextEditor.class);

      te.spellCheck();
   }
}
```

**@Import**

**@import** 注解允许从另一个配置类中加载 @Bean 定义。考虑 ConfigA 类，如下所示：

```java
@Configuration
public class ConfigA {
   @Bean
   public A a() {
      return new A(); 
   }
}
```

你可以在另一个 Bean 声明中导入上述 Bean 声明，如下所示：

```java
@Configuration
@Import(ConfigA.class)
public class ConfigB {
   @Bean
   public B a() {
      return new A(); 
   }
}
```

现在，当实例化上下文时，不需要同时指定 ConfigA.class 和 ConfigB.class，只有 ConfigB 类需要提供，如下所示：

```java
public static void main(String[] args) {
   ApplicationContext ctx = 
   new AnnotationConfigApplicationContext(ConfigB.class);
   // now both beans A and B will be available...
   A a = ctx.getBean(A.class);
   B b = ctx.getBean(B.class);
}
```

还可以在注解中指定bean的初始和销毁方法、生存周期：

```java
@Bean(initMethod = "init", destroyMethod = "cleanup" )

@Scope("prototype")
```

### spring中的事件处理

spring的和兴时ApplicationContext，负责管理beans的完整生命周期。当加载beans时，ApplicationContext发布某些类型的事件。如：当上下文启动时，ContextStartedEvent发布。

通过ApplicationEvent类和ApplicationListener接口来提供在ApplicationContext中处理事件。如果一个bean实现ApplicationListerner，那么每次ApplicationEvent被发布到ApplicationContext上，那个bean都会被通知。

##### spring内置事件：

- ContextRefreshedEvent

ApplicationContext被初始化或刷新时被触发。也可以在ConfigurableApplicationContext接口中使用refresh()方法来触发。

- ContextStartedEvent

当使用ConfigurableApplicationContext接口中的start()方法启动ApplicationContext时触发。可以用来启动数据库，或重启任何停止的应用程序

- ContextStoppedEvent

当使用ConfigurableApplicationContext接口中的stop()方法停止ApplicationContext时触发。可以在接收到这个事件后做必要的清理工作。

- ContextClosedEvent

当使用ConfigurableApplicationContext接口中的close()方法关闭ApplicationContext时触发。一个已关闭的上下文就到达了生命周期的尾端。不能被刷新或重启。

- RequestHandleEvent

这是一个web-specific事件，告诉所有的bean：HTTP请求已经被服务。

##### 监听上下文事件：

为了监听上下文事件，一个bean中应该实现只有一个**onApplicationEvent()**方法的ApplicationListener接口。我们可以看看事件是如何传播的，以及如何用代码来执行基于某些事件所需的任务：

这里是 **HelloWorld.java** 文件的内容：

```java
package com.tutorialspoint;
public class HelloWorld {
   private String message;
   public void setMessage(String message){
      this.message  = message;
   }
   public void getMessage(){
      System.out.println("Your Message : " + message);
   }
}
```

下面是 **CStartEventHandler.java** 文件的内容：继承监听接口并指定监听事件

```java
package com.tutorialspoint;
import org.springframework.context.ApplicationListener;
import org.springframework.context.event.ContextStartedEvent;
public class CStartEventHandler 
   implements ApplicationListener<ContextStartedEvent>{
   public void onApplicationEvent(ContextStartedEvent event) {
      System.out.println("ContextStartedEvent Received");
   }
}
```

下面是 **CStopEventHandler.java** 文件的内容：

```java
package com.tutorialspoint;
import org.springframework.context.ApplicationListener;
import org.springframework.context.event.ContextStoppedEvent;
public class CStopEventHandler 
   implements ApplicationListener<ContextStoppedEvent>{
   public void onApplicationEvent(ContextStoppedEvent event) {
      System.out.println("ContextStoppedEvent Received");
   }
}
```

下面是 **MainApp.java** 文件的内容：手动触发事件

```java
package com.tutorialspoint;

import org.springframework.context.ConfigurableApplicationContext;
import org.springframework.context.support.ClassPathXmlApplicationContext;

public class MainApp {
   public static void main(String[] args) {
      ConfigurableApplicationContext context = 
      new ClassPathXmlApplicationContext("Beans.xml");

      // Let us raise a start event.
      context.start();

      HelloWorld obj = (HelloWorld) context.getBean("helloWorld");

      obj.getMessage();

      // Let us raise a stop event.
      context.stop();
   }
}
```

下面是配置文件 **Beans.xml** 文件：将监听器添加到bean里

```xml
<?xml version="1.0" encoding="UTF-8"?>

<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://www.springframework.org/schema/beans
    http://www.springframework.org/schema/beans/spring-beans-3.0.xsd">

   <bean id="helloWorld" class="com.tutorialspoint.HelloWorld">
      <property name="message" value="Hello World!"/>
   </bean>

   <bean id="cStartEventHandler" 
         class="com.tutorialspoint.CStartEventHandler"/>

   <bean id="cStopEventHandler" 
         class="com.tutorialspoint.CStopEventHandler"/>

</beans>
```

运行结果：

```xml
ContextStartedEvent Received
Your Message : Hello World!
ContextStoppedEvent Received
```

##### 自定义事件

步骤：

1. 继承ApplicationEvent事件类，子类必须定义一个默认的构造函数，并在构造函数里执行父类的构造函数
2. 实现ApplicationEventPublisherAware发布事件，并在xml中为这个类定义一个bean。实现了接口才能将bean识别为事件发布者。
3. 实现ApplicationListener监听接口来处理事件，实现接口中的onApplicationEvent方法

这个是 **CustomEvent.java** 文件的内容：

```java
package com.tutorialspoint;
import org.springframework.context.ApplicationEvent;
public class CustomEvent extends ApplicationEvent{ 
   public CustomEvent(Object source) {
      super(source);
   }
   public String toString(){
      return "My Custom Event";
   }
}
```

下面是 **CustomEventPublisher.java** 文件的内容：

```java
package com.tutorialspoint;
import org.springframework.context.ApplicationEventPublisher;
import org.springframework.context.ApplicationEventPublisherAware;
public class CustomEventPublisher 
   implements ApplicationEventPublisherAware {
   private ApplicationEventPublisher publisher;
   public void setApplicationEventPublisher
              (ApplicationEventPublisher publisher){
      this.publisher = publisher;
   }
   public void publish() {
      CustomEvent ce = new CustomEvent(this);
      publisher.publishEvent(ce);
   }
}
```

下面是 **CustomEventHandler.java** 文件的内容：

```java
package com.tutorialspoint;
import org.springframework.context.ApplicationListener;
public class CustomEventHandler 
   implements ApplicationListener<CustomEvent>{
   public void onApplicationEvent(CustomEvent event) {
      System.out.println(event.toString());
   }
}
```

下面是 **MainApp.java** 文件的内容：

```java
package com.tutorialspoint;
import org.springframework.context.ConfigurableApplicationContext;
import org.springframework.context.support.ClassPathXmlApplicationContext;
public class MainApp {
   public static void main(String[] args) {
      ConfigurableApplicationContext context = 
      new ClassPathXmlApplicationContext("Beans.xml");    
      CustomEventPublisher cvp = 
      (CustomEventPublisher) context.getBean("customEventPublisher");
      cvp.publish();  
      cvp.publish();
   }
}
```

下面是配置文件 **Beans.xml**：

```xml
<?xml version="1.0" encoding="UTF-8"?>

<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://www.springframework.org/schema/beans
    http://www.springframework.org/schema/beans/spring-beans-3.0.xsd">

   <bean id="customEventHandler" 
      class="com.tutorialspoint.CustomEventHandler"/>

   <bean id="customEventPublisher" 
      class="com.tutorialspoint.CustomEventPublisher"/>

</beans>
```

一旦你完成了创建源和 bean 的配置文件后，我们就可以运行该应用程序。如果你的应用程序一切都正常，将输出以下信息：

```xml
My Custom Event
My Custom Event
```

## Spring AOP

spring框架的一个关键组件就是AOP（面向方面编程）。把程序罗杰分解成不同的部分，称为关注点，跨一个应用程序的多个点的功能成为横切关注点。这些横切关注点在概念上独立于应用程序的业务逻辑

AOP模块通过拦截器来拦截一个应用程序，再对拦截的程序进行操作。如当执行一个方法时，可以在方法指定之前与之后添加额外的功能。

**AOP术语**

| 项                      | 描述                                                         |
| ----------------------- | ------------------------------------------------------------ |
| Aspect（方面）          | 一个模块具有一组提供横切需求的 APIs。例如，一个日志模块为了记录日志将被 AOP 方面调用。应用程序可以拥有任意数量的方面，这取决于需求。 |
| Join point（加入点）    | 在你的应用程序中它代表一个点，你可以在插件 AOP 方面。你也能说，它是在实际的应用程序中，其中一个操作将使用 Spring AOP 框架。 |
| Advice（建议）          | 这是实际行动之前或之后执行的方法。这是在程序执行期间通过 Spring AOP 框架实际被调用的代码。 |
| Pointcut（切点）        | 这是一组一个或多个连接点，通知应该被执行。你可以使用表达式或模式指定切入点正如我们将在 AOP 的例子中看到的。 |
| Introduction（引用）    | 引用允许你添加新方法或属性到现有的类中。                     |
| Target object（目标类） | 被一个或者多个方面所通知的对象，这个对象永远是一个被代理对象。也称为被通知对象。 |
| Weaving                 | Weaving 把方面连接到其它的应用程序类型或者对象上，并创建一个被通知的对象。这些可以在编译时，类加载时和运行时完成。 |

##### 通知类型

Spring 方面可以使用下面提到的五种通知工作：

| 通知           | 描述                                                         |
| -------------- | ------------------------------------------------------------ |
| 前置通知       | 在一个方法执行之前，执行通知。                               |
| 后置通知       | 在一个方法执行之后，不考虑其结果，执行通知。                 |
| 返回后通知     | 在一个方法执行之后，只有在方法成功完成时，才能执行通知。     |
| 抛出异常后通知 | 在一个方法执行之后，只有在方法退出抛出异常时，才能执行通知。 |
| 环绕通知       | 在建议方法调用之前和之后，执行通知。                         |

#### 自定义方面

支持**@AspectJ annotation style**的方法和**基于模式**的方法来实现自定义方面

---

**基于xml的自定义方面：**

AOP所需jar包：

- aspectjrt.jar
- aspectjweaver.jar
- aspectj.jar
- aopalliance.jar

xml配置：

在xml添加：

```xml
xmlns:aop="http://www.springframework.org/schema/aop"
```

来使用AOP

**步骤**

1. 声明一个切面：使用元素声明，支持的bean通过ref引用

```xml
<aop:config>
   <aop:aspect id="myAspect" ref="aBean">
   ...
   </aop:aspect>
</aop:config>
```

2. 声明一个切入点：

```XML
<aop:config>
   <aop:aspect id="myAspect" ref="aBean">
       <aop:pointcut id="service"
          <!-- 返回值类型 路径.包名.类名.方法名(参数) -->
          expression="execution(* com.xyz.myapp.service.*.*(..))"/>
       ...
   </aop:aspect>
</aop:config>
```

3. 声明建议：

自定义一个建议类，实现需要的接口

![spring5](F:\Tencent\文档\spring5.png)

在aop配置中加入切点和建议

假设在logging文件中定义了各种通知

```xml
<aop:config>
   <aop:aspect id="log" ref="logging">
      <aop:pointcut id="getName" 
      expression="execution(* com.tutorialspoint.Student.getName(..))"/>
      <aop:before pointcut-ref="getName" method="beforeAdvice"/>
      <aop:after pointcut-ref="getName" method="afterAdvice"/>
   </aop:aspect>
</aop:config>
```

或者：

（logging在bean中已定义）

```xml
<aop:config>
   <aop:aspect id="log" ref="logging">
      <aop:pointcut id="getName" 
      expression="execution(* com.tutorialspoint.Student.getName(..))"/>
      <aop:before pointcut-ref="getName" method="beforeAdvice"/>
      <aop:after pointcut-ref="getName" method="afterAdvice"/>
   </aop:aspect>
</aop:config>
```

**使用注解实现AOP：**

```java
import org.aspectj.lang.annotation.Aspect;
import org.aspectj.lang.annotation.Pointcut;
import org.aspectj.lang.annotation.Before;
import org.aspectj.lang.annotation.After;
import org.aspectj.lang.annotation.AfterThrowing;
import org.aspectj.lang.annotation.AfterReturning;
import org.aspectj.lang.annotation.Around;
@Aspect
public class Logging {
   /** Following is the definition for a pointcut to select
    *  all the methods available. So advice will be called
    *  for all the methods.
    */
   @Pointcut("execution(* com.tutorialspoint.*.*(..))")
   private void selectAll(){}
   /** 
    * This is the method which I would like to execute
    * before a selected method execution.
    */
   @Before("selectAll()")
   public void beforeAdvice(){
      System.out.println("Going to setup student profile.");
   }
   /** 
    * This is the method which I would like to execute
    * after a selected method execution.
    */
   @After("selectAll()")
   public void afterAdvice(){
      System.out.println("Student profile has been setup.");
   }
   /** 
    * This is the method which I would like to execute
    * when any method returns.
    */
   @AfterReturning(pointcut = "selectAll()", returning="retVal")
   public void afterReturningAdvice(Object retVal){
      System.out.println("Returning:" + retVal.toString() );
   }
   /**
    * This is the method which I would like to execute
    * if there is an exception raised by any method.
    */
   @AfterThrowing(pointcut = "selectAll()", throwing = "ex")
   public void AfterThrowingAdvice(IllegalArgumentException ex){
      System.out.println("There has been an exception: " + ex.toString());   
   }  
}
```

xml配置：

```xml
 <aop:aspectj-autoproxy/>
 <bean id="logging" class="com.tutorialspoint.Logging"/> 
```

### 事务

一个数据库事务时一个被是为单一的工作单元的操作序列，这些操作要么完整的执行，要么完全不执行。

事务的四大特性：ACID（原子性、一致性、隔离性、持久性）

**局部事务与全局事务**

局部事务特定于一个单一的事务资源，如一个JDBC连接。而全局事务可以跨多个事务资源，如在一个分布式系统中的事务。

局部事务管理在一个集中的计算环境中是有用的，该计算环境中应用程序组件和资源位于一个单位点，而事务管理只设计到一个运行在一个单一机器中的本地数据管理器。所以局部事务更容易实现。

全局事务管理需要在分布式计算环境中，所有的资源都分布在多个系统中，在这种情况下事务管理需要同时在局部和全局范围内进行。分布式或全局事务跨多个系统执行，它的执行需要全局事务管理系统和所有相关系统的局部数据管理人员之间的协调。

**编程式事务管理与声明式事务管理**

编程式事务管理意味着在编程时管理事务。比较灵活，但难以维护。

声明式事务管理意味着从业务代码中分离事务管理，仅使用注解或xml配置来管理事务。

声明式比编程式更可取，尽管不如编程式灵活。但允许通过代码控制事务。并且作为一种横切关注点，声明式事务管理可以使用AOP进行模块化。Spring支持使用Spring AOP框架的声明式事务管理

**Spring事务抽象**

Spring事务抽象的关键由org.springframework.transaction.PlatformTransactionManager接口定义。事务管理器：

```java

public interface PlatformTransactionManager {

	//根据指定的传播行为，该方法返回当前活动事务或创建一个新的事务

	TransactionStatus getTransaction(TransactionDefinition definition) throws TransactionException;

	//提交给定的事务和关于它的状态

	void commit(TransactionStatus status) throws TransactionException;

	//执行给定事务的回滚

	void rollback(TransactionStatus status) throws TransactionException;

}
```

TransactionDefinition时在Spring中事务支持的核心接口：用来查询事务属性

```java
public interface TransactionDefinition {
    //返回传播行为，Spring提供了与EJB CMT类似的所有的事务传播选项
    int getPropagationBehavior();
    //返回该事务独立于其他事务的工作的程度
    int getIsolationLevel();
    //返回该事务的名称
    String getName();
    //返回以秒为单位的时间限制
    int getTimeout();
    //返回事务是否只读
    boolean isReadOnly();
}
```

下面是隔离级别的可能值:

| 序号 | 隔离 & 描述                                                  |
| ---- | ------------------------------------------------------------ |
| 1    | **TransactionDefinition.ISOLATION_DEFAULT**这是默认的隔离级别。 |
| 2    | **TransactionDefinition.ISOLATION_READ_COMMITTED**表明能够阻止误读；可以发生不可重复读和虚读。 |
| 3    | **TransactionDefinition.ISOLATION_READ_UNCOMMITTED**表明可以发生误读、不可重复读和虚读。 |
| 4    | **TransactionDefinition.ISOLATION_REPEATABLE_READ**表明能够阻止误读和不可重复读；可以发生虚读。 |
| 5    | **TransactionDefinition.ISOLATION_SERIALIZABLE**表明能够阻止误读、不可重复读和虚读。 |

下面是传播类型的可能值:

| 序号 | 传播 & 描述                                                  |
| ---- | ------------------------------------------------------------ |
| 1    | **TransactionDefinition.PROPAGATION_MANDATORY**支持当前事务；如果不存在当前事务，则抛出一个异常。 |
| 2    | **TransactionDefinition.PROPAGATION_NESTED**如果存在当前事务，则在一个嵌套的事务中执行。 |
| 3    | **TransactionDefinition.PROPAGATION_NEVER**不支持当前事务；如果存在当前事务，则抛出一个异常。 |
| 4    | **TransactionDefinition.PROPAGATION_NOT_SUPPORTED**不支持当前事务；而总是执行非事务性。 |
| 5    | **TransactionDefinition.PROPAGATION_REQUIRED**支持当前事务；如果不存在事务，则创建一个新的事务。 |
| 6    | **TransactionDefinition.PROPAGATION_REQUIRES_NEW**创建一个新事务，如果存在一个事务，则把当前事务挂起。 |
| 7    | **TransactionDefinition.PROPAGATION_SUPPORTS**支持当前事务；如果不存在，则执行非事务性。 |
| 8    | **TransactionDefinition.TIMEOUT_DEFAULT**使用默认超时的底层事务系统，或者如果不支持超时则没有。 |

TransactionStatus接口为事务代码提供了一个简单地方法来控制事务的执行和查询事务的状态

```java
public interface TransactionStatus extends SavepointManager {
    //是否是新事务
    boolean isNewTransaction();
    //事务内部是否由保存点，也就是说，基于一个保存点已经创建了嵌套事务
    boolean hasSavepoint();
    //将事务设置为rollback-only
    void setRollbackOnly();
    //是否为rollback-only
    boolean isRollbackOnly();
    //事务是否完成（是否已经提交或回滚）
    boolean isCompleted();
}
```

编程式事务管理：https://www.w3cschool.cn/wkspring/urw31mme.html

声明式事务管理：https://www.w3cschool.cn/wkspring/jcny1mmg.html



