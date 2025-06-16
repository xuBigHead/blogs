---
layout: post
title: Spring Context 缓存.md
categories: [cate1, cate2]
description: some word here
keywords: keyword1, keyword2
mermaid: false
sequence: false
flow: false
mathjax: false
mindmap: false
mindmap2: false
---
# Spring Context 缓存

## 概述

> @since spring 3.1



Spring在3.1版本提供了基于注解的缓存策略，主要依赖如下注解来实现缓存相关操作：

- @Cacheable：缓存存在，则使用缓存；不存在，则执行方法，并将结果塞入缓存；
- @CacheEvit：失效缓存；
- @CachePut：更新缓存；



注意事项：

- @Cacheable注解是基于Spring AOP实现的，因此和AOP一样，对于类内部的方法调用是无效的；
- 缓存的方法返回对象必须实现了Serializable接口；



## 实践应用

### 缓存依赖

添加如下依赖，主要是为了引入spring-context和redis的依赖。

```xml
<dependencies>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
    </dependency>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-data-redis</artifactId>
    </dependency>
</dependencies>
```



### 服务配置

添加缓存相关配置，指定使用redis作为缓存服务。

```yaml
spring:  
  cache:
    type: redis
  data:
    redis:
      host: 43.139.3.252
      port: 6379
      database: 1
      password: ******
```



### @EnableCaching注解

通过在Spring启动类上添加@EnableCaching注解来表示启用Spring的缓存相关功能。

```java
@EnableCaching
@SpringBootApplication
public class MyBlogApplication {
    public static void main(String[] args) {
        SpringApplication.run(MyBlogApplication.class, args);
    }
}
```



### CacheManager注入容器

CacheManager用于配置缓存序列化、过期时间等配置。

```java
@Bean
public RedisCacheManager cacheManager(RedisConnectionFactory factory) {
    RedisCacheConfiguration config = RedisCacheConfiguration.defaultCacheConfig()
        .serializeKeysWith(RedisSerializationContext.SerializationPair.fromSerializer(new StringRedisSerializer()))
        .serializeValuesWith(RedisSerializationContext.SerializationPair.fromSerializer(new GenericJackson2JsonRedisSerializer()))
        .disableCachingNullValues(); // 不缓存null

    return RedisCacheManager.builder(factory)
        .cacheDefaults(config)
        .build();
}
```



### @Cacheable注解

通过@Cacheable注解标注在public方法上，表示将该方法的返回值缓存到指定的缓存服务中。

```java
@Override
@Cacheable(value = "permission", key = "'id:' + #id")
public PermissionVO detail(Long id) {
    var permission = findPermissionById(id);
    return permissionAssembler.poToVO(permission);
}
```



```java
public class PermissionVO implements Serializable {
    // ...
}
```



## 源码解析

### 缓存注解

#### @EnableCaching

```java
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Import(CachingConfigurationSelector.class)
public @interface EnableCaching {

	boolean proxyTargetClass() default false;

	AdviceMode mode() default AdviceMode.PROXY;

	int order() default Ordered.LOWEST_PRECEDENCE;
}
```



#### @Cacheable



#### @CacheEvict



#### @CachePut



#### @Caching