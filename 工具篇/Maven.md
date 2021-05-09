

# 常用命令



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

maven里执行测试用例的插件，即可自动识别和运行src/test目录下利用该框架编写的测试用例。

跳过测试：<skipTests>true</skipTests>  ；忽略测试失败：<testFailureIgnore>true</testFailureIgnore>  

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

## 2. maven-compile-plugin 编译代码



```xml
<plugin>                                                                
    <!-- 指定maven编译的jdk版本,如果不指定,maven3默认用jdk 1.5 maven2默认用jdk1.3 -->                                             
    <groupId>org.apache.maven.plugins</groupId>                         
    <artifactId>maven-compiler-plugin</artifactId>                      
    <version>3.1</version>                                              
    <configuration>                                                                                                        
        <!-- 一般而言，target与source是保持一致的，但是，有时候为了让程序能在其他版本的jdk中运行(对于低版本目标jdk，源代码中不能使用低版本jdk中不支持的语法)，会存在target不同于source的情况 --> 
        <source>1.8</source> <!-- 源代码使用的JDK版本 -->               
        <target>1.8</target> <!-- 需要生成的目标class文件的编译版本 -->   
        <encoding>UTF-8</encoding><!-- 字符集编码 -->
        <skipTests>true</skipTests><!-- 跳过测试 -->                    
        <verbose>true</verbose>
        <showWarnings>true</showWarnings>                               
        <fork>true</fork><!-- 要使compilerVersion标签生效，还需要将fork设为true，用于明确表示编译版本配置的可用 -->                                              
        <executable><!-- path-to-javac --></executable><!-- 使用指定的javac命令，例如：<executable>${JAVA_1_4_HOME}/bin/javac</executable> -->           
        <compilerVersion>1.3</compilerVersion><!-- 指定插件将使用的编译器的版本 -->                                   
        <meminitial>128m</meminitial><!-- 编译器使用的初始内存 -->        
        <maxmem>512m</maxmem><!-- 编译器使用的最大内存 -->                
        <compilerArgument>-verbose -bootclasspath ${java.home}\lib\rt.jar</compilerArgument><!-- 这个选项用来传递编译器自身不包含但是却支持的参数选项 -->         
    </configuration>
</plugin>   
```

## 3. maven-assembly-plugin 多种风格打包

制作项目分发包，该分发包可能包含了项目的可执行文件、源代码、readme、平台脚本等等。 maven-assembly-plugin支持各种主流的格式如zip、tar.gz、jar和war等

支持自定义的打包结构，也可以定制依赖项等。

- maven-jar-plugin，默认的打包插件，用来打普通的project JAR包；
- maven-shade-plugin，用来打可执行JAR包，也就是所谓的fat JAR包；
- maven-assembly-plugin，支持自定义的打包结构，也可以定制依赖项等。

```xml
<build>
    <plugins>
        <plugin>
            <groupId>org.apache.maven.plugins</groupId>
            <artifactId>maven-assembly-plugin</artifactId>
            <version>${maven-assembly-plugin.version}<version>
            <executions>
                <execution>
                    <id>make-assembly</id>
                    <!-- 绑定到package生命周期 -->
                    <phase>package</phase>
                    <goals>
                        <!-- 只运行一次 -->
                        <goal>single</goal>
                    </goals>
                </execution>
            </executions>
            <configuration>
                <!-- 配置描述符文件 -->
                <descriptor>src/main/assembly/assembly.xml</descriptor>
                <!-- 也可以使用Maven预配置的描述符
                <descriptorRefs>
                    <descriptorRef>jar-with-dependencies</descriptorRef>
                </descriptorRefs> -->
            </configuration>
        </plugin>
    </plugins>
</build>
```

assembly插件的打包方式是通过descriptor（描述符）来定义的。
Maven预先定义好的描述符有bin，src，project，jar-with-dependencies等。比较常用的是jar-with-dependencies，它是将所有外部依赖JAR都加入生成的JAR包中，比较傻瓜化。
但要真正达到自定义打包的效果，就需要自己写描述符文件，格式为XML。

```xml
<assembly>
    <id>assembly</id>

    <formats>
        <format>tar.gz</format>
    </formats>

    <includeBaseDirectory>true</includeBaseDirectory>

    <fileSets>
        <fileSet>
            <directory>src/main/bin</directory>
            <includes>
                <include>*.sh</include>
            </includes>
            <outputDirectory>bin</outputDirectory>
            <fileMode>0755</fileMode>
        </fileSet>
        <fileSet>
            <directory>src/main/conf</directory>
            <outputDirectory>conf</outputDirectory>
        </fileSet>
        <fileSet>
            <directory>src/main/sql</directory>
            <includes>
                <include>*.sql</include>
            </includes>
            <outputDirectory>sql</outputDirectory>
        </fileSet>
        <fileSet>
            <directory>target/classes/</directory>
            <includes>
                <include>*.properties</include>
                <include>*.xml</include>
                <include>*.txt</include>
            </includes>
            <outputDirectory>conf</outputDirectory>
        </fileSet>
    </fileSets>

    <files>
        <file>
            <source>target/${project.artifactId}-${project.version}.jar</source>
            <outputDirectory>.</outputDirectory>
        </file>
    </files>

    <dependencySets>
        <dependencySet>
            <unpack>false</unpack>
            <scope>runtime</scope>
            <outputDirectory>lib</outputDirectory>
        </dependencySet>
    </dependencySets>
</assembly>
```

## 4. maven-dependency-plugin

http://maven.apache.org/components/plugins/maven-dependency-plugin/plugin-info.html

**典型场景：**

　　1**.**需要某个特殊的 jar包，但是有不能直接通过maven依赖获取，或者说在其他环境的maven仓库内不存在，那么如何将我们所需要的jar包打入我们的生产jar包中。

　　2.某个jar包内部包含的文件是我们所需要的，或者是我们希望将它提取出来放入指定的位置 ，那么除了复制粘贴，如何通过maven插件实现呢？

**dependency插件我们最常用到的是**

　　dependency:copy 

　　dependency:copy-dependencies：Goal that copies the project dependencies from the repository to a defined location.  

　　dependency:unpack 　　

　　dependency:unpack-dependencies 这四个

**如果要实现上述的两种场景，我们需要的 是 第一个和第三个。**

样例：

```xml
<plugin>
    <groupId>org.apache.maven.plugins</groupId>
    <artifactId>maven-dependency-plugin</artifactId>
    <version>2.10</version>
    <executions>
        <execution>
            <id>copy-dependencies</id>
            <phase>package</phase>
            <goals>
                <goal>copy-dependencies</goal>
            </goals>
            <configuration>
                <outputDirectory>${project.build.directory}/deploydependencis</outputDirectory>
                <overWriteReleases>false</overWriteReleases>
                <overWriteSnapshots>false</overWriteSnapshots>
                <overWriteIfNewer>true</overWriteIfNewer>
                <includeScope>compile</includeScope>
                <includeScope>runtime</includeScope>
            </configuration>
        </execution>
    </executions>
</plugin>
```

