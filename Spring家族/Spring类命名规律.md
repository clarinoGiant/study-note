# 一、Spring

## 1. xxFactoryBean.java

部分内容：

![image-20210506234432508](Spring类命名规律.assets/image-20210506234432508.png)

代码样例，实现FactoryBean接口

public class EarlyInitFactoryBean implements FactoryBean<String> {

public class EhCacheFactoryBean extends CacheConfiguration implements FactoryBean<Ehcache>, BeanNameAware, InitializingBean



## 2. xxBeanPostProcessor

部分查找结果

![image-20210507234353249](Spring类命名规律.assets/image-20210507234353249.png)



## 3. xxAware

查找结果如下：

其中

- ApplicationContextAware
- BeanFactoryAware：
- BeanNameAware：

![image-20210507234538124](Spring类命名规律.assets/image-20210507234538124.png)

## 4. xxAnnotationparser.java



![image-20210507234722961](Spring类命名规律.assets/image-20210507234722961.png)

## 5. EnableXXX

查找结果：

![image-20210507234822191](Spring类命名规律.assets/image-20210507234822191.png)





# 二、SpringBoot

## 1. xxProperties.java

​	查找application.properties参数的拼写

![image-20210506235015197](Spring类命名规律.assets/image-20210506235015197.png)

如RedisProperties内容

![image-20210506235046022](Spring类命名规律.assets/image-20210506235046022.png)