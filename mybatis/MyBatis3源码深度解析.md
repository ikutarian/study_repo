# XML 配置文件的解析

## XPath方式解析XML文件

MyBatis 的主配置文件和 Mapper 配置文件都使用的是 XML 格式。并且是用 XPATH 方式解析 XML 文件中的配置信息

下面是一个示例用的 XML 文件

```xml
<?xml version="1.0" encoding="utf-8" ?>
<users>
    <user id="1">
        <name>张三</name>
        <createTime>2020-11-17 15:57:00</createTime>
        <password>admin</password>
        <phone>18000000000</phone>
        <nickName>阿毛</nickName>
    </user>
    <user id="2">
        <name>李四</name>
        <createTime>2021-11-11 11:11:11</createTime>
        <password>admin</password>
        <phone>18000000001</phone>
        <nickName>明明</nickName>
    </user>
</users>
```

以及对应的 pojo

```java
import org.joda.time.DateTime;

public class UserEntity {

    private Long id;
    private String name;
    private DateTime createTime;
    private String password;
    private String phone;
    private String nickName;
    
    // getter
    // setter
    // toString
}    
```

如果用 Java 的原生代码使用 XPATH 方式解析 XML 文件的代码如下

```java
import org.joda.time.format.DateTimeFormat;
import org.w3c.dom.Document;
import org.w3c.dom.NodeList;

import javax.xml.parsers.DocumentBuilder;
import javax.xml.parsers.DocumentBuilderFactory;
import javax.xml.xpath.XPath;
import javax.xml.xpath.XPathConstants;
import javax.xml.xpath.XPathFactory;
import java.io.InputStream;
import java.util.ArrayList;
import java.util.List;

/**
 * 使用xpath解析XML文件
 */
public class XPathTest {

    public static void main(String[] args) throws Exception {
        // 读取xml文件
        DocumentBuilderFactory factory = DocumentBuilderFactory.newInstance();
        DocumentBuilder builder = factory.newDocumentBuilder();
        InputStream inputSource = XPathTest.class.getClassLoader().getResourceAsStream("users.xml");
        Document doc = builder.parse(inputSource);
        // 创建xpath实例
        XPath xPath = XPathFactory.newInstance().newXPath();
        // 执行xpath表达式，获取xml节点信息
        NodeList nodeList = (NodeList) xPath.evaluate("/users/*", doc, XPathConstants.NODESET);
        List<UserEntity> userEntityList = new ArrayList<>(nodeList.getLength());
        for (int i = 1; i < nodeList.getLength() + 1; i++) {
            String path = "/users/user[" + i + "]";
            String id = (String) xPath.evaluate(path + "/@id", doc, XPathConstants.STRING);
            String name = (String) xPath.evaluate(path + "/name", doc, XPathConstants.STRING);
            String createTime = (String) xPath.evaluate(path + "/createTime", doc, XPathConstants.STRING);
            String password = (String) xPath.evaluate(path + "/password", doc, XPathConstants.STRING);
            String phone = (String) xPath.evaluate(path + "/phone", doc, XPathConstants.STRING);
            String nickName = (String) xPath.evaluate(path + "/nickName", doc, XPathConstants.STRING);

            // 转换为UserEntity实体对象
            UserEntity userEntity = new UserEntity();
            userEntity.setId(Long.parseLong(id));
            userEntity.setName(name);
            userEntity.setCreateTime(DateTimeFormat.forPattern("yyyy-MM-dd HH:mm:ss").parseDateTime(createTime));
            userEntity.setPassword(password);
            userEntity.setPhone(phone);
            userEntity.setNickName(nickName);
            userEntityList.add(userEntity);
        }
        userEntityList.forEach(System.out::println);
    }
}
```

MyBatis 通过 `XPathParser` 工具类封装了对 XML 的解析操作，同时使用 `XNode` 类增强了对 XML 节点的操作。使用 `XNode` 对象，可以很方便地获取节点的属性、子节点等信息

```java
import org.apache.ibatis.io.Resources;
import org.apache.ibatis.parsing.XNode;
import org.apache.ibatis.parsing.XPathParser;
import org.joda.time.format.DateTimeFormat;

import java.util.ArrayList;
import java.util.List;

public class XPathParserTest {

    public static void main(String[] args) throws Exception {
        XPathParser parser = new XPathParser(Resources.getResourceAsReader("users.xml"));
        List<XNode> nodes = parser.evalNodes("/users/*");
        List<UserEntity> userEntityList = new ArrayList<>(nodes.size());
        for (XNode node : nodes) {
            Long id = node.getLongAttribute("id");
            String name = node.evalNode("name").getStringBody();
            String createTime = node.evalNode("createTime").getStringBody();
            String password = node.evalNode("password").getStringBody();
            String phone = node.evalNode("phone").getStringBody();
            String nickName = node.evalNode("nickName").getStringBody();

            UserEntity userEntity = new UserEntity();
            userEntity.setId(id);
            userEntity.setName(name);
            userEntity.setCreateTime(DateTimeFormat.forPattern("yyyy-MM-dd HH:mm:ss").parseDateTime(createTime));
            userEntity.setPassword(password);
            userEntity.setPhone(phone);
            userEntity.setNickName(nickName);
            userEntityList.add(userEntity);
        }
        userEntityList.forEach(System.out::println);
    }
}
```

## 读取 Mybatis 的配置文件

一个简单的 Mybatis 的配置文件

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

读取配置文件，生成 `SqlSessionFactory` 的 Java 代码如下

```java
String resource = "org/mybatis/example/mybatis-config.xml";
InputStream inputStream = Resources.getResourceAsStream(resource);
SqlSessionFactory sqlSessionFactory = new SqlSessionFactoryBuilder().build(inputStream);
```

进入 `build` 方法

```java
public SqlSessionFactory build(InputStream inputStream, String environment, Properties properties) {
  try {
    XMLConfigBuilder parser = new XMLConfigBuilder(inputStream, environment, properties);
    return build(parser.parse());
  } catch (Exception e) {
    throw ExceptionFactory.wrapException("Error building SqlSession.", e);
  } finally {
    ErrorContext.instance().reset();
    try {
      inputStream.close();
    } catch (IOException e) {
      // Intentionally ignore. Prefer previous error.
    }
  }
}
```

它创建了 `XMLConfigBuilder` 对象，调用它的 `parse()` 方法

```java
public Configuration parse() {
  // 防止 parse() 方法被多次调用
  if (parsed) {
    throw new BuilderException("Each XMLConfigBuilder can only be used once.");
  }
  parsed = true;
  // 调用 evalNode 方法通过 XPATH 的语法获取到 configuration 节点
  // 调用 parseConfiguration 方法解析 configuration 节点获取更多信息
  parseConfiguration(parser.evalNode("/configuration"));
  return configuration;
}
```

在 `parseConfiguration()` 方法中，对于 `<configuration>` 标签的子节点，都有一个单独的方法处理

```java
private void parseConfiguration(XNode root) {
  try {
    // issue #117 read properties first
    propertiesElement(root.evalNode("properties"));
    Properties settings = settingsAsProperties(root.evalNode("settings"));
    loadCustomVfs(settings);
    loadCustomLogImpl(settings);
    typeAliasesElement(root.evalNode("typeAliases"));
    pluginElement(root.evalNode("plugins"));
    objectFactoryElement(root.evalNode("objectFactory"));
    objectWrapperFactoryElement(root.evalNode("objectWrapperFactory"));
    reflectorFactoryElement(root.evalNode("reflectorFactory"));
    settingsElement(settings);
    // read it after objectFactory and objectWrapperFactory issue #631
    environmentsElement(root.evalNode("environments"));
    databaseIdProviderElement(root.evalNode("databaseIdProvider"));
    typeHandlerElement(root.evalNode("typeHandlers"));
    mapperElement(root.evalNode("mappers"));
  } catch (Exception e) {
    throw new BuilderException("Error parsing SQL Mapper Configuration. Cause: " + e, e);
  }
}
```

解析完毕之后就返回一个 `Configuration` 对象，它包含了配置文件的中的所有信息。然后 `build()` 方法再调用另外一个重载方法创建 `SqlSessionFactory` 的实现类对象 `DefaultSqlSessionFactory`

```java
public SqlSessionFactory build(Configuration config) {
  return new DefaultSqlSessionFactory(config);
}
```

# 从 SqlSessionFactory 中获取 SqlSession

解析配置文件创建 `SqlSessionFactory` 对象之后，就可以获取 `SqlSession` 了

```java
SqlSession session = sqlSessionFactory.openSession();
```

`openSession()` 会调用 `openSessionFromDataSource()` 方法创建 `SqlSession` 实例 —— `DefaultSqlSession`

```java
private SqlSession openSessionFromDataSource(ExecutorType execType, TransactionIsolationLevel level, boolean autoCommit) {
  Transaction tx = null;
  try {
    // 获取配置文件中 environment 节点的配置信息
    final Environment environment = configuration.getEnvironment();
    // 创建事务管理器工厂
    final TransactionFactory transactionFactory = getTransactionFactoryFromEnvironment(environment);
    // 创建事务管理器
    tx = transactionFactory.newTransaction(environment.getDataSource(), level, autoCommit);
    // 根据配置文件中指定的 Executor 类型，创建 Executor 实例
    final Executor executor = configuration.newExecutor(tx, execType);
    // 创建 DefaultSqlSession 实例
    return new DefaultSqlSession(configuration, executor, autoCommit);
  } catch (Exception e) {
    closeTransaction(tx); // may have fetched a connection so lets call close()
    throw ExceptionFactory.wrapException("Error opening session.  Cause: " + e, e);
  } finally {
    ErrorContext.instance().reset();
  }
}
```

# SqlSession 执行 Mapper 的过程

获取 `SqlSession` 实例之后，就可以调用 Mapper 的方法执行 SQL 语句

```java
BlogMapper mapper = session.getMapper(BlogMapper.class);
Blog blog = mapper.selectBlog(101);
```

我们知道，接口中定义的方法必须通过某个类实现该接口，然后创建该类的实例，才能通过实例调用方法。所以 `SqlSession` 对象的 `getMapper()` 方法返回的一定是某个类的实例。具体是哪个类的实例呢？实际上 `getMapper()` 方法返回的是一个**动态代理对象**

`session.getMapper(BlogMapper.class)` 会调用 `configuration.getMapper` 方法

```java
@Override
public <T> T getMapper(Class<T> type) {
  return configuration.getMapper(type, this);
}
```

`configuration.getMapper` 方法内部会调用 `mapperRegistry.getMapper` 方法

```java
public <T> T getMapper(Class<T> type, SqlSession sqlSession) {
  return mapperRegistry.getMapper(type, sqlSession);
}
```

`mapperRegistry` 是 `Configuration` 对象内部的一个变量

```java
protected final MapperRegistry mapperRegistry = new MapperRegistry(this);
```

`session.getMapper` 调用的是 `mapperRegistry.getMapper` 方法，那么 `mapperRegistry` 的 `addMapper` 方法在哪里调用？是在解析 XML 文件生成 `Configuration` 对象的是调用的

```java
private void parseConfiguration(XNode root) {
  try {
    // issue #117 read properties first
    propertiesElement(root.evalNode("properties"));
    Properties settings = settingsAsProperties(root.evalNode("settings"));
    loadCustomVfs(settings);
    loadCustomLogImpl(settings);
    typeAliasesElement(root.evalNode("typeAliases"));
    pluginElement(root.evalNode("plugins"));
    objectFactoryElement(root.evalNode("objectFactory"));
    objectWrapperFactoryElement(root.evalNode("objectWrapperFactory"));
    reflectorFactoryElement(root.evalNode("reflectorFactory"));
    settingsElement(settings);
    // read it after objectFactory and objectWrapperFactory issue #631
    environmentsElement(root.evalNode("environments"));
    databaseIdProviderElement(root.evalNode("databaseIdProvider"));
    typeHandlerElement(root.evalNode("typeHandlers"));
    mapperElement(root.evalNode("mappers"));   // 《=================== 解析 mapppers 节点
  } catch (Exception e) {
    throw new BuilderException("Error parsing SQL Mapper Configuration. Cause: " + e, e);
  }
}
```

它会调用 `MapperRegistry` 的 `addMapper` 方法

```java
private final Map<Class<?>, MapperProxyFactory<?>> knownMappers = new HashMap<>();

public <T> void addMapper(Class<T> type) {
  if (type.isInterface()) {
    if (hasMapper(type)) {
      throw new BindingException("Type " + type + " is already known to the MapperRegistry.");
    }
    boolean loadCompleted = false;
    try {
      knownMappers.put(type, new MapperProxyFactory<>(type));
      // It's important that the type is added before the parser is run
      // otherwise the binding may automatically be attempted by the
      // mapper parser. If the type is already known, it won't try.
      MapperAnnotationBuilder parser = new MapperAnnotationBuilder(config, type);
      parser.parse();
      loadCompleted = true;
    } finally {
      if (!loadCompleted) {
        knownMappers.remove(type);
      }
    }
  }
}
```

以 Mapper 接口为 key，`MapperProxyFactory` 为 value，加入集合中

这样在获取 Mapper 实现类时

```java
BlogMapper mapper = session.getMapper(BlogMapper.class);
```

可以通过 Mapper 接口的类名，获取它的包装类 `MapperProxyFactory` ，由 `MapperProxyFactory`  来创建动态代理对象

```java
public <T> T getMapper(Class<T> type, SqlSession sqlSession) {
  final MapperProxyFactory<T> mapperProxyFactory = (MapperProxyFactory<T>) knownMappers.get(type);
  if (mapperProxyFactory == null) {
    throw new BindingException("Type " + type + " is not known to the MapperRegistry.");
  }
  try {
    return mapperProxyFactory.newInstance(sqlSession);
  } catch (Exception e) {
    throw new BindingException("Error getting mapper instance. Cause: " + e, e);
  }
}
```

