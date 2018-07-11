# Embedded Mode

To work on this lab, either use the project setup during the Initial Setup or create a new project based on the same ** infinispan-embedded-archetype**archetype.

As a first step lets alter the way in which the cache is configured. As before it depends on which approach we take, programmatic or declarative.

#### Programatically

Copy-Paste the entire definition of the configuration and overwrite the one in the class file

```java
// Use a path on your filesystem
System.setProperty("cacheStorePath", "/home/vchintal/jdg");

GlobalConfiguration globalConfig = new GlobalConfigurationBuilder().transport()
        .defaultTransport()
        .build();

// Build Configuration for DefaultCacheManager via Fluent API
Configuration cacheConfig = new ConfigurationBuilder()
        .clustering()
        .cacheMode(CacheMode.DIST_SYNC)
        .memory()
            .storageType(StorageType.OBJECT)
            .size(50)
        .persistence()
            //.passivation(true)
            .addSingleFileStore()
                .maxEntries(5000)
                .location(System.getProperty("cacheStorePath"))
        .build();

// Use the Configuration to instantiate CacheManager
EmbeddedCacheManager cacheManager = new DefaultCacheManager(globalConfig,cacheConfig);
```

#### Declaratively {#declaratively}

Open up the `infinispan.xml` file in the `src/main/resources` folder and overwrite it completely with the following XML snippet

```xml
<infinispan xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="urn:infinispan:config:8.5 http://www.infinispan.org/schemas/infinispan-config-8.5.xsd"
    xmlns="urn:infinispan:config:8.5">
    <jgroups>
        <stack-file name="external-file" path="jgroups-udp.xml" />
    </jgroups>
    <cache-container>
        <transport stack="external-file" />
        <jmx duplicate-domains="true" />
        <distributed-cache-configuration
            name="persistentCacheConfiguration" mode="SYNC"
            statistics-available="true" statistics="true">
            <memory>
                <object size="50"/>
            </memory>
            <persistence passivation="true">
                <file-store path="${cacheStorePath}" max-entries="5000" />
            </persistence>
        </distributed-cache-configuration>
        <distributed-cache name="persistentCache" configuration="persistentCacheConfiguration" />
    </cache-container>
</infinispan>
```

### Rest of the code and execution {#rest-of-the-code-and-execution}

Now that you have the cache manager configuration is in-place, all you need to do soon after the cacheManager definition is to add lines of code that

1. Instantiate a cache by the name `persistentCache`
2. Set a system property inside the class \(for convenience\) with name `cacheStorePath` and a unique value for each run of the class \(if you are launching multiple instances\)
3. Put 100 &lt;K,V&gt; pairs into the cache and do not stop the cacheManager as we would like to see what is in the memory

Run the same app with different cache configurations:

* No Eviction - Remove the eveiction portion of the cache configuration
* Eviction but no Passivation - Add \(if missing\) the eviction configuration. The result should look like the config above. Ensure that passivation is set to false
* Eviction with Passivation - Same as above but with passivation set to true

Use **JConsole** to peek into the guts of the application to see how many entries are retained in the memory.

#### Expected Outcome

* No Eviction** ** - There will be 100 entries in the cache and 100 entries stored in the store
* Eviction but no Passivation** - ** There will be 50 entries in the cache and 100 entries stored in the store
* Eviction with Passivation** ** - There will be 50 entries in the cache and the rest \(mutually exclusive set\) in the store




