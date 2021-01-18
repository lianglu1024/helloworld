

**Spring Boot Cache 存在以下问题：**

1. 生成 key 过于简单，容易冲突 switchCfgCache::3
2. 无法设置过期时间，默认过期时间为永久不过期
3. 无法配置序列化方式，默认的序列化是 JDK Serialazable

## RedisCache

https://www.jianshu.com/p/931484bb3fdc

```java
@Configuration
@EnableCaching
public class RedisConfig {

    @Value("${cache.default.expire-time}")
    private int defaultExpireTime;
    @Value("${cache.user.expire-time}")
    private int userCacheExpireTime;
    @Value("${cache.user.name}")
    private String userCacheName;

    /**
     * 缓存管理器
     *
     * @param
     * @return
     */
    @Primary
    @Bean
    public CacheManager cacheManager(RedisConnectionFactory RedisConnectionFactory) {
        RedisCacheConfiguration defaultCacheConfig = RedisCacheConfiguration.defaultCacheConfig();
        // 设置缓存管理器管理的缓存的默认过期时间
        defaultCacheConfig = defaultCacheConfig.entryTtl(Duration.ofSeconds(defaultExpireTime))
                // 设置 key为string序列化
                .serializeKeysWith(RedisSerializationContext.SerializationPair.fromSerializer(new StringRedisSerializer()))
                // 设置value为json序列化
                .serializeValuesWith(RedisSerializationContext.SerializationPair.fromSerializer(new GenericJackson2JsonRedisSerializer()))
                // 不缓存空值
                .disableCachingNullValues();

        Set<String> cacheNames = new HashSet<>();
        cacheNames.add(userCacheName);

        // 对每个缓存空间应用不同的配置
        Map<String, RedisCacheConfiguration> configMap = new HashMap<>();
        configMap.put(userCacheName, defaultCacheConfig.entryTtl(Duration.ofSeconds(userCacheExpireTime)));

        RedisCacheManager cacheManager = RedisCacheManager.builder(RedisConnectionFactory)
                .cacheDefaults(defaultCacheConfig)
                .initialCacheNames(cacheNames)
                .withInitialCacheConfigurations(configMap)
                .build();
        return cacheManager;
    }

}
```





## 参考

https://juejin.cn/post/6844903966615011335

https://www.cnblogs.com/Unicron/p/12499499.html