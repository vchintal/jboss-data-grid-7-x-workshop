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

## Client-Server Mode

To work on this lab, either use the project setup during the Initial Setup or create a new project based on the same **infinispan-server-client-archetype** archetype.

Follow the steps below to setup the project further:

### Setup the JDG server in Domain mode {#setup-the-jdg-server-in-domain-mode}

1. Create a new file `commands.cli` in `src/main/resources` folder and paste the contents as shown below. This will create thee caches: persistentCache \(no eviction\), persistentCacheEviction and persistentPassivatedCache
   ```
   /profile=clustered/subsystem=datagrid-infinispan/cache-container=clustered/distributed-cache=persistentCache:add(configuration=persistent-file-store)
   /profile=clustered/subsystem=datagrid-infinispan/cache-container=clustered/configurations=CONFIGURATIONS/distributed-cache-configuration=persistent-file-store-eviction:add(mode=SYNC,start=EAGER)
   /profile=clustered/subsystem=datagrid-infinispan/cache-container=clustered/configurations=CONFIGURATIONS/distributed-cache-configuration=persistent-file-store-eviction/eviction=EVICTION:add(strategy="LRU",size="10000",type="COUNT")
   /profile=clustered/subsystem=datagrid-infinispan/cache-container=clustered/configurations=CONFIGURATIONS/distributed-cache-configuration=persistent-file-store-eviction/file-store=FILE_STORE:add(shared=false,passivation=false,fetch-state=true)
   /profile=clustered/subsystem=datagrid-infinispan/cache-container=clustered/distributed-cache=persistentCacheEviction:add(configuration=persistent-file-store-eviction)
   /profile=clustered/subsystem=datagrid-infinispan/cache-container=clustered/distributed-cache=persistentPassivatedCache:add(configuration=persistent-file-store-passivation)
   :reload-servers
   ```
2. Ensure that no JDG is running with `jps`and run the command `mvn wildfly:run` in the root of the project
3. Run the command `mvn wildfly:execute-commands` to execute the CLI commands we placed in the file
4. Now leave the server running

### Prepare the main class and run it {#prepare-the-main-class-and-run-it}

The steps for preparing the main class is pretty straighforward: 

1. Get handle on each of the three above mentioned caches 
2. Put 10500 entries into each 

For the sake of convenience the code is pasted below. Copy-paste the code between cacheManager instantiation and its stop.

```java
List<String> cacheNames = new ArrayList<String>(Arrays.asList("persistentCache","persistentCacheEviction","persistentPassivatedCache"));

for (String cacheName : cacheNames) {
    RemoteCache<String,String> cache = cacheManager.getCache(cacheName);
    IntStream.range(1, 10501).parallel().forEach( i -> {
        cache.put("key"+i, "value"+i);
    });         
    logger.info("The size of the {} cache is : {} ",cacheName, cache.size());
}
```

#### 

#### Expected Outcome

* No Eviction -  There will be 10500 entries in the `persistentCache` and 10500 entries stored in the store
* Eviction but no Passivation  - There will be 10000 entries in the `persistentCacheEviction`  and 10500 entries stored in the store
* Eviction with Passivation  - There will be 10000 entries in the `persistentPassivatedCache` and the rest \(mutually exclusive set\) 500 in the store

#### Where to note the size of the store\(s\) ? 

Navigate to the folders \(if you have only two servers up\) to find all the .dat files. Their size gives you an approximate feel for how many entries are store in it.

1. src/main/resources/domain/servers/server-one/data/datagrid-infinispan/clustered
2. src/main/resources/domain/servers/server-two/data/datagrid-infinispan/clustered



