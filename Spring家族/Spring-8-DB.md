参考：《精通Spring 4.x企业应用开发实战》

# 一、Spring对DAO的支持

​	UserService -> UserDao( JdbcUserDAo、MybatisUserDao、HibernateUserDao 不同的持久化技术)。

## 1. Spring DAO异常体系

普通的API或框架存在的问题：

- 检查型异常被过多使用，业务代码充斥大量try/catch代码。很多时候业务代码比较少；

- 引发异常的问题往往不可恢复，如数据库连接失败，SQL语法问题

Spring异常体系建立在**运行期异常**基础上，开发者根据需要捕捉感兴趣的异常。

Spring在org.springframework.dao包中提供了**完备优雅**的DAO异常体系，继承自DataAccessException（继承自NestedRuntimeException）。NestedRuntimeException封装了源异常，用户可通过getCause()获取原始异常信息。

问题：普通的JDBC的SQLException，用户通过ErrorCode和SQLState获取错误代码，然后判断原因。问题：偏底层，且ErrorCode与数据库相关。

方案：Spring以分类方法建立了异常分类目录：1）用户不关注底层细节；2）选择感兴趣异常进行处理。

Spring DAO异常体系：

**DataAccessException下一级异常子类**

- CleanupFailureDataAccessException	执行 DAO 操作成功，但在释放数据资源时发生异常，如关闭 Connection 时发生异常。
- ConcurrencyFailureException	并发地操作数据时发生异常，如无法获取乐观锁或悲观锁时、死锁引发的失败等场景。
- DataAccessResourceFailureException	访问数据资源失败，如无法获取数据连接，无法获取 Hibernate 的会话等场景。
- DataRetrievalFailureException	获取数据失败，如找不到对应主键的数据或使用了错误的列索引等场景。
- DataSourceLookupFailureException	无法从 JNDI 中查找到数据源。
- DataIntegrityViolationException	数据操作违反了数据一致性限制时抛出，如插入重复的主键或引用不存在的外键场景。
- InvalidDataAccessApiUsageException	不正确地调用某一种持久化技术时抛出，如在 Spring JDBC 中查询对象在调用前没有事先进行编译操作，就会抛出该异常。这种异常主要是因为不正确地使用持久化技术而产生的。
- InvalidDataAccessResourceUsageException	在访问数据源时使用了不正确的方法时抛出，如写错 SQL 语句。
- PermissionDeniedDataAccessException	数据访问权限不足时抛出。如仅拥有只读权限却试图更改数据。
- UncategorizedDataAccessException	其它未被分类的异常。

每个子类下又细分子类，对应更详细原因，如BadSqlGrammerException

## 2. JDBC的异常转换器

​	传统的JDBC API在发生所有的数据操作异常时都抛出SQLException，用户必须调用进一步方法获取异常信息。

​	SQLException：

​		1. 错误码：与具体数据库相关，getErrorCode()，int

​		2. SQL状态码：标准的错误代码，getSQLState()，String

Spring根据上述错误码和状态码将SQLException翻译成Spring DAO异常体系，内部定义了org.springframework.jdbc.support下定义接口 SQLExceptionTranslator（实现类 SQLErrorCodeSQLExceptionTranslator和SQLStateSQLExceptionTranslator）

## 3. 持久化模板类jdbcTemplate

Spring封装了数据库连接获取等操作，将访问流程固化到模板类中，并将数据访问中固定和变化分开，同时保证模板类是线程安全的；使得多给线程访问共享一个模板实例。访问变化的部分通过回调接口开放(定义具体数据访问和结果返回的操作)。

![image-20210504223034603](Spring-8-DB.assets/image-20210504223034603.png)

​	针对不同持久化技术的模板类

- JDBC: org.springframework.jdbc.core.JdbcTemplate

- JPA: org.springframework.orm.jpa.JpaTemplate
- Hibernate X.0: org.springframework.orm.hibernateX.HibernateTemplate

如果直接使用模板类，一般需要DAO中定义一个模板对象并提供数据源

- JDBC:  org.springframework.jdbc.core.JdbcDaoSupport
- JPA: org.springframework.orm.jpa.JpaDaoSupport
- Hibernamte X.0: org.springframework.orm.hibernateX.HibernateDaoSupport

上述支持类都继承于dao.support.DaoSupport类。DaoSupport类实现了InitialzingBean接口，在afterPropertiesSet()接口中检查模板对象和数据源是否正确设置，否则抛出异常。

所有支持类都是abstract，并非直接使用。

## 4. 数据源（连接池）

​	spring中数据连接通过数据源获取。不仅可以通过JNDI获取应用服务器的数据源，也可以直接在spring容器配置数据源。

### 4.1  dbcp（三方）

​	Apache DBCP是一个依赖Jakarta commons-pool对象池机制的数据连接池，需引入commons-pool的依赖。

   POM配置：

   ```xml
<dependency>
    <groupId>commons-dbcp</groupId>
    <artifactId>commons-dbcp</artifactId>
    <version>1.4</version>
</dependency>
   ```

xml配置（如下仅列举部分属性）

```xml
<bean id="dataSource"
      class="org.apache.commons.dbcp.BasicDataSource"
      destroy-method="close">
    <property name="driverClassName" value="com.mysql.jdbc.Driver"/>
    <property name="url" value="jdbc:mysql://localhost:3306/test?useUnicode=true&characterEncoding=UTF-8"/>
    <property name="username" value="root"/>
    <property name="password" value="root"/>
    
    <!-- 1.数据源连接数量 -->  
    <!--maxActive: 最大连接数量-->  
    <property name="maxActive" value="150"/>
    <!--minIdle: 最小空闲连接-->  
    <property name="minIdle" value="5"/>
    <!--maxIdle: 最大空闲连接-->  
    <property name="maxIdle" value="20"/>
    <!--initialSize: 初始化连接-->  
    <property name="initialSize" value="30"/>
    <!--maxWait: 超时等待时间以毫秒为单位 1000等于60秒-->
    <property name="maxWait" value="1000"/>

    <!-- 2.连接泄露回收 -->
    <!-- 连接被泄露时是否打印 -->
    <property name="logAbandoned" value="true"/>
    <!--removeAbandoned: 是否自动回收超时连接-->  
    <property name="removeAbandoned"  value="true"/>
    <!--removeAbandonedTimeout: 超时时间(以秒数为单位)-->  
    <property name="removeAbandonedTimeout" value="10"/>
    
    <!-- 3.连接健康状况维护和检测 -->
    <!-- 在连接返回调用者前，用此SQL验证从连接池取出的连接是否可用 -->
    <property name="validationQuery" value="SELECT NOW() FROM DUAL"/>
    <!-- 在空闲连接回收器线程运行期间休眠的时间值,以毫秒为单位. -->
    <property name="timeBetweenEvictionRunsMillis" value="10000"/>
    <!--  在每次空闲连接回收器线程(如果有)运行时检查的连接数量 -->
    <property name="numTestsPerEvictionRun" value="10"/>
    <!-- 1000 * 60 * 30  连接在池中保持空闲而不被空闲连接回收器线程-->
    <property name="minEvictableIdleTimeMillis" value="10000"/>

</bean>
```

> 必须设置destroy-method，确保spring关闭时，数据源能正常关闭。

​	**MySQL 8小时问题：**Mysql默认情况下如发现一个连接空闲时间超过8小时，会在数据库端自动关闭该连接。客户端并不感知，此时如果时用时提示connection异常

​	如果采用DBCP默认配置，由于testOnBorrow默认true，数据源在将连接返回DAO前，事先检测连接是否正常，如异常则取一个其他连接给DAO，从而不会发生“8小时问题”。但每次都检测在高并发场景带来性能问题。

​	推荐方案：设置testOnBorrow=false，testWhileIdle=true，并设置timeBetweenEvictionRunsMillis。从而DBCP后台线程定时对空闲连接检测，并对被数据库关闭的连接清除。只要将timeBetweenEvictionRunsMillis设置小于8小时，就可以避免MySQL的8小时问题。

### 4.2 C3P0（三方）

pom依赖

```xml
<dependency>
    <groupId>com.mchange</groupId>
    <artifactId>c3p0</artifactId>
    <version>0.9.5.5</version>
</dependency>
```

bean.xml配置

```xml
<bean id="dataSource" class="com.mchange.v2.c3p0.ComboPooledDataSource"
      destroy-method="close">
      <property name="driverClass" value="com.mysql.jdbc.Driver"/>
      <property name="jdbcUrl" value="jdbc:mysql://localhost:3306/test?useUnicode=true&characterEncoding=UTF-8"/>
	    <!-- 用户名-->
      <property name="user" value="root"/>
      <!-- 用户密码-->
      <property name="password" value="123456"/>
    

       <!--连接池中保留的最大连接数。默认值: 15 --> 
      <property name="maxPoolSize" value="20"/>
      <!-- 连接池中保留的最小连接数，默认为：3-->
      <property name="minPoolSize" value="2"/>
      <!-- 初始化连接池中的连接数，取值应在minPoolSize与maxPoolSize之间，默认为3-->
      <property name="initialPoolSize" value="2"/>

      <!--最大空闲时间，60秒内未使用则连接被丢弃。若为0则永不丢弃。默认值: 0 --> 
      <property name="maxIdleTime">60</property>

      <!-- 当连接池连接耗尽时，客户端调用getConnection()后等待获取新连接的时间，超时后将抛出SQLException，如设为0则无限期等待。单位毫秒。默认: 0 --> 
      <property name="checkoutTimeout" value="3000"/>

      <!--当连接池中的连接耗尽的时候c3p0一次同时获取的连接数。默认值: 3 --> 
      <property name="acquireIncrement" value="2"/>

     <!--定义在从数据库获取新连接失败后重复尝试的次数。默认值: 30 ；小于等于0表示无限次--> 
      <property name="acquireRetryAttempts" value="0"/>

      <!--重新尝试的时间间隔，默认为：1000毫秒--> 
      <property name="acquireRetryDelay" value="1000" />

      <!--关闭连接时，是否提交未提交的事务，默认为false，即关闭连接，回滚未提交的事务 --> 
      <property name="autoCommitOnClose">false</property>

      <!--c3p0将建一张名为Test的空表，并使用其自带的查询语句进行测试。如果定义了这个参数那么属性preferredTestQuery将被忽略。你不能在这张Test表上进行任何操作，它将只供c3p0测试使用。默认值: null --> 
      <property name="automaticTestTable">Test</property>

      <!--如果为false，则获取连接失败将会引起所有等待连接池来获取连接的线程抛出异常，但是数据源仍有效保留，并在下次调用getConnection()的时候继续尝试获取连接。如果设为true，那么在尝试获取连接失败后该数据源将申明已断开并永久关闭。默认: false--> 
      <property name="breakAfterAcquireFailure">false</property>

      <!--每60秒检查所有连接池中的空闲连接。默认值: 0，不检查 --> 
      <property name="idleConnectionTestPeriod">60</property>
      <!--c3p0全局的PreparedStatements缓存的大小。如果maxStatements与maxStatementsPerConnection均为0，则缓存不生效，只要有一个不为0，则语句的缓存就能生效。如果默认值: 0--> 
      <property name="maxStatements">100</property>
      <!--maxStatementsPerConnection定义了连接池内单个连接所拥有的最大缓存statements数。默认值: 0 --> 
      <property name="maxStatementsPerConnection"></property>
 </bean>
```

### 4.3 DriverManagerDataSource(Spring)

 org.springframework.jdbc.datasource.DriverManagerDataSource，实现了javax.sql.DataSource接口，不提供数据库连接池。每次getConnecton()都是获取新连接，可用于在单元测试或简单应用使用。

# 二、Spring JDBC访问数据库

## 1. JdbcTemplate

线程安全，所以所有DAO都可共享同一个实例。

内部使用PreparedStatement执行SQL语句。

```java
package com.smart.dao;

@Repository
public class ForumDao {
    private JdbcTemplate jdbcTemplate;
    
    @Autowired
    public void setJdbcTemplate(JdbcTemplate jdbcTemplate) {
        this.jdbcTemplate = jdbcTemplate;
    }
    
    public void initDB() {
        String sql = "create table t_user(user_id int primary, user_name varchar(60))";
        jdbcTempalte.execute(sql);
    }
}
```

spring配置文件

```xml
<context:component-scan base-package="com.smart" />

<context:property-placeholder location ="classpath:jdbc.properties" />
<bean id="dataSource" class="com.apache.commons.dbcp.BasicDataSource"
      destroy-method="close"
      p:driverClassName="${jdbc.driverClassName}"
      p:url="${jdbc.url}"
      p:username="${jdbc.username}"
      p:password="${jdbc.password}" />
<bean id="jdbcTemplate"
      class="org.springframework.jdbc.core.JdbcTemplate"
      p:dataSource-ref="dataSource" />
```

### 1.1 更改数据update

update方法：允许对数据表记录进行更改和删除。

- int update(String sql): 不带占位符
- int update(String sql, Object... args): 不定参数场景
- int update(String sql, PreparedStatementSetter pss): 第2个参数是回调接口。
- int update(PreparedStatementCreator psc): 参数是回调接口，负责创建一个PreparedStatement实例。
- int upate(PreparedStatementCreator ps, PreparedStatementSetter pss)

使用样例：

```java
public void addForum(Forum forum) {
    String sql = "INSERT INTO t_forum(forum_name, forum_desc) VALUES(?,?)";
    Object[] params = new Object[]{forum,getForumName(), forum.getForumDesc()};
    jdbcTemplate.update(sql, params);
}
```

更好的方式(显示指定每个占位符对应的字段数据类型，避免上面由Spring猜测错误)

```java
import java.sql.Types;

public void addForum(Forum forum) {
    String sql = "INSERT INTO t_forum(forum_name, forum_desc) VALUES(?,?)";
    Object[] params = new Object[]{forum,getForumName(), forum.getForumDesc()};
    jdbcTemplate.update(sql, params, new int[]{Types.VARCHAR, Types.VARCHAR});
}
```

其他样例：

```java
public void addForum(Forum forum) {
    // .. 
    jdbcTemplate.update(sql, new PreparedStatementSetter() {
        public void setValues(PreparedStatement ps) throws SQLException {
            ps.setString(1, forum.getForumName());
            ps.setString(2, forum.getForumDesc());
        }
    });
}

public void addForum(Forum forum) {
    // .. 
    jdbcTemplate.update(sql, new PreparedStatementCreator() {
        public PreparedStatement createPreparedStatement(Connection conn) throws SQLException {
            PreparedStatement ps = conn.getPreparedStatement(sql);
            ps.setString(1, forum.getForumName());
            ps.setString(2, forum.getForumDesc());
            return ps;
        }
    });
}
```

### 1.2 查询querry





# 三、事务