# Settings.xml

## 默认的中央仓库

**当构建一个Maven项目时，首先检查pom.xml文件以确定依赖包的下载位置，执行顺序如下：**

​	1、从本地资源库中查找并获得依赖包，如果没有，执行第2步。

​	2、从Maven默认中央仓库中查找并获得依赖包（http://repo1.maven.org/maven2/），如果没有，执行第3步。

​	3、如果在pom.xml中定义了自定义的远程仓库，那么也会在这里的仓库中进行查找并获得依赖包，如果都没有找到，那么Maven就会抛出异常。



## 阿里云仓库镜像

为提升效率，配置阿里云镜像从国内下载

mirrorOf代表了一个镜像的替代位置，例如central就表示代替官方的中央库。

```xml
<mirrors>  
    ...   
    <mirror>  
      <id>alimaven</id>  
      <name>aliyun maven</name>  
      <url>http://maven.aliyun.com/nexus/content/groups/public/</url>  
      <mirrorOf>central</mirrorOf>          
    </mirror>
</mirrors>
```

# 常用配置

## 编译配置



```xml
 <plugin>
    <groupId>org.apache.maven.plugins</groupId>
    <artifactId>maven-compiler-plugin</artifactId>
    <version>3.6.2</version>
    <configuration>
       <target>1.8</target>   
       <source>1.8</source>
       <encoding>utf-8</encoding>
    </configuration>
  </plugin>
```

## 打包配置





# 常用插件

## 1. maven-surefire-plugin

maven里执行测试用例的插件

```xml
 <build>
    <plugins>
        <plugin>
		   <groupId>org.apache.maven.plugins</groupId>
		   <artifactId>maven-surefire-plugin</artifactId>
		   <version>2.7.2</version>
		   <configuration>
		      <forkMode>once</forkMode>
		      <threadCount>10</threadCount>
		      <argLine>-Dfile.encoding=UTF-8</argLine>
		   </configuration>
		</plugin>
     </plugins>
</build>
```



