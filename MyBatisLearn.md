每个MyBatis应用都围绕 SqlSessionFactory实例展开，SqlSessionFactory实例可以从SqlSessionFactoryBuilder获取，SqlSessionFactoryBuilder可以由XML配置文件来构建一个SqlSessionFactory实例，也可以从配置类的自定义实例中获取。

用XML文件构建一个SqlSessionFactory实例：

```java
String resource = "org/mybatis/example/mybatis-config.xml";
InputStream inputStream = Resources.getResourceAsStream(resource);
SqlSessionFactory sqlSessionFactory =
 new SqlSessionFactoryBuilder().build(inputStream);
```

XML配置文件包含了一些MyBatis的设置，包括用来获取数据库连接实例的数据库资源，以及一个用来决定事务如何被审视和控制的事务管理器；XML配置文件的全部细节都在文档后面，这里是一个小栗子：

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE configuration
 PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
 "http://mybatis.org/dtd/mybatis-3-config.dtd">
<configuration>
 <environments default="development">
     <environment id="development">
     	 <transactionManager type="JDBC"/>
         <dataSource type="POOLED">
             <property name="driver" value="${driver}"/>
             <property name="url" value="${url}"/>
             <property name="username" value="${username}"/>
             <property name="password" value="${password}"/>
         </dataSource>
     </environment>
 </environments>
 <mappers>
 	<mapper resource="org/mybatis/example/BlogMapper.xml"/>
 </mappers>
</configuration>
```

上面的例子指出了最决定性的部分。注意看XML头，XML文档需要被验证，正文中的environment标签包含了事务管理器和连接池的环境设置，mapper标签包含了一个mappers列表：这个XML文件 和/或 带注释的Java接口 包含了SQL语句和mapping定义。

##### 不使用XML，构建SqlSessionFactory

如果你更喜欢在Java里直接构建配置，或者创建一个属于自己的配置构建器，MyBatis提供了一个完整的配置类并且提供了所有与XML相同的配置选项。

```java
DataSource dataSource = BlogDataSourceFactory.getBlogDataSource();
TransactionFactory transactionFactory =
 new JdbcTransactionFactory();
Environment environment =
 new Environment("development", transactionFactory, dataSource);
Configuration configuration = new Configuration(environment);
configuration.addMapper(BlogMapper.class);
SqlSessionFactory sqlSessionFactory =
 new SqlSessionFactoryBuilder().build(configuration);
```

注意，这里的配置添加了一个mapper的class文件，这是包含了Sql映射注解（注解：Annotation）的Java类，并且避免了使用XML，但由于java注解的一些限制和MyBatis映射的复杂性，大多数的高级映射还是需要XML映射（如嵌套连接映射：Nested Join Mapping），因此，如果XML文件存在，MyBatis会自动查看XML文件。因此，BlogMapper.xml将会以BlogMapper.class为基础而被加载。稍后会详述。

##### 从SqlSessionFactory里获取SqlSession

现在你有了一个SqlSessionFactory对象，顾名思义，你可以从它身上获取一个SqlSession实例。SqlSession包含了执行SQL命令所需要的所有方法，你可以直接用SqlSession实例执行映射了的Sql语句，如：

```java
SqlSession session = sqlSessionFactory.openSession();
try {
 Blog blog = session.selectOne(
 "org.mybatis.example.BlogMapper.selectBlog", 101);
} finally {
 session.close();
}
```

##### 探索映射语句

现在你可能好奇SqlSession或Mapper class究竟执行了什么，映射SQL语句是一个很重要的主题，并且很可能会主导这个文档的大部分。这个由MyBatis提供的特色全套功能可以通过基于XML的映射语言来实现，并且让MyBatis这些年越来越火。

这里的基于XML的映射语句小栗子可以满足上面的SqlSession调用。

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper
 PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
 "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="org.mybatis.example.BlogMapper">
     <select id="selectBlog" resultType="Blog">
    	 select * from Blog where id = #{id}
     </select>
</mapper>
```

上面的标签在org.mybatis.example.BlogMapper命名空间中定义了一个名为selectBlog的映射。并允许你调用一个完整可用的名称，例：

```java
Blog blog = session.selectOne(
 "org.mybatis.example.BlogMapper.selectBlog", 101);
```

这个名称可以直接映射到命名空间中的同名Mapper class，并且以映射select标签来匹配名称，参数，返回值

还有一种：

```java
BlogMapper mapper = session.getMapper(BlogMapper.class);
Blog blog = mapper.selectBlog(101);
```

第二种方法有很多优点，首先，他不依靠字符串文字，所以更安全，第二，如果IDE有自动补全功能，你可以在导航到映射SQL语句时利用它。

###### 关于命名空间

命名空间现在是必须的，并且目的也不再不仅是简单地用长长的代码来隔离语句。

命名空间允许接口绑定，可能你现在觉得没什么用，但还是多加练习以防你将来改变想法。

当使用命名空间，并且将它放入合适的package命名空间将会使你的代码更整洁，也会提高MyBatis的长期可用性。

名称解析：

为了减少输入，MyBatis对所有命名的配置元素（包括语句，结果映射，缓存等）使用以下的名称方案：

- 全称：如：com.mypacage.MyMapper.selectAllThings，如果发现可以直接使用。
- 简化名：如：selectAllThings 可以引用任何明确的条目，但如果有同名的，你就会收到错误报告说短名称是不明确的因此必须使用全称。

##### 还有一种映射class文件的方法：

无需映射到XML文件，用Java映射来替换它：

```java
package org.mybatis.example;
public interface BlogMapper {
 @Select("SELECT * FROM blog WHERE id = #{id}")
 Blog selectBlog(int id);
}
```

对于简单的语句，Java映射要简单的多，但Java映射是有限的，并且描述更复杂。

如果要完成复杂操作，最好使用XML映射。

##### 范围和生命周期

了解了这么多，各种范围和生命周期对我们开始变得重要了，如果不能正确使用将会导致严重的并发问题。

依赖注入框架可以创建线程安全，事务性的SqlSession和映射器并把他们注入到你的beans下所以不用再管生命周期了



- SqlSessionFactoryBuilder

  这个类可以在创建完SqlSessionFactory之后就没用了，当然你也可以留着他创建多个SqlSessionFactory，但最好不要保留他，以确保所有的XML解析资源被释放出来用于更重要的事情。

- SqlSessionFactory

  一旦创建好，实例应该在应用程序执行期间都存在。应该几乎没有理由去处理或重新创建它。最好不要在程序运行期间多次重构SqlSessionFactory，这样做被认为是一种“bad smell”。因此SQL SessionFactory的最佳作用域是应用程序作用域，有很多方式可以实现，最简单的是使用单例模式或静态单例模式。

使用下方代码确保你的数据库被正确关闭。

```java
SqlSession session = sqlSessionFactory.openSession();
try {
 // do work
} finally {
 session.close();
}
```

##### 映射器对象

使用映射器接口创建映射器来绑定映射语句，映射器的实例从SqlSession获取接口，因此，广泛的说，mapper实例与请求他们的SqlSession相同。但映射器实例最好的范围是方法作用域，他们不需要显式地关闭，虽然保留他们也不是什么问题，但跟SqlSession相似，你可能会发现管理太多这个级别的资源将很快失去控制，保持简单，在方法范围内保留映射器。如：

```java
SqlSession session = sqlSessionFactory.openSession();
try {
 BlogMapper mapper = session.getMapper(BlogMapper.class);
 // do work
} finally {
 session.close();
}
```





