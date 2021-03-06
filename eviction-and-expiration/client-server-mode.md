# Client-Server Mode

To work on this lab, either use the project setup during the Initial Setup or create a new project based on the same **infinispan-server-client-archetype** archetype.

Follow the steps below to setup the project further:

### Setup the JDG server in Domain mode {#setup-the-jdg-server-in-domain-mode}

1. Create a new file `commands.cli` in `src/main/resources` folder and paste the contents as shown below:

   ```bash
   /profile=clustered/subsystem=datagrid-infinispan/cache-container=clustered/configurations=CONFIGURATIONS/local-cache-configuration=bounded:add()
   /profile=clustered/subsystem=datagrid-infinispan/cache-container=clustered/configurations=CONFIGURATIONS/local-cache-configuration=bounded/memory=OBJECT:add(size=50, strategy=LRU)
   /profile=clustered/subsystem=datagrid-infinispan/cache-container=clustered/configurations=CONFIGURATIONS/local-cache-configuration=bounded/expiration=EXPIRATION:add(interval=100,lifespan=10000)
   /profile=clustered/subsystem=datagrid-infinispan/cache-container=clustered/local-cache=boundedCache:add(configuration=bounded)
   reload --host=master
   ```

2. Ensure that no JDG is running with `jps` and run the command `mvn wildfly:start` in the root of the project
3. Run the command `mvn wildfly:execute-commands` to execute the CLI commands we placed in the file
4. Execute the client application code as described below

### Prepare the main class and run it {#prepare-the-main-class-and-run-it-1}

The steps for preparing the main class are similar to that of this [section](https://vchintal.gitbook.io/jboss-data-grid-7-x-workshop/eviction-and-expiration/embedded-mode#rest-of-the-code-and-execution). For the actual code in case you decided to cheat, use the following and place the code soon after the cacheManager definition:

```java
RemoteCache<String, String> remoteCache = cacheManager.getCache("boundedCache");

IntStream.rangeClosed(1, 100) .parallel().forEach(i -> remoteCache.put("key" + i, "value" + i));
logger.info("The size of the cache before expiration is : {}", remoteCache.size());

// Sleep beyond the lifespan of cache entries
Thread.sleep(11000);

logger.info("The size of the cache after expiration is : {}", remoteCache.size());
cacheManager.stop();
```

### Shutdown Servers

Run the command `mvn wildfly:shutdown` to shutdown JDG servers running in Domain mode.

