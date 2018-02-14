# Listeners and Notification

In this section we will see how to attach a listener class to listen to cache and/or cacheManager events .

## Embedded Mode {#embedded-mode}

To work on this lab, either use the project setup during the Initial Setup or create a new project based on the same **infinispan-embedded-archetype ** archetype.

Testing listening of cache/cacheManager events cannot with done easily with simple code as you would have to listen and at the same time need another process put entries into the cache. Also, in Embedded mode the listening of events happens locally, that is, the listener only captures the updates happening to the local data container.

To make things easy, we will provide you with the exact code to put in

1. JDGConsoleApp - The Java main class
2. ClusterListener - The Java class that acts as the Listener

Also, for simplicity we will go with the _programmatic_ way of defining cache configuration and with _UDP_ as the JGroups discovery protocol.

#### JDGConsoleApp.java

```java
import java.io.IOException;
import java.util.concurrent.CountDownLatch;
import java.util.stream.IntStream;

import org.apache.logging.log4j.LogManager;
import org.apache.logging.log4j.Logger;
import org.infinispan.Cache;
import org.infinispan.configuration.cache.CacheMode;
import org.infinispan.configuration.cache.Configuration;
import org.infinispan.configuration.cache.ConfigurationBuilder;
import org.infinispan.configuration.global.GlobalConfiguration;
import org.infinispan.configuration.global.GlobalConfigurationBuilder;
import org.infinispan.manager.DefaultCacheManager;
import org.infinispan.manager.EmbeddedCacheManager;

public class JDGConsoleApp {

    private static final Logger logger = LogManager.getLogger(JDGConsoleApp.class);

    public static void main(String[] args) throws IOException, InterruptedException {

        System.setProperty("java.net.preferIPv4Stack", "true");

        GlobalConfiguration globalConfig = new GlobalConfigurationBuilder()
                .transport()
                .defaultTransport()
                    .addProperty("configurationFile", "jgroups-udp.xml")
                .build();

        Configuration distConfig = new ConfigurationBuilder()
                .clustering()
                .cacheMode(CacheMode.DIST_SYNC)
                .build();

        CountDownLatch cdl = new CountDownLatch(1);

        EmbeddedCacheManager cacheManager = new DefaultCacheManager(globalConfig);
        cacheManager.defineConfiguration("distCache", distConfig);
        Cache<String, String> cache = cacheManager.getCache("distCache");

        cache.addListener(new ClusterListener());

        if (cacheManager.isCoordinator()) {
            logger.info("*** This is the coordinator instance ***");
            cacheManager.addListener(new ClusterListener(cdl));

            // Suspend execution here till there are three nodes in the cluster
            cdl.await();

            // Necessary to wait for clustering to do its magic before we put in 10 entries
            Thread.sleep(1000);

            // Put 10 entries in the clustered cache
            IntStream.range(1, 11).parallel().forEach( i -> cache.put("key"+i, "value"+i));
        }
    }
}
```

### ** ClusterListener.java**

```java
import java.util.concurrent.CountDownLatch;

import org.apache.logging.log4j.LogManager;
import org.apache.logging.log4j.Logger;
import org.infinispan.notifications.Listener;
import org.infinispan.notifications.cachelistener.annotation.CacheEntryCreated;
import org.infinispan.notifications.cachelistener.event.CacheEntryCreatedEvent;
import org.infinispan.notifications.cachemanagerlistener.annotation.ViewChanged;
import org.infinispan.notifications.cachemanagerlistener.event.ViewChangedEvent;

@Listener
public class ClusterListener {
    private static final Logger logger = LogManager.getLogger(ClusterListener.class);

    private static final int MAX_NODES_UP = 3;
    private CountDownLatch cdl;

    public ClusterListener() {
    }

    public ClusterListener(CountDownLatch cdl) {
        this.cdl = cdl;
    }

    @ViewChanged
    public void viewChanged(ViewChangedEvent event) {
        logger.info("The total # of nodes up so far is is : " + event.getNewMembers().size());
        if (event.getCacheManager().isCoordinator() && event.getNewMembers().size() >= MAX_NODES_UP) {
            this.cdl.countDown();
        } else
            logger.info("Still waiting for at least " + MAX_NODES_UP + " nodes to be up");
    }

    @CacheEntryCreated
    public void cacheEntryCreated(CacheEntryCreatedEvent<String, String> event) {
        if (!event.isPre()) {
            logger.info("Cache Entry created with key " + event.getKey());
        }
    }
}
```

### Execution

Using JBDS, run 3 instances of the **JDGConsoleApp **and notice the output on their consoles

## Client-Server Mode {#client-server-mode}

To work on this lab, either use the project setup during the Initial Setup or create a new project based on the same **infinispan-server-client-archetype** archetype.

Follow the steps below to setup the project further:

### Setup the JDG server in Domain mode {#setup-the-jdg-server-in-domain-mode}

1. Create a new file`commands.cli` in `src/main/resources` folder and paste the contents as shown below:
   ```
   /profile=clustered/subsystem=datagrid-infinispan/cache-container=clustered/configurations=CONFIGURATIONS/replicated-cache-configuration=listener:add(mode="SYNC")
   /profile=clustered/subsystem=datagrid-infinispan/cache-container=clustered/configurations=CONFIGURATIONS/replicated-cache-configuration=listener/compatibility=COMPATIBILITY:add(enabled=true)
   /profile=clustered/subsystem=datagrid-infinispan/cache-container=clustered/replicated-cache=listenerCache:add(configuration=listener)
   :reload-servers
   ```
2. Ensure that no JDG is running with `jps` and run the command `mvn wildfly:run` in the root of the project
3. Run the command `mvn wildfly:execute-commands` to execute the CLI commands we placed in the file
4. Now leave the server running

### Prepare the main class and run it {#prepare-the-main-class-and-run-it}

The steps for preparing the main class \(JDGRemoteClientConsoleApp.java\) is pretty straighforward:

1. Add a new class \(ClusteredClientListener.java\) where the class is annotated with `@ClientListener` and with a  method declared `cacheEntryCreated(CacheEntryCreatedEvent<String> event)` annotated with `@ClientCacheEntryCreated` . This to say that this class can act as a listener and that it listens to `ClientCacheEntryCreated` events where the key is of type `String`
2. In JDGRemoteClientConsoleApp.java, instantiate a cache by the namel`listenerCache`
3. Add the created client listener \(ClusteredClientListener.java\) to the cache
4. Run the main class but suspend it somehow after the listener is attached 

#### Adding entries into the cache

Using a REST tool such as [RESTClient](https://addons.mozilla.org/en-US/firefox/addon/restclient/) submit a cache put with the following settings and verify the output:

* URL: [http://127.0.0.1:8080/rest/listenerCache/1](http://127.0.0.1:8080/rest/listenerCache/1)
* Method: **PUT**
* Body: One

For your convenience, the code for the listener class and for the main program are provided below

#### JDGRemoteClientConsoleApp.java

```java
import java.io.IOException;
import java.util.concurrent.CountDownLatch;

import org.infinispan.client.hotrod.RemoteCache;
import org.infinispan.client.hotrod.RemoteCacheManager;
import org.infinispan.client.hotrod.configuration.Configuration;
import org.infinispan.client.hotrod.configuration.ConfigurationBuilder;

public class JDGRemoteClientConsoleApp {

    public static void main(String[] args) throws IOException, InterruptedException {
        Configuration configuration = new ConfigurationBuilder().build();
        CountDownLatch cdl = new CountDownLatch(1);
        
        RemoteCacheManager cacheManager = new RemoteCacheManager(configuration);
        RemoteCache<String, String> remoteCache = cacheManager.getCache("listenerCache");
        remoteCache.addClientListener(new ClusteredClientListener());
        cdl.await();
    }
}
```

#### ClusteredClientListener.java

```java
import org.infinispan.client.hotrod.annotation.*;
import org.infinispan.client.hotrod.event.ClientCacheEntryCreatedEvent;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

@ClientListener
public class ClusteredClientListener {
    private static final Logger logger =  LoggerFactory.getLogger(ClusteredClientListener.class);
    
    @ClientCacheEntryCreated
    public void cacheEntryCreated(ClientCacheEntryCreatedEvent<String> event) {
        logger.info("Cache Entry created with key " + event.getKey());
    }
}
```



