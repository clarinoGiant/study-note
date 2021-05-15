# 一、Spring

## 1. xxFactoryBean.java

功能：

### 1.1 查找结果

仅列举部分

![image-20210506234432508](Spring类命名规律.assets/image-20210506234432508.png)

代码样例，实现FactoryBean接口

```java
public class EarlyInitFactoryBean implements FactoryBean<String> {

public class EhCacheFactoryBean extends CacheConfiguration implements FactoryBean<Ehcache>, BeanNameAware, InitializingBean
```



## 2. xxBeanPostProcessor

### 2.1 查找结果

​	仅列举部分

![image-20210507234353249](Spring类命名规律.assets/image-20210507234353249.png)



## 3. xxAware

### 3.1 查找结果

- ApplicationContextAware
- BeanFactoryAware：
- BeanNameAware：

![image-20210507234538124](Spring类命名规律.assets/image-20210507234538124.png)

## 4. xxAnnotationparser.java



![image-20210507234722961](Spring类命名规律.assets/image-20210507234722961.png)

## 5. EnableXXX

### 查找结果

![image-20210507234822191](Spring类命名规律.assets/image-20210507234822191.png)

### 实现原理



## 6. xxxBeanFactory







# 二、SpringBoot

## 1. xxProperties.java

定义在application.properties参数的定义和默认值

### 1.1 查找结果

![image-20210506235015197](Spring类命名规律.assets/image-20210506235015197.png)

如RedisProperties内容

![image-20210506235046022](Spring类命名规律.assets/image-20210506235046022.png)