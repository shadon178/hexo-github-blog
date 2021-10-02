---
title: Feign全局配置和局部配置
tags: feign
categories: java
---

# Feign全局配置和局部配置

在使用Feign的时候需要注意是否为全局配置和局部配置，否则可能会导致各种奇怪异常，而且在调试的是否也不容易看出问题。全局配置和局部配置的差别比较微妙，稍不注意可能会出问题。因此，这里总结给大家，防止大家也出现异常的问题。



### 全局配置：

```java
@Configuration
public class FeignConfiguration {

    @Bean
    public FeignBasicAuthRequestInterceptor basicAuthRequestInterceptor() {
        return new FeignBasicAuthRequestInterceptor();
    }

}
```

全局配置时**不要**在@FeignClient中使用configuration属性，虽然这样也不会抛错，这里一定要注意。

```java
@FeignClient(
    value = StandardServiceAPI.SERVICE_NAME,
    url = "${spring.gateway.host}:${spring.gateway.port}",
    configuration = FeignConfiguration.class
)
```



注意：

1. 全局配置的时候，就不需要在具体的FeignClient类中再配置configuration属性了。
2. 全局配置会使得所有的@FeignClient类都生效，因为@configuration会自动注册到Spring中，导致feign默认使用该配置。



### 局部配置：
``` java
public class ManagerFeignConfig {
    @Bean
    public RequestInterceptor managerFeignInterceptor() {
        return new ManagerFeignInterceptor();
    }
}
```


``` java
@FeignClient(value = StandardServiceAPI.SERVICE_NAME,
    url = "${spring.gateway.host}:${spring.gateway.port}",
    configuration = ManagerFeignConfig.class)
public interface StandardServiceClient {

    @PutMapping(StandardServiceAPI.STANDARD_PROCESS_APPLY_STATUS)
    ResultData<Void> updateProcessRequestStatus(
        @Valid @RequestBody
            UpdateProcessRequestStatusDto updateProcessRequestStatusDto);

}
```

注意：

1. `ManagerFeignConfig `类不要加任何Spring注解，防止注入到Spring中，导致所有FeignClient都生效。
2. FeignClient会自动将`ManagerFeignConfig `类注入到Spring容器并且只应用到具体的FeignClient类中，因此，也可以在`ManagerFeignConfig`中使用@Value注解。