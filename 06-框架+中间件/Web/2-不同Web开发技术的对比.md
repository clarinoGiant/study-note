# 一、基础知识

​	WEB开发基于Servlet基础。

## 1、Servlet容器类型和对比

​	Tomcat、Jetty等

# 二、不同开发方式对比

## 1、对比

- 纯Servlet开发：

  ​	1）针对不同的url配置不同的Servlet和对应Mapping；

  ​	2）service代码直接继承HttpServlet开发，并覆盖实现其中的doGet、doPost等方法

- SpringMVC：

  ​	1）在web.xml中配置dispatchServlet，负责处理所有的请求，通常仅配置这一个Servlet，负责转发不同的请求到实现方；

  ​	2）代码使用@RequestMapping等注解实现Controller；

  ​	3）web.xml中context-param中指定contextConfigLocation指定applicationContext.xml等信息，用于web启动时初始化加载spring bean。

- SpringBoot：1）内部封装了tomcat；2）代码直接基于注解实现controller；

## 2、纯Servlet开发样例

​	需要手写跳转servlet，继承HttpServlet，覆写其中的doGet、doDelete等方法，开发效率比较低

## 3、SpringMVC

web.xml同一配置DispatcherServlet：实现了servlet接口，来自前端的请求先到这里，然后去匹配后台合适的handler

具体controller开发，使用spring提供的注解 @RequestMapping等

classpath：指的就是编译之后，在Tomcat\工程根目录\WEB-INF\classes文件夹

- web.xml

```xml
<!-- 要求初始化ContextLoaderListener设置的contextConfigLocation才能生效 -->
<listener>
    <listener-class>org.springframework.web.context.ContextLoaderListener</listener-class>
</listener>
<!-- 设置contextConfigLocation的参数，指明初始化spring bean路径 -->
<context-param>
    <param-name>contextConfigLocation</param-name>
    <param-value>classpath:applicationContext.xml</param-value>
</context-param>

<servlet>  
    <servlet-name>dispatcherServlet</servlet-name> 
    <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
	<init-param>
       <param-name>contextConfigLocation</param-name>
       <param-value>classpath:springmvc.xml</param-value>
    </init-param>
    <load-on-startup>1</load-on-startup>
</servlet>
<servlet-mapping>
  	<servlet-name>dispatcherServlet</servlet-name>
  	<url-pattern>/</url-pattern>
</servlet-mapping>
```

- **applicationContext.xml**  --- 配置spring bean扫描，数据库、事务等配置

```xml
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:context="http://www.springframework.org/schema/context"
    xmlns:aop="http://www.springframework.org/schema/aop"
    xmlns:tx="http://www.springframework.org/schema/tx"
    xsi:schemaLocation="http://www.springframework.org/schema/beans
        http://www.springframework.org/schema/beans/spring-beans-4.2.xsd
        http://www.springframework.org/schema/context
       http://www.springframework.org/schema/context/spring-context-4.2.xsd
       http://www.springframework.org/schema/aop
       http://www.springframework.org/schema/aop/spring-aop-4.2.xsd
       http://www.springframework.org/schema/tx
       http://www.springframework.org/schema/tx/spring-tx-4.2.xsd">
     <!-- 配置组件扫描器，使用注解方式开发,不用配置dao和service -->
      <!-- 在springmvc.xml文件中也可以配置这个属性 -->  
     <context:component-scan base-package="com.edu.test"/>
      
    <!-- 数据源 -->
    <bean id="dataSource"
        class="org.springframework.jdbc.datasource.DriverManagerDataSource">
        <property name="driverClassName" value="com.mysql.jdbc.Driver" />
        <property name="url" value="jdbc:mysql://localhost:3306/test" />
        <property name="username" value="root" />
        <property name="password" value="" />
    </bean>
    
    <!-- 配置session工厂 -->
    <bean id="sqlSessionFactory" class="org.mybatis.spring.SqlSessionFactoryBean">
        <property name="dataSource" ref="dataSource"/>
        <property name="configLocation" value="classpath:mybatis-config.xml"/>
    </bean>
    
    <!-- 事务管理器 -->
    <bean id="transactionManager" class="org.springframework.jdbc.datasource.DataSourceTransactionManager">
        <property name="dataSource" ref="dataSource"/>
    </bean>
    
    <!-- 配置AOP通知 -->
    <tx:advice id="txAdvice" transaction-manager="transactionManager">
         <!-- 配置事务属性 -->
         <tx:attributes>
             <!-- 添加事务管理的方法 -->
             <tx:method name="save*" propagation="REQUIRED"/>
             <tx:method name="delete*" propagation="REQUIRED"/>
             <tx:method name="update*" propagation="REQUIRED"/>
             <tx:method name="select*" read-only="true"/>
         </tx:attributes>
     </tx:advice>
     
     <!-- 配置AOP，为添加事务管理的操作配置AOP -->
    <aop:config>
        <!-- 引入的Spring定义的事务通知，需要使用aop:advisor -->
        <!-- 下面难 -->
        <aop:advisor advice-ref="txAdvice"
            pointcut="execution(* com.edu.test.service.*.*(..))"
        />
    </aop:config>
</beans>
```

- springmvc.xml文件: 主要配置Controller的组件扫描器和视图解析器

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"  
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:p="http://www.springframework.org/schema/p" 
    xmlns:context="http://www.springframework.org/schema/context" 
    xmlns:mvc="http://www.springframework.org/schema/mvc" 
    xmlns:task="http://www.springframework.org/schema/task"
    xsi:schemaLocation="
        http://www.springframework.org/schema/beans 
        http://www.springframework.org/schema/beans/spring-beans-4.2.xsd 
        http://www.springframework.org/schema/context 
        http://www.springframework.org/schema/context/spring-context-4.2.xsd 
        http://www.springframework.org/schema/mvc 
        http://www.springframework.org/schema/mvc/spring-mvc-4.2.xsd 
        http://www.springframework.org/schema/task 
        http://www.springframework.org/schema/task/spring-task-4.2.xsd">    
        
        <!-- 使用注解开发，不用配置controller，需要配置一个组件扫描器 -->  
        <context:component-scan base-package="com.edu.test.controller"/>
    	<!-- 自动注册RequestMappingHandlerMapping与RequestMappingHandlerAdapter两个Bean,SpringMVC为@Controller分发请求所需 -->
        <mvc:annotation-driven/>
    
        <!-- JSP视图解析器 -->                
        <bean class="org.springframework.web.servlet.view.InternalResourceViewResolver">
            <!-- 配置从项目根目录到指定目录一端路径 ,建议指定浅一点的目录-->
            <property name="prefix" value="/WEB-INF/jsp/"></property>
            <!-- 文件的后缀名 -->
            <property name="suffix" value=".jsp"></property>
        </bean>
</beans>
```



## 4、SpringBoot