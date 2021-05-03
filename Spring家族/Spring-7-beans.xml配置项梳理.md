# 一、application.xml样例

``` xml
<?xml version="1.0" encoding="UTF-8" ?>
<beans xmlns="http://www.springframework.org/schema/beans"
	   xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns:tx="http://www.springframework.org/schema/tx"
	   xmlns:context="http://www.springframework.org/schema/context"
	   xmlns:p="http://www.springframework.org/schema/p" xmlns:aop="http://www.springframework.org/schema/aop"
	   xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans-4.0.xsd
    http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context-4.0.xsd
    http://www.springframework.org/schema/tx http://www.springframework.org/schema/tx/spring-tx-4.0.xsd http://www.springframework.org/schema/aop http://www.springframework.org/schema/aop/spring-aop.xsd">
	<!-- 扫描com.smart.dao包下所有标注@Repository的DAO组件 -->
    <context:component-scan base-package="com.smart.dao"/>
    <context:property-placeholder location="classpath:jdbc.properties"/>

	<bean id="dataSource"
		class="org.apache.commons.dbcp.BasicDataSource"
		destroy-method="close"
	    p:driverClassName="${jdbc.driverClassName}"
	    p:url="${jdbc.url}"
	    p:username="${jdbc.username}"
	    p:password="${jdbc.password}" />
	
	<bean id="sessionFactory"
		class="org.springframework.orm.hibernate4.LocalSessionFactoryBean">
		<property name="dataSource" ref="dataSource" />
	    <property name="packagesToScan">
            <list>
                <value>com.smart.domain</value>
            </list>
        </property>
		<property name="hibernateProperties">
			<props>
				<prop key="hibernate.dialect">
					org.hibernate.dialect.MySQLDialect
				</prop>
				<prop key="hibernate.show_sql">true</prop>
				<prop key="hibernate.cache.use_second_level_cache">true</prop>
				<prop key="hibernate.cache.region.factory_class">
					org.hibernate.cache.ehcache.EhCacheRegionFactory
				</prop>
				<prop key="hibernate.cache.use_query_cache">false</prop>
			</props>
		</property>
	</bean>
	<bean id="hibernateTemplate"
		class="org.springframework.orm.hibernate4.HibernateTemplate"
		 p:sessionFactory-ref="sessionFactory" />

	<bean id="transactionManager"
		  class="org.springframework.orm.hibernate4.HibernateTransactionManager">
		<property name="sessionFactory" ref="sessionFactory" />
	</bean>

    <!--  如下tx:advice+aop是spring事务相关配置（使用AOP技术）,对指定包路径下不同方法配置不同事务级别 -->
	<tx:advice id="txAdvice" transaction-manager="transactionManager">
		<tx:attributes>
			<tx:method name="*" propagation="REQUIRED" read-only="true"/>
			<tx:method name="create*" propagation="REQUIRED" read-only="false" />
			<tx:method name="save*" propagation="REQUIRED" read-only="false" />
			<tx:method name="reg*" propagation="REQUIRED" read-only="false" />
			<tx:method name="update*" propagation="REQUIRED" read-only="false" />
			<tx:method name="delete*" propagation="REQUIRED" read-only="false" />
		</tx:attributes>
	</tx:advice>
	<aop:config>
		<aop:advisor id="managerTx" advice-ref="txAdvice" pointcut="execution(* com.smart.*.*(..)))" order="1"/>
	</aop:config>
</beans>
```



# 二、<beans 级别

- default-lazy-init：true（项目启动时不会实例化注解的bean，除非启动项目时需要用到）
- 



# 三、<context 全局

## 1. component-scan

​	扫bean路径

## 2. property-placeholder

​	指定依赖的外部配置文件

​	<context:property-placeholder location="classpath:jdbc.properties"/>



# 四、<bean





# 五、<aop 切面





# 六、<tx 事务增强