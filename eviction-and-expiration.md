# Eviction and Expiration

In this section, we will see :

* **Eviction** - How to limit the number of entries in the cache 
* **Expiration** - How to expire the entries after a well defined Time-to-Live \(TTL\)

## Embedded Mode {#embedded-mode}

To work on this lab, either use the project setup during the Initial Setup or create a new project based on the same **infinispan-embedded-archetype** archetype.

### Prepare the main class and run it

Open up the only Java main class in the project **JDGConsoleApp** and depending on how you are instantiating the CacheManager follow one of the two approachs show below.

#### Programatically

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

#### Declaratively

Open up the `infinispan.xml` file in the `src/main/resources` folder and overwrite it completely with the following XML snippet

```xml
<?xml version = "1.0"?>
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

### Rest of the code and execution {#rest-of-the-code-and-execution}

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

### Setup the JDG server in Domain mode

1. Create a new file `commands.cli` in `src/main/resources` folder and paste the contents as shown below:
   ```
   /profile=clustered/subsystem=datagrid-infinispan/cache-container=clustered/configurations=CONFIGURATIONS/local-cache-configuration=bounded:add()
   /profile=clustered/subsystem=datagrid-infinispan/cache-container=clustered/configurations=CONFIGURATIONS/local-cache-configuration=bounded/eviction=EVICTION:add(size=50,strategy=LRU,type=COUNT)
   /profile=clustered/subsystem=datagrid-infinispan/cache-container=clustered/configurations=CONFIGURATIONS/local-cache-configuration=bounded/expiration=EXPIRATION:add(interval=100,lifespan=10000)
   /profile=clustered/subsystem=datagrid-infinispan/cache-container=clustered/local-cache=boundedCache:add(configuration=bounded)
   reload --host=master
   ```
2. Ensure that no JDG is running with `jps` and run the command `mvn wildfly:run` in the root of the project
3. Run the command `mvn wildfly:execute-commands` to execute the CLI commands we placed in the file
4. Now leave the server running

### Prepare the main class and run it

The steps for preparing the main class are similar to that of this [section](/eviction-and-expiration.md#rest-of-the-code-and-execution). For the actual code in case you decided to cheat, use the following and place the code soon after the cacheManager definition:

```java
RemoteCache<String, String> remoteCache = cacheManager.getCache("boundedCache");

IntStream.rangeClosed(1, 100) .parallel().forEach(i -> remoteCache.put("key" + i, "value" + i));

logger.info("The size of the cache before expiration is : {}", remoteCache.size());

// Sleep beyond the lifespan of cache entries
Thread.sleep(11000);

logger.info("The size of the cache after expiration is : {}", remoteCache.size());
cacheManager.stop();
```



