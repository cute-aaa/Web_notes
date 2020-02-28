![](F:\素材\Images\Ps\servletGetPath.png)

```String getRequestedSessionID()```	

> 返回客户端指定的 Session ID，这可能与当前有效的session不同，如果没有指定则返回null

```String getRequestURL()```

> 返回从HTTP请求到查询字符串或HTTP协议之间的字符串：
>
> > 如post /some/path.html HTTP/1.1	返回/some/path.html
> >
> > get http://foo.bar/a.html HTTP/1.0	返回/a.html
> >
> > head /xyz?a=b HTTP/1.1	返回/xyz

> ```StringBuffer getRequestURL()```	重新构造客户端用于发出请求的URL，返回的URL包含协议、服务器名称、端口号、服务器路径，但不包含查询字符串参数。
>
> ```java
> String getServletPath()
> ```
>
> 返回请求URL中调用servlet的部分（在web.xml的\<url-pattern\>中定义的路径，以/开头，包含servlet名称和路径，但不包含额外的路径信息和查询字符串。与CGI变量SCRIPT_NAME的值相同。如果使用/*匹配用于处理此请求的servlet，则返回“ ”
>
> ```java
> HttpSession getSession(boolean create)
> ```
>
> 返回与此请求关联的当前Session，如果没有，且create的值时true，则返回一个新session，如果没有有效的session且create是false，则返回null.
>
> 为了确保会话得到正确维护,必须在提交响应之前调用此方法,如果容器使用cookie来维护会话完整性且再提交响应时要求创建一个新会话,则会抛出IllegalStateException
>
> *注意：要使用getSession()或getSession(true)来创建对象，使用getSession(false)来获取对象。*
>
> ```java
> HttpSession getSession()
> ```
>
> 返回与此请求关联的httpSession,如果没有则创建，与getSession(true)相同
>
> ```java
> boolean isRequestedSessionIdValid()
> ```
>
> 检查当前请求的session ID是否在当前session环境中依然有效.
>
> ```java
> boolean isRequestedSessionIdFromCookie()
> ```
>
> 检查请求中的session ID是否作为cookie(和URL)进入.
>
> ```getRemoteAddr()```
>
> 返回客户机用于发送请求的IP地址；如：192.168.18.10
>
> ```java
> getRemoteHost()
> ```
>
> 返回发出请求的客户机的主机名，如果servlet引擎不能解析则返回客户机IP
>
>  ```java
> getRemotePort()
>  ```
>
> 返回客户机所使用的网络接口的端口号（这个值是由客户机的网络接口随机分配的）
>
> ```java
> getLocalAddr()
> ```
>
> 返回Web服务器上接收请求的网络接口使用的IP地址，如：192.168.18.254（位于服务器端）
>
> ```java
> getLocalName()
> ```
>
> 返回Web服务器上接受请求的网络接口使用的IP地址所对应的主机名，如：webserver
>
> ```java
> getLocalPort()
> ```
>
> 返回服务器上接受请求的网络接口的端口号，如：8080
>
> ```java
> getServerName()
> ```
>
> 返回Http请求消息的Host字段的值的主机名部分，如：localhost
>
> ```java
> getServetPort()
> ```
>
> 返回Http请求消息的Host字段的值的端口号部分，如：8080
>
> ```java
> getScheme()
> ```
>
> 返回请求的协议名，如：http、https
>
> ```java
> getRequestURL()
> ```
>
> 返回完整的请求URL（不包括参数部分），返回StringBuffer。

## Cookie操作

注意：Cookie中只能保存ISO-8859-1编码支持的字符，如果要保存更复杂的数据就要进行编码，一般将复杂数据以Base64格式编码，也也可以使用java.net.URLEncoder和URLDecoder编码解码。

#### Cookie创建方式：

1. 用Cookie类的构造函数创造一个包含保存信息的Cookie对象

2. setMaxAge设置Cookie对象在客户端的保存时间，setPath设置客户端访问什么路径传递Cookie对象。临时Cookie不用设置MaxAge属性。

3. resp.addCookie将Cookie对象传递给客户端。

#### Cookie删除：

用setMaxAge设置生命周期为0即可

```java
public class Cookie{
    public Cookie(String name, String value);	//构造方法，name不能包含空格、逗号、分号，且不能以$开头。
    getName()	//返回Cookie名称
    get/setValue()
    get/setMaxAge()	
    //返回Cookie在客户机的有效时间（秒），如果为0则发送到客户端时立即删除，设置负数则不会保存在硬盘上而是作为临时Cookie
    临时Cookie：只存在于当前浏览器的进程中，当浏览器关闭时Cookie自动失效。对于IE，不同窗口不能共享临时Cookie。对于FireFox，所有的进程和标签页都共享临时Cookie
    使用Ctrl+N或使用JavaScript的Windows.open语句打开的窗口和他们的父窗口属于同一个浏览器进程。
    get/setPath()	//返回当前Cookie的有效web路径。如果创建Cookie时未设置它的path属性则该Cookie只对当前访问的Servlet所在的Web路径及其子路径有效，如果想要对站点中所有可访问路径都有效就把path设置/
    get/setDomain()	//返回当前Cookie的有效域，如：www.foo.com
    get/setComment()	//返回当前Cookie的注释
    get/setVersion()	//返回当前Cookie协议版本
    get/setSecure()		//返回当前Cookie是否只能使用安全的协议传输Cookie。
}
```

##### 设置Cookie：

```java
//创建临时Cookie（不设置MaxAge就可以）
Cookie cookie = new Cookie("name", value);
resp.addCookie(cookie);
//有效期的Cookie
上述代码中，再添加：
setMaxAge(60*60*24);
setPath("/");
```

##### 通过name查找Cookie

``` java
Cookie[] cookies = request.getCookies();	// Cookie[] getCookies()
if(cookies != null){
    for(Cookie c : cookies){
        if("name".equals(c.getName()))return c;
    }
}else{
    return null;
}
```

##### 将复杂Cookie编码

获取MyCookie类的字节码数组，人后通过编码解析成字符串classStr，最后通过Cookie将classStr保存到客户端。

```java
//创建一个用于进行编码的对象。
sun.misc.BASE64Encoder base64Encoder = new sun.misc.BASE64.Encoder();
//创建一个用于接收被序列化对象实例字节流的对象
ByteArrayOutputStream classBytes = new ByteArrayOutputStream();
//创建一个用于向流中写入对象的对象
ObjectOutputStream oos = new ObjectOutputStream(classBytes);	//将字节写入路径指向classBytes
//写入对象实例
oos.writeObject(new MyCookie());//将MyCookie写入到classBytes
//关闭输出流
oos.close();
//将被序列化的对象实例的字节流按Base64编码格式进行编码
String classStr = base64Encoder.encode(classBytes.toByteArray());//cb现在有值
Cookie cookie = new Cookie("myCookie", classStr);//解析为字符串
//将编码写入Cookie
cookie.setMaxAge(60*60*24);	//一天
resp.addCookie(cookie);
```

MyCookie类：由于MyCookie类需要被序列化，因此它必须实现Serializable接口，否则会抛出java.io.NotSerializableException异常。

```java
public class MyCookie implements java.io.Serializable{
    //获取字符串
    public String getMsg(){
        return "MyCookie对象实例是从Cookie获得的";
    }
}
```

##### 将复杂Cookie解码

读取MyCookie对象实例，并调用其中的getMsg方法：

```java
//创建用于进行解码的对象
sun.misc.BASE64Decoder base64Decoder = new sun.misc.BASE64Decoder();
//通过调用getCookieValue方法获得名为name的Cookie
Cookie cookie = getCookieValue(request.getCookies(), "name");//假设有这么一个查询方法。
if(cookie == null)return;
//取mycookir的值，也就是被序列化的MyCookie类
String classStr = cookie.getValue();
//对classStr字符串进行解码
byte[] classBytes = base64Decoder.decodeBuffer(classStr);
//根据序列化的字节流创建ObjectInputStream对象
ObjectInputStream ois = new ObjectInputStream(new ByteArrayInputStream(classBytes));
//得到MyCookie对象实例
MyCookie myCookie = (MyCookie)ois.readObject();
out.println(myCookie.getMsg);
```

### 处理Session

Cookie的缺陷：超过或接近4KB时会由于在服务端和客户端之间频繁传送Cookie信息二浪费大量带宽，或者是数据结构复杂，Cookie就显得力不从心。

在服务端的开发方案中有一种将大量数据保存在服务端的技术，并使用SessionID（session的唯一标识）对这些数据进行跟踪，这就是Session技术。Session并不会随着浏览器的关闭而消失，而是由服务器端控制。客户端只能获取而不能创建。

> 一个请求只能属于一个Session，但一个Session一个拥有多个请求

> 浏览器第一次访问服务器时会在服务器端生成一个session，有一个SessionID与它对应，Tomcat生成的叫jsessionID。session在getSession(true)时创建，同时服务器会为该Session生成唯一的SessionID（浏览器发送请求时就看请求里有没有SessionID，再看服务器里有没有对应的ID），这个ID在随后的请求中会被用来重新获得已经创建的Session。
>
> 创建Session之后，就可以调用Session相关的方法往Session里增加内容了。但这些内容指挥保存在服务器中，发到客户端的只有SessionID。当客户端再次发送请求时，会将这个SessionID带上服务器接收到请求之后就会根据ID找到对应的Session，从而再次使用。
>
> tomcat的ManagerBase类提供创建SessionID的方法：随机数＋时间+jvmID；
>
> Tomcat的StandardManager类将Session存储在内存中，也可以持久化到file，database，memcache，redis等。
>
> 客户端只保存SessionID到Cookie中，而不会保存Session。
>
> 销毁Session只能通过invaliddate或超时，关闭浏览器不会关闭Session

在Servlet API中，使用javax.servlet.http.HttpSession接口来封装Session信息，主要方法：

```java
long getCreationTime();//返回session的创建GMT时间，毫秒。
String getId();//返回一个分配给此会话的唯一标识符的字符串。标识符由servlet容器分配且依赖于实现。
long getLastAccessedTime();//返回客户端总后一次发送与此会话关联的请求的时间，GMT毫秒
ServletContext getServletContext();//返回此会话所属的servlet上下文，返回一个web应用程序的ServletContext对象。
int get/
void setMaxInactiveInterval(int interval);//指定会话维持时间（秒）。负数表示会话永不应超时。
//HttpSessionContext getSessionContext();//弃用，在将来会被删除。
Object getAttrubute(String name);//	返回name指定的对象。没有则返回null
Enumeration getAttrubuteNames();//返回所有绑定此会话的字符串对象，以枚举存储。
void setAttribute(String name, Object value);//使用指定的名称将对象绑定到此会话，同名对象会被替换。	执行此方法后，如果新对象实现了HttpSessionBindingListener接口，容器将会调用HttpSessionBindingListener.value.Bound，然后容器通知web应用中的所有HttpSessionBindingListener。		如果一个对象已经绑定了实现了HttpSessionBindingListener的同名会话，它的valueBound也会被调用。	如果传入的值为空，这个就跟removeAttribute()有同样的效果。
void removeAttribute(String name);	//从会话中删除指定名称的的对象，如果没有对应名称则什么也不干。执行此方法后，如果新对象实现了HttpSessionBindingListener接口，容器将会调用HttpSessionBindingListener.value.Bound，然后容器通知web应用中的所有HttpSessionBindingListener。
void invalidate();	//使此会话失效，并解除与对象的绑定
boolean isNew();	//当服务端创建了session，但客户端没有加入，则返回true。如果客户端还不知道session（if the client does not yet know about the session），或客户端选择不加入session（如服务器只使用基于cookie的session，而客户端禁用了cookie，这时每请求一次都将新建一个session。

```

##### 创建Session的实例

```java

```

![1541153755840](C:\Users\I\AppData\Roaming\Typora\typora-user-images\1541153755840.png)

![1541153825817](C:\Users\I\AppData\Roaming\Typora\typora-user-images\1541153825817.png)

![1541153881599](C:\Users\I\AppData\Roaming\Typora\typora-user-images\1541153881599.png)

##### 但如果，浏览器关闭了Cookie...

比如在IE里可以在 工具-》Internet选项-》隐私-》高级-》高级隐私策略设置-》覆盖自动Cookie处理-》两个拒绝。

**注意：** 在IE中，如果使用localhost访问Servlet，即使关闭了Cookie功能依然会通过请求消息头的Cookie字段来传递SessionID。

就无法使用Cookie保存和传递SessionID，这时，我们可以改用URL（有种把这个也关了啊）

方式：URL重写

1. 通过encodeURL方法，对所有内嵌在Servlet中的URL进行重写
2. encodeRedirectURL方法，对sendRedirect方法所使用的URL进行重写

**步骤 **：如果Http请求消息头中没有Cookie字段，或是Cookie字段中没有名为JSessionID的Cookie，这两个方法就会在调用getSession方法获得SessionID后，将SessionID写到当前请求的URL后面

1. 包含了SessionServlet类，并调用了encodeURL来重写URL：

```java
//获得指向SessionServlet的RequestDispatcher对象
RequestDispatcher sessionServlet = getServletContext().getRequestDispatcher("/servlet/SessionServlet");
//开始包含SessionServlet
sessionServlet.include(req, resp);
//向客户端输出被重写的URL
out.println("<a href='" + resp.encodeURL("SessionServlet") + "'>SessionServlet</a>");
```

在浏览器输入URL，如：http://localhost:8080/servlet/Test

单击上面的链接之后，在IE中将跳转到指定的Servlet，UR了则变成：

...../servlet/Test;jessionid=46846446F54446D1E1151113A5645CB8466

有了这个URL，在会话有效期间，在其他浏览器上输入这个URL将会显示同样的结果。

### 注意：

> ServletRequest、HttpSession、ServletContext对象都可以存储对象，但：
>
> ServletRequest对象存储的对象只能被当前请求的Servlet访问，HttpSession对象存储对象可以被当前会话中所有的Servlet访问，而ServletContext对象存储的对象可以被所有的Servlet访问和共享。

> 对于乱码：
>
> Java文件编码格式一般时UTF-8，但Java在编译源代码时，会将其转换成UCS2编码。由于UCS2支持世界上所有的语言，因此Java源代码中的元素（包括变量名、变量值、类名、方法名等）可以使用任何一个国家的语言来编写。UCS2编码中所有的字符都是由两个字节组成（英文字母也是），UTF-8是USC2的优化，将单字节字符以一个字节表示，中文则使用三个字节表示。
>
> 如果web项目乱码，一般使用：
>
> resp.setContentType("text/html; charset=UTF-8");
>
> 同时设置了服务器转换字符和客户端显示字符的编解码方法。
>
> 如果不支持这么好的方法：
>
> //获得UTF-8编码的字节数组后，将其按原样保存在String对象中

#### JSP技术

当页面中的静态内容多于动态内容时，由于静态内容在Servlet中书写不方便，就有了JSP技术，jsp中的动态内容将会交给Servlet引擎，生成对应的java文件和class文件。静态页面使用write方法输出到客户端（write方法只能输出字符串、字符数组、int型的数据，不过静态内容都是以字符串形式保存在jsp文件中，所以足够用了）。

1. <%...%>：这种形式按照原样插入到由jsp生成的Servlet源代码中。每段可以不是完整的代码，但整体必须完整符合Java代码。且：内容会被照原样插入到_jspService方法中，所以，如果在<%...%>中定义了方法或其他不能再Java方法中出现的语法，再翻译jsp时就会抛出异常。，如果要声明方法要使用<%!...%>。支持注释。

2. <%=...%>：并不直接把内容放到Servlet源程序中，而是通过print方法将=后面的内容输出到客户端。不能使用分号

3. <%@ page ...%>：jsp的page指令，jsp引擎按照指令类型和属性翻译成相应的Java代码<br>共有三种指令：<br>1. **page**: 定义页面的依赖属性，如脚本语言、error页面、缓存需求等<br>                   属性：<br>                    buffer  指定out对象使用缓冲区的大小<br>                    autoFlush  控制out对象的缓存区<br>                    contextType  指定当前JSP页面的MIME类型 和字符编码<br>                    errorPage  指定当JSP页面发生异常时需要转向的错误处理页面<br>                    isErrorPage  指定当前页面是否可以作为另一个JSP页面的错误处理页面（注意：不                                                     是查询，而是指定）<br>                    extends  指定servlet从那个类继承<br>                    import  导入Java类<br>                    info  定义JSP页面的描述信息<br>                    isThreadSafe  指定对JSP页面的访问是否为线程安全<br>                    language  定义JSP页面用的脚本语言，默认是Java<br>                    session  指定JSP页面是否使用session<br>                    isELlgnored  指定JSP页面是否执行EL表达式<br>                    isScriptingEnabled  指定是否使用脚本<br>2. **include**: 包含其他文件（在JSP文件被转换成Servlet的时候），可以是JSP，HTML或文本。被包含的文件就像是该JSP文件的一部分，会被同时编译执行。如果没有给关联文件路径，则默认当前路径<br>                    语法：<br>                     <%@ include file = "文件相对URL地址" %><br>3. **taglib**: 引入标签库的定义，可以是自定义标签（一个自定义标签库就是自定义标签的集合）<br>                     语法：<br>                     <%@ taglib uri = "uri" prefix = "prefixOfTag" %><br>                     uri属性确定标签库的位置，prefix属性指定标签库的前缀<br>

4. <%!...%>：内容会被插入到_jspService方法外，也就是说内容将作为生成的Servlet类的全局内容，所以<%!...%>的内容必须符合Java类的定义规则。如：不能直接使用System.out.println("..."),要将单独的语句放到static{...}中。如：static{sout("...")};方法则可以正常定义。定义变量一般也在这里

5. <%-- ... --%>：注释

6. **jsp行为**：行为标签使用XML语法结构来控制Servlet引擎，与指令元素不同的是，JSP元素在请求处理阶段起作用，他只有一种语法格式：<br><jsp:action_name attribute="value" /><br> 所有的动作都有两个属性：**id属性和scope属性**：<br>        id属性： id是动作元素的唯一标识，可以在JSP页面中引用。动作元素创建的id值可以通过                  PageContext来调用<br>        scope属性：scope属性用于识别动作元素的声明周期，id属性和scope属性有直接关系，scope属性定义了相关联id对象的寿命，scope属性有四个可能的值：page、request、session、application（也就是跟哪个一起创立与消失）<br>action_name参数有：<br>1. jsp:include page = "相对URL路径" flush = "true"  用于在当前页面中包含静态或动态资源。与include指令不同：jsp:include的插入过程是动态的，也就是当JSP页面运行到<jsp:include>指令时才执行插入动作。

    - RequestDispatcher.include、pageContext.include、include指令、<jsp:include>标签的区别：
    - RequestDisPatcher.include和pageContext.include在功能上时完全相同的，后者是为了简化前者。这两个是include方法。include方法、指令、标签的区别是：
    - 1、引入资源的方式：只有include指令时静态引入Web资源的，其他两个都是动态。
    - 2、Http响应头的改变：只有include指令能改变响应头和响应状态码
    - 3、只有include指令的相对路径是相对于文件的，其他两个都是相对于页面的。大概是：假如在web.xml中改了JSP文件的映射路径，就只有include指令管用了，解决方法是将要包含的文件跟jsp放在同一目录下，或直接使用相对于Web根目录的相对路径（也就是路径前加/）
    - 4、include指令在引用时，无论扩展名是不是.jsp，都会按照jsp页面来处理，而其他两个则只处理.jsp文件，其他后缀当作静态文件将文件内容按原样输出。
    - 5、当指向的Web资源不存在时，include指令抛出异常，而其他两个则输出提示后继续执行后面的JSP。

   <br>2. jsp:useBean  class = "package.class" type = "变量类型" beanName = "Bean名"  寻找和初始化一个JavaBean组件，语法：

   ​	<jsp:userBean id = "name" scope = "page | request|session|application" typeSpec>

   		<p>
   		    typeSpec可以用来设置属性
   		</p>
   		这里面可以时任意内容，包括JSP表达式、EL表达式、或是<jsp:serProperty标签
   ​	</jsp:userBean >

   ​	typeSpec :: = 

   ​		class="className" |

   ​		class="className" type ="typeName" |

   ​		beanName = "  beanName | <%= expression %> | EL "type=" type=name   "

   id  表示对象实例名，如 Object obj = new Object()，id就是其中的obj。

   scope  指定有效范围（(默认)page < request < session < application ），如将scope设为

   ​     	session，就意味着所有属于同一个session的JSP页面和Servlet都可以访问该对象实例。		

   ​	示例：如果将scope设为page，则JavaBean创建的对象只对当前页面有效，则每次刷新页

   ​	面都将新建对象。而如果设为session，则新建之后在当前session会话下都不会再新建。也

   ​	就是<jsp:userBean >标签里的内容只输出一次。  

   class  属性指定Bean的完整包名，如java.util.Date

   type  指定将引用该对象变量的类型，即要实例化的父类名（package.classname）

   beanName  与class属性相似，也是要实例化的类名。只是在JSP引擎翻译JSP页面时有所差异。class属性翻译成静态创建对象实例（也就是new）而beanName翻译成动态创建（通过java.beans.Beans的instantiate()方法指定Bean的名字，是个反射）。另外，beanName可以使用JSP表达式（如<%=  %>  和 ${  }）而class的属性值只能是静态字符串。示例：

   ​	1、最简单的：

   ​		<jsp:userBean id = "myDate" class = "java.util.Date">

   ​		相当于：

   ​		<% java.util.Date myDate = new java.util.Date(); %>

   ​	2、使用class和type：

   ​		<jsp:userBean id = "myDate" class = "java.util.Date" type = "Object">

   ​		相当于：

   ​		<% Object myDate = new java.util.Date(); %>

   ​	3、使用beanName和type：

   ​		<jsp:useBean id = "myDate" type = "Object" beanName = "java.util.Date">

   ​		相当于：

   ​		<% Object myDate = (Object)java.beans.Beans.instantiate(this.getClass().getClassLoader("java.util.Date" ); %>

   注：beanName和type必须同时出现，且beanName不能和class同时使用。<br><br>3. jsp:setProperty name = "Bean的id" property = "要设置的属性" value = "" param = ""  设置JavaBean组件的值。property属性匹配*。value<br>4. jsp:getProperty  将JavaBean组件的值插入到output中<br>    Bean结合setP和getP: <br>

   ```
   <jsp:useBean id="id" class="bean 包+类" scope="bean 作用域">
      <jsp:setProperty name="bean 的 id" property="属性名"  
                       value="value"/>
      ...........
   </jsp:useBean>
   <p>
      <jsp:getProperty name="bean 的 id" property="属性名"/>
   </p>
   ```

    <br>5. jsp:forward  从一个jsp文件向另一个文件传递用户请求的request对象（将当前请求转发给其他的静态资源、JSP页面、Servlet），使用方法与forward方法（pageContext.forward和RequestDispatcher.forward）相同。语法：<br><jsp:forward page="relativeURL | <%= expression %>" >

   ​	{ <jsp:param ... /> } *

   </jsp:forward ><br>其中page属性不仅可以是相对路径，也可以是JSP表达式返回的值。<br>注意：

   ​	forward之前会清空out缓冲区

   ​	如果调用之前out缓冲区已经被刷新，则会抛出异常

   ​	当标签前面的输出字符数量（包括静态和out对象的输出）大于缓冲区尺寸会抛异常

   ​	如果当前页未使用out缓冲区，且标签前由任意字符（包括换行）则会抛异常

​	小栗子：

​	假设com.MyClass类里有name和age属性并有对应的get和set，ustBean的id是类的对象名。setP/getP的name也是类的对象名，即给那个对象设置和获取。

```jsp
使用setP和getP设置和获取属性值
<jsp:useBean id = "myclass" class = "com.MyClass" />
相当于：（beanName是new的对象名，type是接收对象的类型，如Object
<jsp:useBean id = "myClass" beanName = "com.MyClass" type = "com.MyClass" />

设置：
<%-- 直接设置属性值 --%>
<jsp:setProperty property = "name" name = "myClass" value = "bill" />
<jsp:setProperty property = "age" name = "myClass: value = "32" />
<%-- 使用URL中的同名参数赋值，URL中必须有name和age --%>
<jsp:setProperty property = "name" name = "myClass" />
<jsp:setProperty property = "age" name = "myClass" />
<%-- 使用不同名的URL参数设置属性值 --%>
<jsp:setProperty property = "name" name = "myClass" param = "newName" />
<jsp:setProperty property = "age" name = "myClass" param = "newParam" />
<%-- 使用URL自动设置所有的属性 --%>
<jsp:setProperty property = "*" name = "myClass" />

获取：
<jsp:getProperty property = "name" name = "myClass" />
<jsp:getProperty property = "age" name = "myClass" />
```

​	从上面可以看出，setP的property是要设置的变量名，name是要给哪个对象设置，value是设			  

​	置的值，param是要获取的URL中的参数的值。<br>6. jsp:plugin  用于生成的HTML页面中包含Applet和JavaBean对象，通过Java插件运行。如果没有插件则会自动下载。

```jsp
<jsp:plugin type="applet" codebase="dirname" code="MyApplet.class"
                           width="60" height="80">
   <jsp:param name="fontcolor" value="red" />
   <jsp:param name="background" value="black" />
 
   <jsp:fallback>
      Unable to initialize Java Plugin
   </jsp:fallback>
 
</jsp:plugin>
```

<br><br>7. jsp:element  动态创建一个XML元素<br>8. jsp:attribute  定义动态创建的XML元素的属性<br>9. jsp:body  定义动态创建的XML元素的主体

```jsp
<%@ page language="java" contentType="text/html; charset=UTF-8"
    pageEncoding="UTF-8"%>
<!DOCTYPE html>
<html>
<head>
<meta charset="utf-8">
<title>菜鸟教程(runoob.com)</title>
</head>
<body>
<jsp:element name="xmlElement">
<jsp:attribute name="xmlElementAttr">
   属性值
</jsp:attribute>
<jsp:body>
   XML 元素的主体
</jsp:body>
</jsp:element>
</body>
</html>
```

运行结果：

![img](http://www.runoob.com/wp-content/uploads/2014/01/7D8C47F0-0DDE-4F1D-8BE1-B2C9C955683E.jpg)

<br>10. jsp:text  用于封装模板数据，允许在JSP和文档中使用写入文本的模板，格式：<br>11. jsp:param  可以向其他标签传递参数，对于jsp:include和jsp:forward传递的是URL参数，如：使用jsp:param将参数添加到jsp:include的URL：<br><jsp:include page="/test/test.jsp?id=1234&age=30" >

   ​	<jsp:param name="name" value = "bill" />

   ​	<jsp:param name  ="age" value = "35" />

   ​	<jsp:param name = "salary" value = "<%=3000 / 1.2 %>" />

   </jsp:include ><br>另一个页面处理传过来的参数：<br><% @page pageEncoding = "UTF-8" %>

   ​	d: ${param.id}

   ​	name: ${param.name}

   ​	age: ${param.age}

   ​	salary: ${param.salary}

   输出：id: 1234   name: bill   age: 30   salary: 2500.0<br>forward也会得到同样的结果，可以看到age并不是param的35，而是URL自带的30，所以URL的值会覆盖<jsp:param >中的同名参数。<br>

7. **jsp内置对象：**jsp支持九个自动定义的变量，江湖人称隐含对象：

   - request  HttpServletRequest类的实例，使用方法与servlet的getXXX对象一样。
   - response  HttpServletResponse类的实例，使用方法与servlet的getXXX对象一样。
   - out  PrintWriter类的实例，最常用的内置对象，用于向客户端输出文本形式的数据，JSP对象实际上是一个JSPWriter对象，有pageContext对象的getOut方法获取。与PrintWriter对象相似，他也有自己的缓冲区，使用out对象向客户端输出数据时，系统会先将数据放在out对象的缓冲区中（如果使用缓冲区的话）直到缓冲区被装满或整个JSP页面结束，这时缓冲区的内容就会被写道Servlet引擎提供的缓冲区中，最后系统将Servlet引擎中的数据输出到客户端。而PrintWriter则直接输出到Servlet缓冲区。<br>假设有以下过程：<br>JSPW输出<br>PW输出<br>页面结束<br>则如果JSPW如果没有将JSPW缓冲区填满，则在页面结束之前不会将内容输出到Servlet缓冲区中，而PW直接输出到Servlet缓冲区，所以最后的输出结果会在JSPW前面。如果JSPW把JSPW缓冲区填满并稍稍超过，则JSPW会先输出到Servlet缓冲区中，然后PW输出到Servlet缓冲区中，但由于只是JSPW稍稍超过，所以超出缓存区大小的内容仍然在JSPW缓冲区中，知道页面结束，输出到Servlet缓冲区，所以结果会是JSPW首先输出的部分、PW部分、JSPW超出缓冲区的部分。（注意：需要先使用out.clear方法将JSPW缓冲区内容清空以免有残留信息。）
   - session  HttpSession类的实例，使用方法与servlet的getXXX对象一样。
   - application  ServletContext类的实例，用于获得和当前Web应用相关的信息。由PageContext类的getServletContext方法返回。可以获得全局的初始化参数、某个Web资源的绝对路径、Servlet引擎的版本号等信息，也可以用于保存和获得全局对象（setAttribute和getAttribute方法）<br>application.getInitParameter(name)获取初始化参数的值<br>application.getRealPath("test.jsp")获取jsp的绝对路径<br>application.getMajorVersion() + "." + application.getMinorVersion()获取Servlet版本号<br>application.getContextPath()获取应用路径<br>
   - config  ServletConfig类的实例，有PageContext类的getServletConfig方法返回。可以获得web.xml文件中与当前JSP页面相关的配置信息。如初始化参数和Servlet名等。例：config.getServletName()可以获得Servlet类名。例：config.getInitParameter("name")输出初始化参数值（<init-param>标签中的<param-name>和<param-value>）。
   - pageContext  javax.servlet.jsp.PageContext类的对象实例，封装了JSP页面的运行信息。可以通过此对象的getXxx方法来获得其他的内置对象。所以当JSP页面调用普通Java类时，秩序将pageContext对象作为参数传入相应方法，就可以获取和操作其他的JSP内置对象了。
   - page  类似于Java中的this关键字，如page.getClass().getName()可以获取当前Servlet名
   - exception  Exception类的对象，代表发生错误的JSP的页面中对应的异常对象，只有page指令的isErrorPage属性为true时才会创建exception对象。可以获得异常的相关信息。

示例：

```
<%= new java.util.Random().nextInt(1000);
相当于：
out.print(new java.util.Random().nextInt(1000));
```

获取Http信息头实例：

```jsp
<%@ page language = "java" contentType = "text/html;charset = UTF-8" pageEncodeing = "UTF-8" %>
<%@ page import = "java.io.*,java.util.*" %>
<!DOCTYPE html>
<html>
    <head>
        <meta charset = "utf-8">
        <title>菜鸟教程</title>
    </head>
    <body>
        <h2>
            HTTP头部请求
        </h2>
        <table width = "100%" border = "1" align = "center">
            <tr bgcolor = "#949494" >
                <th>Head Name</th> <th>Header Value(s)</th>
            </tr>
            <% 
            	Enumeration headerNames = req.getHeaderNames();
            while(headerNames.hasMoreElements()) {
                String paramName = (String)headerNames.nextElement();
                out.print("<tr><td>" + paramName + "</td>\n");
                String paramValue = req.getHeader(paramName);
                out.println("<td>" + paramValue + "</td></tr>\n");
            }
            %>
        </table>
    </body>
</html>
```

运行结果：![img](http://www.runoob.com/wp-content/uploads/2014/01/jspheadmsg.jpg)

### EL表达式（jsp表达式语言。Expression Language）

使用EL可以很方便地访问JSP页面相关的数据和javaBean的属性，基本语法为：${表达式}，由于EL只代表一个值（像<%=...%>那样），因此不仅可以出现在JSP标签的属性值中，也可以出现在JSP页面的模板文本（静态内容区）中。

如：访问名为key的请求参数：

```java
java: 	<%= request.getParameter("key") %>
EL:		${param.key}
```

如：访问JavaBean属性：

```java
java:	<% MyBean mb = (MyBean)pageContext.findAttribute("mb");%>
		<%= mb.getName(); %>
EL:		${mb.name}
```

如果name属性返回了一个对象，这个对象包含两个属性：

```java
java:	<%= mb.getName().getfirstName() %>
    	<%= mb.getName().getlastName() %>
EL:		${mb.name.firstName}
		${mb.name.lastName}
```

常用的param.name：(param时EL内置对象,用于获取请求参数的值，还有cookie对象，直接使用${cookie.name};

```jsp
<!-- el.jsp -->
<%@ page language="java" pageEncoding="UTF-8" %>
<html>
    <head>
        <title>在JSP中使用EL</title>
    </head>
    <body>
        <form method = "post">
            &nbsp; key:
            <input type = "text" name = "key"/>
            <p>
                value:
                <input type = "text" name = "value" />
                <input type = "submit" value = "提交" />
            </p>
        </form>
        &nbsp; key: ${param.key}
        <p>
            value: ${param.value}
            <-- 根据class属性创建一个类的对象，id指定对象名。这里将变量名作为key保存在request中 -->
            <jsp:useBean id = "t" class = "java.util.Date" scope = "request" />
        </p>
    </body>
</html>
```

当EL返回值为NULL时,会输出空串"",而Java会输出"null",第一次访问jsp时,由于还没有提交,上面的param.key就会输出"",而Servlet会输出null

如果EL表达式出错，并不会向Java一样抛出异常，而是向客户端输出一个描述错误的信息，如${5/0}会向客户端输出Infinity信息，表示分母为0，会产生一个无穷大的数，如果使用Java则会抛出ArithmeticException异常。

##### 实例：

利用EL函数替换HTML中的特殊字符。（JSP允许在EL中调用Java类的静态方法，称为EL函数）：

${pkg:fun(a1, a2)}（注释：${包名:函数名(参数1，参数2)}   ）

功能：将一个字符串中的特殊字符（空格、<、>、）转换为网页可以显示的字符（&nbsp；、&lt；、&gt；）

1. 编写Java类的静态方法：

```java
package com;
public class ELFun{
    public static String processStr(String s){
        s = s.replaceAll("<", "&lt;");
        s = s.replaceAll(">", "&gt;");
        s = s.replaceAll(" ", "&nsbp;");
        return s;
    }
}
```

2. 在WEB-INF目录下建立tld目录，目录下建立一个elfun.tld，编写TLD文件：

   ```xml
   <?xml version="1.0" encoding="UTF-8" ?>
   <taglib xmlns = "http://java.sun.com/xml/ns/javaee"
           xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
           xsi:schemaLocation="http://java.sun.com/xml/ns/javaee/web-jsptaglibrary_2_1.xsd"
           version="2.1">
       <tlib-version>1.0</tlib-version>
       <description>用于转换特殊字符</description>
       <uri>myelfun</uri>	<!-- 定义URI，可以通过uri来引用 -->
       <function>			
           <name>ps</name>	<!-- 定义EL函数名 -->
           <!-- 指定EL函数所在的类 -->
           <function-class>com.ELFun</function-class>
           <function-signature>
               <!-- 指定EL函数名 -->
               java.lang.String processStr(java.lang.String)
           </function-signature>
       </function>
   ```

   3. 修改web.xml文件

      ```xml
      <jsp-config>
          <taglib>
              <taglib-uri>/WEB-INF/tld/</taglib-uri>
          </taglib>
      </jsp-config>
      ```

      4. 调用EL函数，在com目录下建立一个elfun.jsp文件：

         ```jsp
         <%@ page language="java" pageEncoding="UTF-8" %>
         <!-- 声明tld文件 -->
         <%@ taglib uri="/WEB-INF/tld/elfun.tld" prefix="elfun" %>
         <html>
             <head>
                 <title>调用EL函数</title>
             </head>
             <body>
                 <!-- 通过form为EL函数比较一个字符串 -->
                 <form method = "post">
                     请输入一个字符串：
                     <input type = "text" name = "text"/>
                     <input type = "submit" value = "提交" />
                 </form>
                 直接输出文本框的内容：
                 ${param.text}	<!--输出请求参数text的值-->
                 使用定制函数输出文本框中的内容：
                 ${elfun:ps(param.text)}	<!-- tld文件中定义的 -->
             </body>
         </html>
         ```

##### EL函数注意点：

1. EL函数对应的Java类的方法必须是静态的（可以自己定义啊）

2. TLD文件扩展名必须为tld
3. 除了使用TLD文件路径引用tld文件外，还可以使用tld文件中定义的URL引用tld文件。如可以将上面的jsp内容改为：

```xml
<%@ taglib uri = "myelfun" prefix = "elfun" %>
```

其中myelfun是在tld文件中定义的uri，但为了避免与其他tld文件中定义的EL函数产生冲突，这个uri在向外发布时最好使用域名，如：http://www.sun.com/myelfun

4. 如果使用路径引用tld文件，tld文件可以放到web工程的任何目录（包括lib、classes等），而如果使用uri来引用tld文件则必须放到WEB-INF目录中或WEB-INF的子目录（包括lib、和classes）
5. 在使用路径引用tld文件时，在修改调用EL函数的JSP页面后，无需重启服务器或重新发布web工程就能生效，而使用uri则必须重启或重新发布。
6. 这两种方法在本质上是一样的，可以生成相同的Java代码：

```java
static{
    _jspx_dependants = new java.util.ArrayList(1);
    _jspx_dependants.add("/WEB-INF/tld/elfun.tld");
}
```

* 如果使用uri引用tld文件，jsp引擎会先在WEB-INF目录及其子目录下寻找所有的tld文件。如果发现某个tld文件中的<uri>标签定义的uri与taglib中uri属性的值相等，就会记住这个.tld路径，在生成Servlet的同时，就会将这个tld文件的路径也加进来。

* 如果使用路径引用tld文件，jsp引擎会直接将tld文件的路径加入到Servlet源码中。

总结：

​	**uri：** 必须放到WEB-INF目录下，但可以随意改变位置，且无需修改调用EL函数的JSP文件，但需要重启服务器或重新发布。

​	**路径：**可以直接找到tld文件，但需要修改JSP文件中的tld路径。无需重启或重新发布。



### jsp指令

jsp指令是jsp程序的控制部分，如果将JSP指令与由jsp页面生成的Servlet类源代码相对照，相当于Servlet类中的引用部分（如import语句）以及一些设置方法的调用（如setContentType方法），此外通过jsp指令还可以实现包含其他的web资源。



## JSTL

#### taglib指令：

主要用于为JSP代码引入JSTL

<%@ taglib uri = "tagLibURI" prefix = "tagPrefix" %>

其中uri是JSTL标签库的唯一标识，可以是任意字符串。prefix是JSTL标签库的前缀，目的是区分其他同名标签。其实也就是uri是包名，prefix是类名。

**<c:out >**

语法格式1：（通过default属性设置默认值）

```jsp
<c:out value = "String | <%= expression %> | EL "  [escapeXml = "{true | false}"][default = "default"] />
```

语法格式2：（通过标签体设置默认值）

```jsp
<c:out value = "String | <%= expression %> | EL" [escapeXml = "{true | false}"] >
    defalultValue
</c:out>
```

其中escapeXml属性设置是否将输出结果中的特殊字符（如 "<" ">" "&"）转换成相应的字符串以便客户端能正常显示。如果为false则原样输出。默认true

**<c:set >**

语法格式1：

```jsp
<c:set value = "String | <%= expression %> | EL" var = "varName" [scope = "{page|request|session|application}"] />
```

语法格式2：

```jsp
<c:set var = "varName" [scope = "{page|request|session|application}"] >

	String | <%= expression %> |EL

</c:set >
```

小栗子：

```jsp
<%@ page contentType = "text/html; charset = UTF-8" pageEncoding="UTF-8" %>
<!-- 项目右键-》buildPath-》library-》MyEclipse lib-》JSTL
jstl包-》jstl-implxxx.jar-》META-INF-》c.tld-》<uri>http://java.sun.com/jsp/jstl/core</uri>中间就是uri -->
<%@ taglib uri = "http://java.sun.com/jsp/jstl/core" prefix = "c" %>
<c:out value="用c:out输出字符串" />
<br>
<c:set var = "scopeVar1" value = "为变量赋值并保存在request" scope = "request" />
scopeVar1 = <c:out value = "${scopeVar1 }" escapeXml = "true" /><br>
<c:set var = "scopeVar2" scope = "session" >
hello ${param.name }
</c:set>
scopeVar2 = <c:out value = "${scopeVar2 } " />
```

输出结果：

```jsp
用<c:out>输出字符串
scopeVar1 = 为变量赋值，并保存在request
scopeVar2 = hello
```

##### 条件标签

<c:if >语法格式：

```jsp
<c:if test = "Condition" [var = "varName"] [scope = "{page|request|session|application}"]>
    标签体（包括jsp表达式和EL）
</c:if>
```

其中Condition是一个Boolean表达式，包括JSP表达式和EL。

当Condition为true时，会执行标签体部分，var和scope可选但必须同时出现。var将Condition的值保存在scope指定范围的一个变量中，供JSP来提取结果。

<c:choose >、<c:when >、<c:otherwise >：

if只能处理单一的条件，而choose可以处理一个条件组，相当于switch语句。而when相当于case，otherwise就是default了。

示例：

```jsp
<c:if test="true">
	当test为true时处处标签体
</c:if>
<c:set var = "hour" value = "<%= Calendar.getInstance().get(Calendar.HOUR_OF_DAY) %>"></c:set>
<c:if test="${hour <12 }">上午好</c:if>
<c:if test="${hour >= 12 }">下午好</c:if>
<c:choose>
	<c:when test = "${param.grade >= 90 && param.grade <= 100 }" >优秀</c:when>
	<c:when test="${param.grade >= 80 && param.grade < 90 }">良好</c:when>
	<c:when test="${param.grade >= 70 && param.grade <80 }">中等</c:when>
	<c:when test="${param.grade >= 60 && param.grade < 70 }">及格</c:when>
</c:choose>
<!-- 在浏览器中输入...../xxx.jsp?grade=95 -->
```

##### 循环标签

<c:forEach >标签：

可以用来对：

- 数组
- Collection
- Iterator
- Enumeration
- Map
- 逗号（,）分隔的字符串

进行枚举。

格式1：相当于forEach循环：

```jsp
<c:forEach [var = "varName"] items = "collection" [varStatus = "varStatusName"] [begin = "begin"] [end = "end"] [step = "step"]>
    标签体（包括JSP和EL）
</c:forEach>
```

格式2：相当于for循环：

```jsp
<c:forEach [var = "varName"] [varStatus = "varStatusName"] begin = "begin" end = "end" [step = "step"] >
    标签体（包括JSP和EL）
</c:forEach>
```

其中带上item属性之后就相当于有了个迭代器，所以相当于foreach。如果item为空则不进行枚举。

begin和end在存在item时表示枚举集合中从begin到end的元素，不存在item时则表示循环的起始和终止位置。begin默认0，且不能小于0，end默认集合最后一个元素的位置。step表示步长。

其中varStatus的属性值可以代表当前循环到的元素，如：varStatusName.index可以返回当前元素的索引，x.count是计数序号（索引+1），x.first判断此项是否是第一项，x.last判断是否最后一项，x.begin和end和step返回起始和终止索引与步长。

varStatus代表循环状态，var代表当前条目。如跟下标有关的操作就是varStatus，跟当前循环到的变量（不是当前变量的下标）有关的就是var。如：

varStatus = "i" var = "iter" item = "${list}"



| 标签           | 描述                                                      |
| -------------- | --------------------------------------------------------- |
| <c:out >       | 直接把运算结果显示出来                                    |
| <c:set >       | 保存数据                                                  |
| <c:remove >    | 删除数据                                                  |
| <c:catch >     | 处理异常并将错误储存起来                                  |
| <c:if >        | if语句                                                    |
| <c:choose >    | switch                                                    |
| <c:when >      | case                                                      |
| <c:otherwise > | default                                                   |
| <c:import >    | 检索一个相对或绝对URL并将内容暴露给页面                   |
| <c:forEach >   | for和forEach的结合体，item是数组，varStatus是i，var是a[i] |
| <c:forTokens > | 根据指定的分隔符来分隔内容并迭代                          |
| <c:url >       | 使用可选的查询参数来创造一个URL                           |
| <c:param >     | 给包含或重定向的页面传递参数                              |
| <c:redirect >  | 重定向至一个新的URL                                       |

c:import例：

```jsp
<c:import var="data" url="http://www.runoob.com"/>
<c:out value="${data}"/>
```

以上程序将会打印url中网站的源码。

c:redirect例：

```jsp
<c:redirect url="http://www.runoob.com"/>
```

程序将会使页面跳转到指定URL。

c:url例：

```jsp
<c:url var = "myURL" value = "main.jsp" >
    <c:param name = "name" value = "Runoob" />
    <c:param name = "url" value = "runoob.com" />
</c:url>
```

查看源代码，可以发现a标签的内容为：

```html
<a href = "/main.jsp?name=Runoob&url=runoob.com">....</a>
```

#### 格式化标签

用于格式化并输出文本、日期、时间、数字。使用以下语法来引用个格式化标签：

```jsp
<%@ taglib prefix = "fmt" uri = "http://java.sun.com/jsp/jstl/fmt" %>
```

| 标签                   | 描述                                     |
| ---------------------- | ---------------------------------------- |
| <fmt:formatNumber >    | 使用指定的格式或精度格式化数字           |
| <fmt:parseNumber >     | 解析一个代表着数字，货币或百分比的字符串 |
| <fmt:formatDate >      | 使用指定的风格或模式格式化日期和时间     |
| <fmt:parseDate >       | 解析一个代表着日期或时间的字符串         |
| <fmt:bundle >          | 绑定资源                                 |
| <fmt:setLocale >       | 指定地区                                 |
| <fmt:setBundle >       | 绑定资源                                 |
| <fmt:timeZone >        | 指定时区                                 |
| <fmt:setTimeZone >     | 指定时区                                 |
| <fmt:message >         | 显示资源配置文件信息                     |
| <fmt:requestEncoding > | 设置request的字符编码                    |

如：简单的日期格式化：

```jsp
<p>日期格式化 (3): <fmt:formatDate type="both" 
            value="${now}" /></p>
<p>日期格式化 (4): <fmt:formatDate type="both" 
            dateStyle="short" timeStyle="short" 
            value="${now}" /></p>
<p>日期格式化 (5): <fmt:formatDate type="both" 
            dateStyle="medium" timeStyle="medium" 
            value="${now}" /></p>
<p>日期格式化 (6): <fmt:formatDate type="both" 
            dateStyle="long" timeStyle="long" 
            value="${now}" /></p>
```

结果：

```jsp
日期格式化 (3): 2016-6-26 11:19:43

日期格式化 (4): 16-6-26 上午11:19

日期格式化 (5): 2016-6-26 11:19:43

日期格式化 (6): 2016年6月26日 上午11时19分43秒
```

详情见http://www.runoob.com/jsp/jsp-jstl.html



#### SQL标签

使用方式：

```jsp
<%@ taglib prefix="sql" 
           uri="http://java.sun.com/jsp/jstl/sql" %>
```

| 标签                 | 描述                                                         |
| -------------------- | ------------------------------------------------------------ |
| <sql:setDataSource > | 指定数据源                                                   |
| <sql:query >         | 运行SQL查询语句                                              |
| <sql:update >        | 运行SQL更新语句                                              |
| <sql:param >         | 将SQL语句中的参数设为指定值                                  |
| <sql:dateParam >     | 将SQL语句中的日期参数设为指定的java.util.Date 对象值         |
| <sql:transaction >   | 在共享数据库连接中提供嵌套的数据库行为元素，将所有语句以一个事务的形式来运行 |

<set:setDataSource 

​	var = "<String>"		代表数据库的变量

​	scope = "<String>"		var属性的作用域	默认page

​	dataSource = "<String>"		事先准备好的数据库

​	driver = "<String>"		要注册的JDBC驱动

​	url = "<String>"		JDBC的URL

​	user = "<String>"		数据库用户名

​	password = "<String>" />	数据库密码

小栗子：

```jsp
<sql:setDataSource var="snapshot" driver="com.mysql.jdbc.Driver"
     url="jdbc:mysql://localhost/TEST"
     user="user_id"  password="mypassword"/>

<sql:query dataSource="${snapshot}" sql="..." var="result" />
```

<sql:query 

​	sql = "<String>"		sql命令（返回ResultSet对象）	默认：Body

​	dataSource = "<String>"		所使用的数据库连接	默认：默认数据库

​	maxRows = "<String>"		存储在变量中的最大结果数	默认：无穷大

​	startRows = "<String>"		开始记录的结果的行数		默认：0

​	var = "<String>"		代表数据库的变量		默认：默认配置

​	scope = "<String>" />		var属性的作用域		默认：page

栗子在上面。

<sql:update 

​	sql = "<String>"		sql语句（不返回）

​	dataSource = "<String>"		所使用的数据库连接

​	var = "<String>"		用来存储所影响行数的变量

​	scope = "<String>" />	var属性的作用域

栗子：假设已配置好setDataSource：

```jsp
<sql:update dataSource="${snapshot}" var="count">
   INSERT INTO Employees VALUES (104, 2, 'Nuha', 'Ali');
</sql:update>

<c:set var="empId" value="103"/>
<sql:update dataSource="${snapshot}" var="count">
  DELETE FROM Employees WHERE Id = ?
  <sql:param value="${empId}" />
</sql:update>
```

<sql:dataParam

​	value = "<String>"	需要设置的日期参数（java.util.Date）

​	type = "<String>" />		DATE只有日期 TIME只有时间 TIMESTAMP日期和时间

```jsp
<%
Date DoB = new Date("2001/12/16");
int studentId = 100;
%>
 
<sql:update dataSource="${snapshot}" var="count">
   UPDATE Students SET dob = ? WHERE Id = ?
   <sql:dateParam value="<%=DoB%>" type="DATE" />
   <sql:param value="<%=studentId%>" />
</sql:update>
```

<sql:transaction

​	dataSource = "<string>"		所使用的数据库

​	isolation = "<string>" />		事务隔离等级（READ_COMMITED,  READ_UNCOMMITED,  REPEATABLE_READ,  SERIALIZABLE）。默认为数据库默认。

这个标签用来将<sql:query >标签和<sql:update >标签封装至事务中，是他们成为单一的事务。它确保对数据库的修改不是被提交就是被回滚。

事务的特性：ACID（atomicity、consistency、isolation、durability）

- 原子性：事务要么全部被执行，要么全部不被执行，如果成功则提交，失败则回滚。
- 一致性（可串性）：事务的执行使数据库从一种正确状态换成另一种正确状态。
- 隔离性：事务正确提交之前，不允许把该事务对数据的任何改变提供给任何其他事务，即事务执行的可能结果不应显示给任何其他事务
- 持久性：事务正确提交后，结果将永久保存在数据库中。即使提交后有了其他故障结果也会得到保存。

#### XML标签

使用：

```jsp
<%@ taglib prefix = "x" uri = "http://java.sun.com/jsp/jstl/xml" %>
```

需要XercesImpl.jar、xalan.jar

- <x:parse >：用来解析XML文档
- <x:transform >：将XSL转换应用在XML文档中
- <x:param >：与x：transform一起使用，用于设置XSL样式表
- <x:forEacn >：迭代XML文档中的节点
- <x:set >：设置XPath表达式
- <x:if >：判断XPath表达式，为真则执行本体中的内容，否则跳过本体
- <x:out >：与<%= ... >类似，不过只用于XPath表达式
- <x:choose >
- <x:when >
- <x:otherwise >：他们与switch、case、default相同。

小栗子：

假设有如下HTML：

```html
<books>
    <book>
        <name>Padam History</name>
        <author>ZARA</author>
        <price>100</price>
    </book>
    <book>
        <name>Great Mistry</name>
        <author>NUHA</author>
        <price>2000</price>
    </book>
</books>
```

jsp：

```jsp
<c:import var = "bookInfo" url = "http://localhost:8080/books.xml" />
<x:parse xml = "${bookInfo}" var = "output" />
第一本书是：
<x:out select = "$output/bools/book[1]/name" />
<br>
第二本书的价格是：
<x:out select = "$output/books/book[2]/price" />
```

结果：

第一本书是：Padam History

第二本书的价格是：2000

 <x:set >用法：？？？

```jsp
<x:set var="fragment" select="$output//book"/>
<b>The price of the second book</b>: 
<c:out value="${fragment}" />
```

