# Eviction and Expiration

In this section, we will see :

* **Eviction** - How to limit the number of entries in the cache 
* **Expiration** - How to expire the entries after a well defined Time-to-Live \(TTL\)

## Embedded Mode {#embedded-mode}

To work on this lab, either use the project setup during the Initial Setup or create a new project based on the same **infinispan-embedded-archetype** archetype.

Open up the only Java main class in the project **JDGConsoleApp** and depending on how you are instantiating the CacheManager follow one of the two approachs show below.

### Programatically

Copy-Paste the entire definition of the configuration and overwrite the one in the class file

```java
Configuration configuration = new ConfigurationBuilder()
        .clustering()
        .cacheMode(CacheMode.LOCAL)
        .eviction()
            .type(EvictionType.COUNT)
            .size(50)
            .strategy(EvictionStrategy.LIRS)
        .expiration()
            .lifespan(10, TimeUnit.SECONDS)
            .wakeUpInterval(100)
        .build();
```

And ensure that the cacheManager definition looks like the line below

```java
EmbeddedCacheManager cacheManager = new DefaultCacheManager("infinispan.xml");
```

### Declaratively

Open up the `infinispan.xml` file in the `src/main/resources` folder and overwrite it completely with the following XML snippet

```xml
<infinispan xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="urn:infinispan:config:8.2 http://www.infinispan.org/schemas/infinispan-config-8.2.xsd"
    xmlns="urn:infinispan:config:8.2">
    <cache-container default-cache="default">
        <local-cache-configuration name="local">
            <eviction type="COUNT" size="50" strategy="LIRS" />
            <expiration lifespan="10000" interval="100" />
        </local-cache-configuration>
        <local-cache name="boundedCache" configuration="local" />
    </cache-container>
</infinispan>
```

And ensure that the cacheManager definition looks like the line below

```java
EmbeddedCacheManager cacheManager = new DefaultCacheManager("infinispan.xml");
```

### Rest of the code and execution

Now that you have the cache manager configuration is in-place, all you need to do soon after the cacheManager definition is to add lines of code that

1. Instantiate a cache by the name `boundedCache`
2. Based on the type of &lt;K,V&gt; for the `boundedCache` insert 75 or 100 entries into the cache
3. Do a quick count of the cache entries
4. Force sleep the main thread for more than 10 seconds \(choose 11 or 12\)
5. Do a quick count of the cache entries again 

If you gave up of the how to do this all by yourself, feel free to cheat with the following code snippet

```java
Cache<String, String> cache = cacheManager.getCache("boundedCache");

IntStream.range(1, 76).parallel().forEach( i -> cache.put("key"+i, "value"+i));

// The size should be reported as 50 here as the rest 25 should be evicted out
logger.info("The size of the cache before expiration is : {}, mode of the cache is : {} ", cache.size(),
                cache.getCacheConfiguration().clustering().cacheMode());

Thread.sleep(11000);

// The size should be reported as 0 here as the entries should have expired by now
logger.info("The size of the cache after expiration is : {}, mode of the cache is : {} ", cache.size(),
                cache.getCacheConfiguration().clustering().cacheMode());
```

Run the Java application as you have done several times by now, either by JBDS or on the command line by running `mvn clean compile exec:exec`

## Client-Server Mode

To work on this lab, either use the project setup during the Initial Setup or create a new project based on the same ** infinispan-server-client-archetype ** archetype.

Follow the steps below to setup the project further:

1. Create a new file `commands.cli` in `src/main/resources` folder and paste the contents as shown below:
2. Ensure that no JDG is running with `jps` and run the command `mvn wildfly:run` in the root of the project
3. Run the command `mvn wildfly:execute-commands` to execute the CLI commands we placed in the file



