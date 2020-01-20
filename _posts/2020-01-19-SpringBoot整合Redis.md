---
layout:     post
title:      SpringBoot整合Redis
subtitle:   Redis安装部署、SpringBoot整合
date:       2020-01-19
author:     yym439
header-img: img/post-bg-keybord.jpg
catalog: true
tags:
    - SpringBoot 
---

## 1. Dockert安装部署Redis

```
# 拉取 redis 镜像
> docker pull redis

# 运行 redis 容器
> docker run --name myredis -d -p 6379:6379 redis  --requirepass "mypassword"

# 执行容器中的 redis-cli，可以直接使用命令行操作 redis
> docker exec -it myredis redis-cli...
```

### 1.1 安装redis可视化客户端
[下载地址](https://github.com/qishibo/AnotherRedisDesktopManager/releases)


### 1.2 redis 常用命令

```
auth:密码认证
```
## 2. Springboot2.x整合redis + RedisTemplate

[参考资料](https://juejin.im/post/5dd88b9be51d45231005b564)

### 2.1 引入依赖

```
<dependencies>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-data-redis</artifactId>
    </dependency>
    <dependency>
        <groupId>org.apache.commons</groupId>
        <artifactId>commons-pool2</artifactId>
    </dependency>
</dependencies>

```

### 2.2 配置redis

```
spring:
  redis:
    host: 192.168.66.129
    port: 6379
    timeout: 3600ms
    database: 0
    lettuce:
      pool:
        min-idle: 0
        max-idle: 8
        max-wait: -1ms
        max-active: 8
    password: root
```

### 2.3 自定义redis配置类

```
@Configuration
@AutoConfigureAfter(RedisAutoConfiguration.class)
public class RedisCacheAutoConfiguration {
    @Bean
    public RedisTemplate<String, Serializable> redisCacheTemplate(LettuceConnectionFactory redisConnectionFactory) {
        RedisTemplate<String, Serializable> template = new RedisTemplate<>();
        template.setKeySerializer(new StringRedisSerializer());
        template.setValueSerializer(new GenericJackson2JsonRedisSerializer());
        template.setConnectionFactory(redisConnectionFactory);
        return template;
    }
}
```

> 注意：Spring Boot 的自动化配置，只能配置单机的 Redis ，如果是 Redis 集群，则所有的东西都需要自己手动配置

## 3. redis注解

### 3.1 引入依赖

```
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-cache</artifactId>
</dependency>
```

### 3.2 开启缓存注解

>在启动类上增加注解：@EnableCaching

> 开启缓存后，Spring Boot就会自动帮我们在后台配置一个RedisCacheManager,相关配置放在：RedisCacheConfiguration

### 3.3 注解使用

>@CacheConfig:注解在类上使用，用来描述该类中所有方法使用的缓存名称

>@Cacheable:注解一般加在查询方法上，表示将一个方法的返回值缓存起来

>@CachePut:注解一般加在更新方法上，当数据库中的数据更新后，缓存中的数据也要跟着更新，使用该注解，可以将方法的返回值自动更新到已经存在的key上

>@CacheEvict:注解一般加在删除方法上，当数据库中的数据删除后，相关的缓存数据也要自动清除

## 4.总结

> 在Spring Boot中，使用Redis缓存，既可以使用RedisTemplate自己来实现，也可以使用使用注解方式，注解方式是Spring Cache提供的统一接口，实现既可以是Redis，也可以是Ehcache或者其他支持这种规范的缓存框架。从这个角度来说，Spring Cache和Redis、Ehcache的关系就像JDBC与各种数据库驱动的关系。