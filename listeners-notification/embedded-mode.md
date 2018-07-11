# Embedded Mode

To work on this lab, either use the project setup during the Initial Setup or create a new project based on the same **infinispan-embedded-archetype**  archetype.

Testing listening of cache/cacheManager events cannot with done easily with simple code as you would have to listen and at the same time need another process put entries into the cache. Also, in Embedded mode the listening of events happens locally, that is, the listener only captures the updates happening to the local data container.

In our lab we will listen to both kind of events:

1. Cluster view \(Cache Manager\) events that let us know if a node joined or left the cluster
2. Entries added \(Cache\) event, that lets us know what entries got added to the cache

Below is the way one can attach a listener to a cache or to a cacheManager

```java
// Add listener to a cache 
cache.addListener(new ClusterListener());

// Add a listener to a cacheManager
cacheManager.addListener(new ClusterListener());
```

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

###  **ClusterListener.java**

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

Using JBDS, run 3 instances of the **JDGConsoleApp** and notice the output on their consoles


