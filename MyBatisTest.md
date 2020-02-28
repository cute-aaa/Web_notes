namespace：

namespace指定调用这个xml文件的路径，如dao.mapper.xml里的函数就可以通过session.select("dao.mapper.selectByName", name);

![1541248672037](C:\Users\I\AppData\Roaming\Typora\typora-user-images\1541248672037.png)

上面mapper里的标签，如select，是使用时的类型，如可以通过session.select("id", 参数)来调用标签为select的所有id，id参数来指定所有使用select标签中id为“id”的那一个，并把后面的参数传给他。而将Student对象传过去之后，#{id}可以直接获取对象中的id值，name值等。

例：调用addStudent：

session.select("addStudent",  new Student(1,2,3));

![1541249232551](C:\Users\I\AppData\Roaming\Typora\typora-user-images\1541249232551.png)

![1541249249775](C:\Users\I\AppData\Roaming\Typora\typora-user-images\1541249249775.png)

### 模糊查询

可以通过拼接SQL语句：

在<select>标签中，使用${ }来拼接字符串，但${}中只能使用value来代表传给它的参数：

```xml
<select id="findStudentByName" parameterMap="java.lang.String" resultType="Student">
    SELECT * FROM student WHERE name LIKE '%${value}%' 
</select>
```

其中语句就相当于 ...LiKE '%value%'  ，其中value是参数。





# 报错解决：

艰辛的旅程：

1. java.io.IOException: Could not find resource<br>原因：在IDEA中的普通Java项目而不是web项目中，并不会编译xml文件（不知道是不是用了		maven的缘故），xml文件只能放在maven生成的resources下。mapper文件也是一样。<br>解决方法：如果mapper文件过多不想都放在resources下，可以在pom.xml文件中增加：<br>

   ```xml
   <build>
       <resources>
           <resource>
               <directory>src/main/java</directory>
               <includes>
                   <include>dao/StudentDaoMapper.xml</include>
               </includes>
           </resource>
       </resources>
   </build>
   ```

   这样在编译的时候就会找到mapper文件包括进来。

2. SLF4J: Class path contains multiple SLF4J bindings.<br>原因：包含了重复的SLF4J的jar包。<br>解决方法：在Project Structure-》libraries里删除重复的jar包就行了

3. log4j:WARN No appenders could be found for logger (org.apache.ibatis.logging.LogFactory).<br>原因：使用了log4j包，却没有设置log4j.properties文件<br>解决方法：在sources里新增log4j.properties文件，里面增加的内容：<br>基础内容：<br> 1. 

   ```properties
   # Configure logging for testing: optionally with log file
   log4j.rootLogger=WARN, stdout
   # log4j.rootLogger=WARN, stdout, logfile
   
   log4j.appender.stdout=org.apache.log4j.ConsoleAppender
   log4j.appender.stdout.layout=org.apache.log4j.PatternLayout
   log4j.appender.stdout.layout.ConversionPattern=%d %p [%c] - %m%n
   
   log4j.appender.logfile=org.apache.log4j.FileAppender
   log4j.appender.logfile.File=target/spring.log
   log4j.appender.logfile.layout=org.apache.log4j.PatternLayout
   log4j.appender.logfile.layout.ConversionPattern=%d %p [%c] - %m%n
   ```

   2. 进阶内容：参照https://www.cnblogs.com/zhangwufei/p/7144306.html（已保存到印象笔记servlet笔记）

4. WARNING: An illegal reflective access operation has occurred<br>原因：JDK版本过高，JDK9或10会出现这个问题，但并不影响结果。

5. 如果出现com.mysql.jdbc.Driver抛出的错误，一般是因为mySql服务没有启动。

6. 使用两个@Test也会出现错误？
