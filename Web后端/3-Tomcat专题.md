# web.xml加载顺序

​	context-param->listener -> filter -> servlet

# web.xml内容

## 1. ServletContext初始化参数



```xml
<context-param>
    <param-name>name</param-name>
    <param-value>value</param-value>
    <description>descript</description>
</context-param>
```

## 2. 会话配置

```xml
<session-config>
    <!--session会话超时时间，单位为分钟-->
    <session-timeout>30</session-timeout>
</session-config>
```

## 3. Servlet声明及映射

在向servlet或JSP页面制定初始化参数或定制URL时，必须首先命名servlet或JSP页面。Servlet元素就是用来完成此项任务的

servlet + servlet-mapping

```xml
<servlet>
    <servlet-name>myServlet</servlet-name>
    <!--拦截后的处理类-->
    <servlet-class>com.rabbit.tomcat.Hello</servlet-class>
    <!--初始化参数-->
    <init-param>
        <param-name>name</param-name>
        <param-value>value</param-value>
    </init-param>
    <!--大于0表示项目启动的时候初始化-->
    <load-on-startup>1</load-on-startup>
</servlet>

<servlet-mapping>
    <servlet-name>myServlet</servlet-name>
    <!--可以配置多个拦截路径-->
    <url-pattern>/*</url-pattern>
    <url-pattern>*.do</url-pattern>
</servlet-mapping>
```

## 4. 应用声明周期监听器

监听器用于监听应用的请求，如果配置多个监听器，那么请求会依次经过监听器，多个监听器就形成了监听器链。

```xml
<listener>
    <!--监听器出例类，必须实现javax.servlet.ServletContextListener接口-->
    <listener-class>com.rabbit.tomcat.Hello</listener-class>
</listener>
```

## 5. Filter定义及映射

Filter用于配置web应用过滤器，用于过滤资源请求及响应。比如增加安全消息头

```xml
<filter>
    <filter-name>myFilter</filter-name>
    <!--必须实现javax.servlet.Filter接口-->
    <filter-class>com.rabbit.tomcat.Hello</filter-class>
    <!--初始化参数-->
    <init-param>
        <param-name>name</param-name>
        <param-value>value</param-value>
    </init-param>
</filter>
```

## 6. 欢迎文件列表

```xml
<welcome-file-list>
    <welcome-file>index.html</welcome-file>
    <welcome-file>index.jsp</welcome-file>
</welcome-file-list>
```

## 7. 错误页面

```xml
<error-page>
    <error-code>404</error-code>
    <location>/404.html</location>
</error-page>
<error-page>
    <exception-type>java.lang.RuntimeException</exception-type>
    <location>/error.jsp</location>
</error-page>
```

