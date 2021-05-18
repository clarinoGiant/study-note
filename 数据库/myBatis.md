# 一、基本使用

官网：https://mybatis.org/mybatis-3/zh/index.html

- 每个基于 MyBatis 的应用都是以一个 SqlSessionFactory 的实例为核心的。

- SqlSessionFactory 的实例可以通过 SqlSessionFactoryBuilder 获得。

-  SqlSessionFactoryBuilder 则可以从 XML 配置文件或一个预先配置的 Configuration 实例来构建出 SqlSessionFactory 实例。

- MyBatis 包含一个名叫 Resources 的工具类，它包含一些实用方法，使得从类路径或其它位置加载资源文件更加容易。

## 1.1 SqlSessionFactoryBuilder

这个类可以被实例化、使用和丢弃，一旦创建了 SqlSessionFactory，就不再需要它了。 因此 SqlSessionFactoryBuilder 实例的最佳作用域是==方法作用域（也就是局部方法变量==）。 你可以重用 SqlSessionFactoryBuilder 来创建多个 SqlSessionFactory 实例，但最好还是不要一直保留着它，以保证所有的 XML 解析资源可以被释放给更重要的事情。

## 1.2 SqlSessionFactory

SqlSessionFactory 一旦被创建就应该在应用的运行期间一直存在，没有任何理由丢弃它或重新创建另一个实例。 使用 SqlSessionFactory 的最佳实践是在应用运行期间不要重复创建多次，多次重建 SqlSessionFactory 被视为一种代码“坏习惯”。因此 SqlSessionFactory 的==最佳作用域是应用作用域==。 有很多方法可以做到，最简单的就是==使用单例模式或者静态单例模式==。

## 1.3 SqlSession

每个线程都应该有它自己的 SqlSession 实例。==SqlSession 的实例不是线程安全的，因此是不能被共享的，所以它的最佳的作用域是请求或方法作用域==。 

绝对不能将 SqlSession 实例的引用放在一个类的静态域，甚至一个类的实例变量也不行。 也绝不能将 SqlSession 实例的引用放在任何类型的托管作用域中，比如 Servlet 框架中的 HttpSession。 

如果你现在正在使用一种 Web 框架，考虑将 SqlSession 放在一个和 HTTP 请求相似的作用域中。 换句话说，==每次收到 HTTP 请求，就可以打开一个 SqlSession，返回一个响应后，就关闭它==。 这个关闭操作很重要，为了确保每次都能执行关闭操作，你应该把这个关闭操作放到 finally 块中。 下面的示例就是一个确保 SqlSession 关闭的标准模式：

```
try (SqlSession session = sqlSessionFactory.openSession()) {
  // 你的应用逻辑代码
}
```

在所有代码中都遵循这种使用模式，可以保证所有数据库资源都能被正确地关闭。

## 1.4 映射器实例

映射器是一些绑定映射语句的接口。==映射器接口的实例是从 SqlSession 中获得的。==虽然从技术层面上来讲，任何映射器实例的最大作用域与请求它们的 SqlSession 相同。但方法作用域才是映射器实例的最合适的作用域。 ==映射器实例应该在调用它们的方法中被获取，使用完毕之后即可丢弃==。 映射器实例并不需要被显式地关闭。 因此，最好将映射器放在方法作用域内。

```
try (SqlSession session = sqlSessionFactory.openSession()) {
  BlogMapper mapper = session.getMapper(BlogMapper.class);
  // 你的应用逻辑代码
}
```



# 二、入门操作

## 2.1 mySQL创表

```sql
CREATE DATABASE 'mybatis';

USE 'mybatis';

CREATE TABLE 'user'(
    'id' INT(20) NOT NULL PRIMARY_KEY,
    'name' VARCHAR(30) DEFAULT NULL,
    'pwd' VARCHAR(30) DEFAULT NULL
) ENGINE=INNODB DEFAULT CHARSET=utf8

INSERT INTO 'user' ('id', 'name', 'pwd') VALUES
(1, '张三', '123456'),
(2, '李四', '123456')
```

## 2.2 新建Maven工程

1. 增加pom依赖

   ```xml
   <dependencies>
           <dependency>
               <groupId>mysql</groupId>
               <artifactId>mysql-connector-java</artifactId>
               <version>5.1.27</version>
           </dependency>
           <dependency>
               <groupId>org.mybatis</groupId>
               <artifactId>mybatis</artifactId>
               <version>3.5.3</version>
           </dependency>
           <dependency>
               <groupId>junit</groupId>
               <artifactId>junit</artifactId>
               <version>4.12</version>
               <scope>test</scope>
           </dependency>
       </dependencies>
   ```

1. 定义实体类

   ```java
   public class MybatisUtils {
       // 由SqlSessionFactoryBuilder根据配置生成SqlSessionFactory
       private static SqlSessionFactory sqlSessionFactory;
   
       static {
           try {
               String resource = "mybatis-config.xml";
               InputStream inputStream = Resources.getResourceAsStream(resource);
               sqlSessionFactory = new SqlSessionFactoryBuilder().build(inputStream);
           } catch (IOException e) {
               e.printStackTrace();
           }
       }
   
       // SqlSession 包含了执行SQL的所有方法
       public static SqlSession getSqlSession() {
           return sqlSessionFactory.openSession();
       }
   }
   
   ```

2. 创建mybatis-config.xml

   >  XML 配置文件中包含了对 MyBatis 系统的核心设置，包括获取数据库连接实例的数据源（DataSource）以及决定事务作用域和控制方式的事务管理器（TransactionManager）。

   ```xml
   <?xml version="1.0" encoding="UTF-8" ?>
   <!DOCTYPE configuration
           PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
           "http://mybatis.org/dtd/mybatis-3-config.dtd">
   <configuration>
   
       <properties>
           <property name="username" value="root"/>
           <property name="password" value="123456"/>
       </properties>
   
       <environments default="development">
           <environment id="development">
               <transactionManager type="JDBC"/>
               <dataSource type="POOLED">
                   <property name="driver" value="com.mysql.jdbc.Driver"/>
                   <property name="url" value="jdbc::mysql://localhost:3306/mybatis?userSSL=true&amp;useUnicode=true&amp;characterEncoding=UTF-8"/>
                   <property name="username" value="${username}"/>
                   <property name="password" value="${password}"/>
               </dataSource>
           </environment>
       </environments>
       <mappers>
           <mapper resource="UserMapper.xml"/>
       </mappers>
   </configuration>
   ```

3. 创建POJO对象

   ```java
   @Getter
   @Setter
   @Data
   public class User {
       private int id;
       private String name;
       private String pwd;
   
       public User(int id, String name, String pwd) {
           this.id = id;
           this.name = name;
           this.pwd = pwd;
       }
   }
   ```
   
4. 定义Dao接口

   ```java
   public interface UserMapper {
       public List<User> getUserList();
   }
   ```

5. 书写Mapper文件，并在mybatis-config.xml中增加Mapper路径配置

   ```xml
   <?xml version="1.0" encoding="UTF-8" ?>
   <!DOCTYPE mapper
           PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
           "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
   
   <!-- 绑定一个对应的Dao/Mapper接口，不用生成对应的DaoImpl -->
   <mapper namespace="com.test.dao.UserMapper">
       <select id="getUserList" resultType="com.test.pojo.User">
           select * from mybatis.User
       </select>
   </mapper>
   ```

## 2.3 测试

```java
public class UserDaoTest extends TestCase {
    @Test
    public void test() {
        // 获取SqlSession对象
        SqlSession sqlSession = MybatisUtils.getSqlSession();

        // 获取Mapper对象
        UserMapper userMapper = sqlSession.getMapper(UserMapper.class);
        List<User> userList = userMapper.getUserList();

        sqlSession.close();
    }
}
```

# 三、注意点

增删改场景，需要执行提交事务语句，相关语句才会执行



# 四、myBatis与Spring集成

pom依赖



# 五、普通SQL命令

## SELECT

## INSERT

## UPDATE

## DELETE



## 其他语法

### ResultMap



# 六、动态语句

if

foreach