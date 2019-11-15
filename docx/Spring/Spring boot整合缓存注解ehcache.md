###Spring boot整合缓存注解ehcache
#####依赖

```
       <dependency>
            <groupId>net.sf.ehcache</groupId>
            <artifactId>ehcache</artifactId>
            <version>2.9.1</version>
        </dependency>
```
#####新增ehcache.xml文件
在classpaath下增加`ehcache.xml`文件，内容如下：

```
<?xml version="1.0" encoding="UTF-8"?>
<ehcache updateCheck="false">
    <diskStore path="java.io.tmpdir"/>
<!--
       name:缓存名称。
       maxElementsInMemory：缓存最大个数。
       eternal:对象是否永久有效，一但设置了，timeout将不起作用。
       timeToIdleSeconds：设置对象在失效前的允许闲置时间（单位：秒）。
                仅当eternal=false对象不是永久有效时使用，可选属性，默认值是0，也就是可闲置时间无穷大。
       timeToLiveSeconds：设置对象在失效前允许存活时间（单位：秒）。最大时间介于创建时间和失效时间之间。
                    仅当eternal=false对象不是永久有效时使用，默认是0.，也就是对象存活时间无穷大。
       overflowToDisk：当内存中对象数量达到maxElementsInMemory时，Ehcache将会对象写到磁盘中。
       diskSpoolBufferSizeMB：这个参数设置DiskStore（磁盘缓存）的缓存区大小。默认是30MB。每个Cache都应该有自己的一个缓冲区。
       maxElementsOnDisk：硬盘最大缓存个数。
       diskPersistent：是否缓存虚拟机重启期数据 Whether the disk store persists between restarts 
                of the Virtual Machine. The default value is false.
       diskExpiryThreadIntervalSeconds：磁盘失效线程运行时间间隔，默认是120秒。
       memoryStoreEvictionPolicy：当达到maxElementsInMemory限制时，Ehcache将会根据指定的策略去清理内存。
                        默认策略是LRU（最近最少使用）。你可以设置为FIFO（先进先出）或是LFU（较少使用）。
       clearOnFlush：内存数量最大时是否清除。
    -->
    <defaultCache
            maxElementsInMemory="10000"
            eternal="false"
            timeToIdleSeconds="60"
            timeToLiveSeconds="60"
            overflowToDisk="true"
            maxElementsOnDisk="10000000"
            diskPersistent="false"
            diskExpiryThreadIntervalSeconds="120"
            memoryStoreEvictionPolicy="LRU"
            />

   <cache name="payCache"
           maxElementsInMemory="10000"
           eternal="false"
           timeToIdleSeconds="20"
           timeToLiveSeconds="20"
           overflowToDisk="false"
           diskPersistent="false"
           diskExpiryThreadIntervalSeconds="1"
    />
</ehcache>
```

其中`cache`的`name`属性要与注解中使用的`value`属性对应，不然会报错。

#####Spring配置文件applicationContext.xml需添加配置

```
<beans 
 xmlns:cache="http://www.springframework.org/schema/cache"
 xsi:schemaLocation=" 
 http://www.springframework.org/schema/cache 
 http://www.springframework.org/schema/cache/spring-cache-3.2.xsd"
 >

<!-- 启用缓存注解功能，这个是必须的，否则注解不会生效，另外，该注解一定要声明在spring主配置文件中才会生效 -->
<cache:annotation-driven cache-manager="ehcacheManager"/>
<!-- cacheManager工厂类，指定ehcache.xml的位置 -->
<bean id="ehcacheManagerFactory" class="org.springframework.cache.ehcache.EhCacheManagerFactoryBean">
<property name="configLocation" value="classpath:ehcache.xml" />
</bean>
<!-- 声明cacheManager -->
<bean id="ehcacheManager" class="org.springframework.cache.ehcache.EhCacheCacheManager">
<property name="cacheManager" ref="ehcacheManagerFactory" />
</bean>
```

####注解使用样例
#####@Cacheable注解

```
    @Cacheable(value = "payCache" ,key = "#id")
    public List<PayChannelDTO> getPayChannelCache(String id) {
        return payChannelRepository.getPayChannel();
    }
```
其中`value`与ecache.xml文件中的name对应，指定缓存库名，**必须指定**
`key`为缓存key，使用el表达式，此外还支持`condition`条件，如：

```
@Cacheable(value = "QY_api_productTop",key="#isbn.id",condition = "#isbn.id<10")
public Manual findManual(ISBN isbn, boolean checkWarehouse)
```
isbn.id小于10。

#####@CacheEvict 注解
- 清除value下缓存中所有的对象

```
@CacheEvict(value = "QY_api_productTop",allEntries=true) 
public void loadManuals(Long id)
```

- 清除指定key的缓存：

```
@CacheEvict(value = "QY_api_productTop", key = "#id")
public void loadManuals(Long id)
```


####Spring 2.5 to 3.1版本可能还依赖以下jar包

```
  <dependency>
      <groupId>com.googlecode.ehcache-spring-annotations</groupId>
      <artifactId>ehcache-spring-annotations</artifactId>
      <version>1.1.2</version>
  </dependency>
```


参考文档：https://my.oschina.net/lemonzone2010/blog/405202



###使用redis作为@Cacheable注解默认的缓存管理器
- appContext.txt中配置更换：

```
<!-- redis template definition  序列化方式 -->
    <bean id="stringKeySerializer" class="org.springframework.data.redis.serializer.StringRedisSerializer"/>


    <!-- Jedis连接池属性设置 可用默认方式-->
    <bean id="poolConfig" class="redis.clients.jedis.JedisPoolConfig">
        <!--最大空闲线程数-->
        <property name="maxIdle" value="${redis.pool.maxIdle}"/>
        <!--最大线程总数-->
        <property name="maxTotal" value="${redis.pool.maxTotal}"/>
        <!--获取连接时的最大等待毫秒数，默认-1-->
        <property name="maxWaitMillis" value="${redis.pool.maxWaitMillis}"/>
        <!--是否检测可用性，默认false-->
        <property name="testOnBorrow" value="${redis.pool.testOnBorrow}"/>
    </bean>

    <!--redis连接设置-->
    <bean id="jedisConnFactory"
          class="org.springframework.data.redis.connection.jedis.JedisConnectionFactory"
          p:usePool="true" p:hostName="${redis.connection.host}" p:port="${redis.connection.port}"
          p:password="${redis.connection.password}" p:database="${redis.connection.db}" p:poolConfig-ref="poolConfig"/>

    <!--redis设置-->
    <bean id="defaultRedisTemplate" class="org.springframework.data.redis.core.RedisTemplate"
          p:connectionFactory-ref="jedisConnFactory"
          p:keySerializer-ref="stringKeySerializer"
          p:hashKeySerializer-ref="stringKeySerializer" />

    <!--ehCache注解管理器使用redis作为缓存-->
    <bean id="redisCacheManager" class="org.springframework.data.redis.cache.RedisCacheManager">
        <constructor-arg name="redisOperations" ref="defaultRedisTemplate"/>
    </bean>

    <!--使用jedis连接池做缓存连接-->
    <bean id="stringRedisTemplate" class="org.springframework.data.redis.core.StringRedisTemplate"
          p:connectionFactory-ref="jedisConnFactory"/>

    <cache:annotation-driven cache-manager="redisCacheManager"/>
```

替换为上述配置后无需新增ehcache.xml文件，直接会在redis下生成指定名为value的缓存库缓存数据