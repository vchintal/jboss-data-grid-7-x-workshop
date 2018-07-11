# Client-Server Mode

To work on this lab, either use the project setup during the Initial Setup or create a new project based on the same **infinispan-server-client-archetype** archetype.

Follow the steps below to setup the project further:

### Setup the JDG server in Domain mode {#setup-the-jdg-server-in-domain-mode}

1. Create a new file`commands.cli` in `src/main/resources` folder and paste the contents as shown below:

   ```bash
   /profile=clustered/subsystem=datagrid-infinispan/cache-container=clustered/configurations=CONFIGURATIONS/replicated-cache-configuration=listener:add(mode="SYNC")
   /profile=clustered/subsystem=datagrid-infinispan/cache-container=clustered/configurations=CONFIGURATIONS/replicated-cache-configuration=listener/compatibility=COMPATIBILITY:add(enabled=true)
   /profile=clustered/subsystem=datagrid-infinispan/cache-container=clustered/replicated-cache=listenerCache:add(configuration=listener)
   :reload-servers
   ```

2. Ensure that no JDG is running with `jps` and run the command `mvn wildfly:start` in the root of the project
3. Run the command `mvn wildfly:execute-commands` to execute the CLI commands we placed in the file
4. Now leave the server running

### Prepare the main class and run it {#prepare-the-main-class-and-run-it}

The steps for preparing the main class \(JDGRemoteClientConsoleApp.java\) is pretty straighforward:

1. Add a new class \(ClusteredClientListener.java\) where the class is annotated with `@ClientListener` and with a  method declared `cacheEntryCreated(CacheEntryCreatedEvent<String> event)` annotated with `@ClientCacheEntryCreated` . This to say that this class can act as a listener and that it listens to `ClientCacheEntryCreated` events where the key is of type `String`
2. In JDGRemoteClientConsoleApp.java, instantiate a cache by the namel`listenerCache`
3. Add the created client listener \(ClusteredClientListener.java\) to the cache
4. Run the main class but suspend it somehow after the listener is attached 

#### Adding entries into the cache

Using a REST tool such as [RESTClient](https://addons.mozilla.org/en-US/firefox/addon/restclient/) submit a cache put with the following settings and verify the following output. If the REST interface requires authentication use `dgreader` for username and `dgreader1!` for password.

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

### Shutdown Servers

Run the command `mvn wildfly:shutdown` to shutdown JDG servers running in Domain mode.
