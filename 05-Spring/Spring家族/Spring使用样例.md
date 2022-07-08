

# 一、org.springframework.context

ConfigurableApplicationContext



## 1.1 annotation 注解

### 1. Configuration & Bean	

```java
@Configuration
public class NacosConfigConfiguration { 
    @Bean
    public FilterRegistrationBean nacosWebFilterRegistration() {
        FilterRegistrationBean<NacosWebFilter> registration = new FilterRegistrationBean<>();
        registration.setFilter(nacosWebFilter());
        registration.addUrlPatterns("/v1/cs/*");
        registration.setName("nacosWebFilter");
        registration.setOrder(1);
        return registration;
    }
```

### 2. Conditional

```java
@Conditional(ConditionDistributedEmbedStorage.class)
@Bean
public FilterRegistrationBean transferToLeaderRegistration() {
```

## 1.2 ApplicationListener

```java
public interface ApplicationListener<E extends ApplicationEvent> extends EventListener {
    void onApplicationEvent(E event);
	static <T> ApplicationListener<PayloadApplicationEvent<T>> forPayload(Consumer<T> consumer) {
		return event -> consumer.accept(event.getPayload());
	}
}    
```





# 二、org.springframework.core

## 2.1 Ordered



```java
@Override
public int getOrder() {
    return HIGHEST_PRECEDENCE;
}
```

### PriorityOrdered（继承）

```java
public interface PriorityOrdered extends Ordered {
}
```



## 2.2 env.ConfigurableEnvironment



# 三、org.springframework.boot

## 3.1 SpringApplicationRunListener



```java
@Override
public void starting() {
        
@Override
public void environmentPrepared(ConfigurableEnvironment environment) {
    
@Override
public void contextPrepared(ConfigurableApplicationContext context) 
    
@Override
public void contextLoaded(ConfigurableApplicationContext context) 
    
@Override
public void started(ConfigurableApplicationContext context) {
    
@Override
public void running(ConfigurableApplicationContext context) {
    
@Override
public void failed(ConfigurableApplicationContext context, Throwable exception) {
    
    
```

## 3.2 SpringApplicationEvent



### 1. ApplicationEnvironmentPreparedEvent(派生)



