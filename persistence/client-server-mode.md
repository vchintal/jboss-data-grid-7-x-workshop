# Client-Server Mode

To work on this lab, either use the project setup during the Initial Setup or create a new project based on the same **infinispan-server-client-archetype** archetype.

Follow the steps below to setup the project further:

## Setup the JDG server in Domain mode {#setup-the-jdg-server-in-domain-mode}

1. Create a new file `commands.cli` in `src/main/resources` folder and paste the contents as shown below. This will create thee caches: persistentCache \(no eviction\), persistentCacheEviction and persistentPassivatedCache

   ```text
   /profile=clustered/subsystem=datagrid-infinispan/cache-container=clustered/distributed-cache=persistentCache:add(configuration=persistent-file-store)
   /profile=clustered/subsystem=datagrid-infinispan/cache-container=clustered/configurations=CONFIGURATIONS/distributed-cache-configuration=persistent-file-store-eviction:add(mode=SYNC,start=EAGER)
   /profile=clustered/subsystem=datagrid-infinispan/cache-container=clustered/configurations=CONFIGURATIONS/distributed-cache-configuration=persistent-file-store-eviction/memory=OBJECT:add(size=50, strategy=LRU)
   /profile=clustered/subsystem=datagrid-infinispan/cache-container=clustered/configurations=CONFIGURATIONS/distributed-cache-configuration=persistent-file-store-eviction/file-store=FILE_STORE:add(shared=false,passivation=false,fetch-state=true)
   /profile=clustered/subsystem=datagrid-infinispan/cache-container=clustered/distributed-cache=persistentCacheEviction:add(configuration=persistent-file-store-eviction)
   /profile=clustered/subsystem=datagrid-infinispan/cache-container=clustered/distributed-cache=persistentPassivatedCache:add(configuration=persistent-file-store-passivation)
   :reload-servers
   ```

2. Ensure that no JDG is running with `jps`and run the command `mvn wildfly:start` in the root of the project
3. Run the command `mvn wildfly:execute-commands` to execute the CLI commands we placed in the file
4. Execute the client application code as described below

## Prepare the main class and run it {#prepare-the-main-class-and-run-it}

The steps for preparing the main class is pretty straighforward:

1. Get handle on each of the three above mentioned caches 
2. Put 10500 entries into each cache

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

### Expected Outcome

* No Eviction -  There will be 10500 entries in the `persistentCache` and 10500 entries stored in the store
* Eviction but no Passivation  - There will be 10000 entries in the `persistentCacheEviction`  and 10500 entries stored in the store
* Eviction with Passivation  - There will be 10000 entries in the `persistentPassivatedCache` and the rest \(mutually exclusive set\) 500 in the store

### Where to note the size of the store\(s\) ?

Navigate to the folders \(if you have only two servers up\) to find all the .dat files. Their size gives you an approximate feel for how many entries are store in it.

1. src/main/resources/domain/servers/server-one/data/datagrid-infinispan/clustered
2. src/main/resources/domain/servers/server-two/data/datagrid-infinispan/clustered

### Shutdown Servers

Run the command `mvn wildfly:shutdown` to shutdown JDG servers running in Domain mode.


