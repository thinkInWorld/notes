# 核心概念
Spring 缓存框架的核心就是将`缓存抽象应用到Java的方法上`，通过声明式缓存来透明地缓存方法的返回值。

> ！注意<br>
> 方法参数与返回值必须是具有一一对应的关系，也就是说不管什么时候，相同的参数调用相同的方法返回结果一定是一致的，否则就会出现脏数据的问题

对于开发者来说，只需要注意一下2点
- 缓存声明
- 缓存配置

# 注解说明
- `@Cacheable` 代表该方法的返回应该被缓存：触发缓存
- `@CacheEvict` 触发缓存清除：该策略的配置直接影响到是否出现脏数据
- `@CachePut` 触发更新缓存：
- `@Cache` 聚合多个缓存操作
- `@CacheConfig` 类级别上共享公用的缓存相关配置

```java
// 若缓存中存在相同的isbn值则不会执行方法而直接返回缓存结果： 缓存的名称叫做books
@Cacheable("books")
public Book findBook(ISBN isbn) {...}

//配置多个缓存
@Cacheable({"books", "isbns"})
public Book findBook(ISBN isbn) {...}

//自定义缓存Key【spel + customer interface】: 缓存的本质就是 K-V结果
@Cacheable(value="books", key="#isbn.rawNumber")
public Book findBook(ISBN isbn, boolean checkWarehouse, boolean includeUsed)

@Cacheable(value="books", key="T(someType).hash(#isbn)")
public Book findBook(ISBN isbn, boolean checkWarehouse, boolean includeUsed)

@Cacheable(value="books", keyGenerator="myKeyGenerator")
public Book findBook(ISBN isbn, boolean checkWarehouse, boolean includeUsed)
```

## 自定义缓存实现以及缓存条件
通过配置`CacheManager`来指定不同的`org.springframework.cache.interceptor.CacheResolver`实现类，默认实现是CacheResolver

```java
//指定cacheManager
@Cacheable(value="books", cacheManager="anotherCacheManager")
public Book findBook(ISBN isbn) {...}

//指定缓存实现
@Cacheable(cacheResolver="runtimeCacheResolver")
public Book findBook(ISBN isbn) {...}

//带条件的缓存
@Cacheable(value="book", condition="#name.length < 32", unless="#result.hardback")
public Book findBook(String name)
```
> https://docs.spring.io/spring/docs/4.1.x/spring-framework-reference/html/cache.html #Available caching SpEL evaluation context

## 更新缓存
通过获取执行方法的返回值来更新缓存从而能够让其他地方得到最新值
```java
@CachePut(value="book", key="#isbn")
public Book updateBook(ISBN isbn, BookDescriptor descriptor)
```
