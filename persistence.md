# Persistence

In this section we will explore how to back up the cache with persitence store such that:

1. We achieve durability of the entries
2. We use the store as an overflow storage in case we cannot fit all of the entries in the memory 

In this lab we will try to different combination of cache configuration invoving persistence:

1. No eviction enabled
2. Eviction enabled
3. Eviction enabled with passivation

## Embedded Mode

To work on this lab, either use the project setup during the Initial Setup or create a new project based on the same ** infinispan-embedded-archetype**archetype.

As a first step lets alter the way in which the cache is configured. As before it depends on which approach we take, programmatic or declarative.

#### Programatically

Copy-Paste the entire definition of the configuration and overwrite the one in the class file

```java
Configuration config = new ConfigurationBuilder()
        .clustering()
        .cacheMode(CacheMode.DIST_SYNC)
        .eviction()
            .size(50)
            .strategy(EvictionStrategy.LIRS)
            .type(EvictionType.COUNT)
        .persistence()
            .addSingleFileStore()
                .maxEntries(5000)
                .location(System.getProperty("cacheStorePath"))
        .build();
```

#### Declaratively {#declaratively}

Open up the `infinispan.xml` file in the `src/main/resources` folder and overwrite it completely with the following XML snippet

```xml
<infinispan xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="urn:infinispan:config:8.4 http://www.infinispan.org/schemas/infinispan-config-8.4.xsd"
    xmlns="urn:infinispan:config:8.4">
    <jgroups>
        <stack-file name="external-file" path="jgroups-udp.xml" />
    </jgroups>
    <cache-container>
        <transport stack="external-file" />
        <jmx duplicate-domains="true" />
        <distributed-cache-configuration
            name="persistentCacheConfiguration" mode="SYNC"
            statistics-available="true" statistics="true">
            <eviction max-entries="50" type="COUNT" />
            <persistence passivation="true">
                <file-store path="${cacheStorePath}" max-entries="5000" />
            </persistence>
        </distributed-cache-configuration>
        <distributed-cache name="persistentCache" configuration="persistentCacheConfiguration" />
    </cache-container>
</infinispan>
```

## Client-Server Mode



