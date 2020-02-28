# MVC示例

不要建maven项目！！！

不要建maven项目！！！

不要建maven项目！！！

新建spring项目，勾选mvc

新建后目录不可更改。

可以添加maven框架

这里是 **HelloController.java** 文件的内容：

```java
package com.tutorialspoint;
import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RequestMethod;
import org.springframework.ui.ModelMap;
@Controller
//访问路径
@RequestMapping("/hello")
public class HelloController{ 
    //处理get请求
   @RequestMapping(method = RequestMethod.GET)
   public String printHello(ModelMap model) {
       //将数据存入map，就可以在要返回的文件里获取了
       //视图解析器将model中每个元素都通过request.setAttribute(name, value)添加到request请求域中，这样就可以在jsp页面中通过EL表达式来获取对应的值
      model.addAttribute("message", "Hello Spring MVC Framework!");
       //返回一个视图，将视图呈现出来
      return "hello";
   }
}
```

下面是 Spring Web 配置文件 **web.xml** 的内容

```xml
<web-app id="WebApp_ID" version="2.4"
   xmlns="http://java.sun.com/xml/ns/j2ee" 
   xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
   xsi:schemaLocation="http://java.sun.com/xml/ns/j2ee 
   http://java.sun.com/xml/ns/j2ee/web-app_2_4.xsd">

   <display-name>Spring MVC Application</display-name>

    <!--
   <context-param>
        <param-name>contextConfigLocation</param-name>
        <param-value>/WEB-INF/applicationContext.xml</param-value>
   </context-param>
   <listener>
        <listener-class>
            org.springframework.web.context.ContextLoaderListener
        </listener-class>
   </listener>
    -->
    
   <servlet>
      <servlet-name>HelloWeb</servlet-name>
       <!-- 翻译：调度servlet -->
      <servlet-class>
         org.springframework.web.servlet.DispatcherServlet
      </servlet-class>
      <load-on-startup>1</load-on-startup>
   </servlet>

   <servlet-mapping>
      <servlet-name>HelloWeb</servlet-name>
      <url-pattern>/</url-pattern>
   </servlet-mapping>

</web-app>
```

下面是另一个 Spring Web 配置文件 **HelloWeb-servlet.xml** 的内容

```xml
<beans xmlns="http://www.springframework.org/schema/beans"
   xmlns:context="http://www.springframework.org/schema/context"
   xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
   xsi:schemaLocation="
   http://www.springframework.org/schema/beans     
   http://www.springframework.org/schema/beans/spring-beans-3.0.xsd
   http://www.springframework.org/schema/context 
   http://www.springframework.org/schema/context/spring-context-3.0.xsd">

   <!-- 启动注解扫描，指定要扫描的空间，不能用通配符，指定父包就好 -->
   <context:component-scan base-package="com.tutorialspoint" />

   <mvc:annotation-driven/>
    
    <!-- 内部资源视图解释器 -->
   <bean class="org.springframework.web.servlet.view.InternalResourceViewResolver">
      <property name="prefix" value="/WEB-INF/jsp/" />
      <property name="suffix" value=".jsp" />
   </bean>

</beans>
```

下面是 Spring 视图文件 **hello.jsp** 的内容

```jsp
<%@ page contentType="text/html; charset=UTF-8" %>
<html>
<head>
<title>Hello World</title>
</head>
<body>
    <%!-- 获取servlet传递过来的信息 --%>
   <h2>${message}</h2>
</body>
</html>
```

访问**http://localhost:8080/hello**来获取结果





**ModelAndView:**

可以用来指定向哪个文件传递参数。

mav = new ModelAndView("hello")//指定hello.jsp

mav.addObject("time", new Date());//自带的添加方法

mav.getModel().put("name", "sure");//继承自Model的方法，两种方法都可以向request中添加

new ModelAndView("hello", "name", "sure");//使用构造函数添加



JSP中：

time:${requestScope.time}

name:${name}





**访问静态页面**

由于Spring MVC拦截了所有请求，访问静态资源文件时也会被DispatcherServlet拦截，而且会进行一系列复杂的处理，所以对静态资源必须进行特殊的配置

方法1：

采用**<mvc:default-servlet-handler />**

配置<mvc:default-servlet-handler />后，会在Spring MVC上下文中定义一个org.springframework.web.servlet.resource.DefaultServletHttpRequestHandler，它会像一个检查员，对进入DispatcherServlet的URL进行筛查，如果发现是静态资源的请求，就将该请求转由Web应用服务器默认的Servlet处理，如果不是静态资源的请求，才由DispatcherServlet继续处理。

注：一般Web应用服务器默认端Servlet名称是”default“，因此DefaultServletHttpRequestHandler可以找到它。如果你的Web应用服务器默认的Servlet名称不是“default”，则需要通过

```xml
<mvc:default-servlet-handler default-servlet-name="所使用的Web服务器默认使用的Servlet名称" />
```

方法2：

采用**<mvc:resources />**

<mvc:default-servlet-handler />将静态资源的处理经由Spring MVC框架交回Web应用服务器处理。而<mvc:resources />更进一步，由Spring MVC框架自己处理静态资源，并添加一些有用的附加值功能。

```xml
<mvc:resources mapping="/javascript/**"  location="/static_resources/javascript/"/>  
<mvc:resources mapping="/styles/**"  location="/static_resources/css/"/>  
<mvc:resources mapping="/images/**"  location="/static_resources/images/"/>
```

<mvc:resources />允许静态资源放在任何地方，如WEB-INF下，类路径下，甚至可以将JS等静态文件打包到jar，通过location属性指定静态资源的位置。由于location属性是Resources类型，因此可以使用诸如"classpath:"等的资源前缀指定资源位置。传统Web容器的静态资源只能放在web容器的根路径下，<mvc:resources />打破了这个限制。

location指定资源的源路径

mapping指定映射位置

```xml
<mvc:resources location="/,classpath:/META-INF/publicResources/" mapping="/resources/**"/>
```

以上配置将Web根路径"/"及类路径下 /META-INF/publicResources/ 的目录映射为/resources路径。假设Web根路径下拥有images、js这两个资源目录,在images下面有bg.gif图片，在js下面有test.js文件，则可以通过 /resources/images/bg.gif 和 /resources/js/test.js 访问这二个静态资源。

假设WebRoot还拥有images/bg1.gif 及 js/test1.js，则也可以在网页中通过 /resources/images/bg1.gif 及 /resources/js/test1.js 进行引用。