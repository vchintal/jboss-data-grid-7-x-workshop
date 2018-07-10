# Embedded Mode

To work on this lab, either use the project setup during the Initial Setup or create a new project based on the same  **infinispan-embedded-archetype**  archetype.

### Prepare the main class and run it

Open up the only Java main class in the project  **JDGConsoleApp**  and depending on how you are instantiating the CacheManager follow one of the two approachs show below.

#### Programatically

Copy-Paste the entire definition of the configuration and overwrite the one in the class file

```java
GlobalConfiguration globalConfig = new GlobalConfigurationBuilder()
        .transport()
        .defaultTransport()
            .addProperty("configurationFile", "jgroups-udp.xml")
        .build();

Configuration replConfig = new ConfigurationBuilder()
        .clustering()
        .cacheMode(CacheMode.REPL_SYNC)
        .build();

Configuration distConfig = new ConfigurationBuilder()
        .clustering()
        .cacheMode(CacheMode.DIST_SYNC)
        .build();
```

And ensure that the cacheManager definition looks like the line below

```java
EmbeddedCacheManager cacheManager = new DefaultCacheManager(globalConfig);
cacheManager.defineConfiguration("replCache", replConfig);
cacheManager.defineConfiguration("distCache", distConfig);
```

#### Declaratively

Open up the`infinispan.xml`file in the`src/main/resources`folder and overwrite it completely with the following XML snippet

```markup
<infinispan xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="urn:infinispan:config:8.5 http://www.infinispan.org/schemas/infinispan-config-8.5.xsd"
    xmlns="urn:infinispan:config:8.5">
    <jgroups>
        <stack-file name="external-file" path="jgroups-udp.xml" />
    </jgroups>
    <cache-container>
        <transport stack="external-file" />
        <distributed-cache name="distCache" mode="SYNC" />
        <replicated-cache name="replCache" mode="SYNC" />
    </cache-container>
</infinispan>
```

And ensure that the cacheManager definition looks like the line below

```java
EmbeddedCacheManager cacheManager = new DefaultCacheManager("infinispan.xml");
```

### Rest of the code and execution {#rest-of-the-code-and-execution}

Now that you have the cache manager configuration is in-place, all you need to do soon after the cacheManager definition is to add lines of code that:

1. Instantiate a cache by the name `replCache` or `distCache`
2. Based on the type of &lt;K,V&gt; cache, put into the cache about 100 entries if the `cacheManager.isCoordinator()` while also priniting that this node/JVM is the coordinator. **Note:**  if there is not a `cacheManager.stop()`as the end of the code the process will keep running, so keep it that way
3. Regardless of whether the `cacheManager.isCoordinator()` or not, print the total size of the cache
4. Start another process in parallel to the first one and notice what is displayed in the output

If you just need the code, here it is, paste it right after `cacheManager` definition:

```java
Cache<String, String> cache = cacheManager.getCache("distCache");

if (cacheManager.isCoordinator()) {
    logger.info("*** This node is the coordinator ***");
        IntStream.range(1, 101).parallel().forEach(i -> cache.put("key" + i, "value" + i));
}

logger.info("The size of the cache is : {}, mode of the cache is : {} ", cache.size(),
        cache.getCacheConfiguration().clustering().cacheMode());
```

