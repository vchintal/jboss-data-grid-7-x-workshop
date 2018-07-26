# Client-Server Mode

To work on this lab, either use the project setup during the Initial Setup or create a new project based on the same **infinispan-server-client-archetype** archetype.

Follow the steps below to setup the project further:

### Setup the JDG server in Domain mode

1. Create a new file `commands.cli` in `src/main/resources` folder and paste the contents as shown below:

   ```bash
   /profile=clustered/subsystem=datagrid-infinispan/cache-container=clustered/distributed-cache=indexedCache:add(configuration=indexed)
   reload --host=master
   ```
2. Ensure that no JDG is running with `jps` and run the command `mvn wildfly:start` in the root of the project
3. Run the command `mvn wildfly:execute-commands` to execute the CLI commands we placed in the file
4. Execute the client application code as described below

### Add a Domain entity class to the project

Since we will be working with a cache where the entities will be of type **Person**, lets create a Java class for it with all the required annotations that will facilitate indexing/searching of the entities based on it. 

Please the definition below in the same package structure as you currently have for convenience.

```java
import java.util.ArrayList;

import org.infinispan.protostream.annotations.ProtoDoc;
import org.infinispan.protostream.annotations.ProtoField;

import com.google.gson.Gson;

@ProtoDoc("@Indexed")
public class Person {
    private Long id;
    private String firstName;
    private String lastName;
    private ArrayList<String> nicknames;
    
    private int age;

    public Person() {
        
    }
    
    public Person(Long id, String firstName, String lastName, ArrayList<String> nicknames, int age) {
        this.age = age;
        this.firstName = firstName;
        this.lastName = lastName;
        this.nicknames = nicknames;
        this.id = id;
    }
    
    @ProtoDoc("@Field")
    @ProtoField(number = 1)
    public Long getId() {
        return id;
    }

    public void setId(Long id) {
        this.id = id;
    }

    @ProtoDoc("@Field(index=Index.YES, analyze=Analyze.NO)")
    @ProtoField(number = 2)
    public String getFirstName() {
        return firstName;
    }

    public void setFirstName(String firstName) {
        this.firstName = firstName;
    }

    @ProtoDoc("@Field(index=Index.YES, analyze=Analyze.YES)")
    @ProtoField(number = 3)
    public String getLastName() {
        return lastName;
    }

    public void setLastName(String lastName) {
        this.lastName = lastName;
    }

    @ProtoDoc("@Field(index=Index.YES)")
    @ProtoField(number = 4, required = true)
    public int getAge() {
        return age;
    }

    public void setAge(int age) {
        this.age = age;
    }
    
    public ArrayList<String>  getNicknames() {
        return nicknames;
    }

    @ProtoDoc("@Field(index=Index.YES)")
    @ProtoField(number = 5)
    public void setNicknames(ArrayList<String> nicknames) {
        this.nicknames = nicknames;
    }

    public String toString() {
        Gson gson = new Gson();
        return gson.toJson(this);
    }
}
```

### Prepare the main class and run it 

Since of all the labs the querying lab has a long main class with many steps to configure the **cacheManager** and **remoteCache**, we will give away the entire code below with adequate comments/documentation on what is being done at each step. 

```java 
import java.io.IOException;
import java.util.ArrayList;
import java.util.List;
import java.util.stream.IntStream;

import org.infinispan.client.hotrod.RemoteCache;
import org.infinispan.client.hotrod.RemoteCacheManager;
import org.infinispan.client.hotrod.configuration.Configuration;
import org.infinispan.client.hotrod.configuration.ConfigurationBuilder;
import org.infinispan.client.hotrod.marshall.ProtoStreamMarshaller;
import org.infinispan.protostream.SerializationContext;
import org.infinispan.protostream.annotations.ProtoSchemaBuilder;
import org.infinispan.protostream.annotations.ProtoSchemaBuilderException;
import org.infinispan.client.hotrod.Search;
import org.infinispan.query.dsl.Query;
import org.infinispan.query.dsl.QueryFactory;
import org.infinispan.query.remote.client.ProtobufMetadataManagerConstants;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

public class JDGRemoteClientConsoleApp {

	private static final Logger logger = LoggerFactory.getLogger(JDGRemoteClientConsoleApp.class);

	public static void main(String[] args) throws ProtoSchemaBuilderException, IOException {

		// Build the cache configuration and instantiate a remote cache
		Configuration remoteConfig = new ConfigurationBuilder().addServers("127.0.0.1:11222")
		        .marshaller(new ProtoStreamMarshaller()).build();
		RemoteCacheManager cacheManager = new RemoteCacheManager(remoteConfig);
		RemoteCache<Long, Person> remoteCache = cacheManager.getCache("indexedCache");

		// Set ProtoStreamMarshaller as the default marshaller for serialization
		SerializationContext ctx = ProtoStreamMarshaller.getSerializationContext(cacheManager);

		// Generate the 'person.proto' schema file based on the annotations on
		// Person class and register it with the SerializationContext of the
		// client
		ProtoSchemaBuilder protoSchemaBuilder = new ProtoSchemaBuilder();
		String personSchemaFile = protoSchemaBuilder.fileName("person.proto").addClass(Person.class)
		        .build(ctx);

		// Register the schemas with the JDG server by placing the schema
		// into a
		// special cache
		// ProtobufMetadataManagerConstants.PROTOBUF_METADATA_CACHE_NAME
		RemoteCache<String, String> metadataCache = cacheManager
		        .getCache(ProtobufMetadataManagerConstants.PROTOBUF_METADATA_CACHE_NAME);
		metadataCache.put("person.proto", personSchemaFile);
		String errors = metadataCache.get(ProtobufMetadataManagerConstants.ERRORS_KEY_SUFFIX);
		if (errors != null) {
			throw new IllegalStateException("Protobuf schema file contain errors:\n" + errors);
		}

		logger.info("Putting entries into the cache");

		IntStream.rangeClosed(1, 100).forEach(i -> {
			Person p = new Person(new Long(i), "fn" + i, "ln" + i, new ArrayList<String>(), 15 + i);
			remoteCache.put(p.getId(), p);
		});

		logger.info("Done putting entries");
		
		String queryString = "from Person where lastName : '*3*'";
		QueryFactory qf = Search.getQueryFactory(remoteCache);
		Query q = qf.create(queryString);
		List<Person> persons = 	q.list();
		
		if (persons.size() != 0) {
			logger.info("Matching {} entries are  :", persons.size());
			persons.stream().forEach(p -> {logger.info(p.toString());});
		} else {
			logger.info("There were no matches for the queryString {} ",queryString);
		}

		// Stop the cache manager
		cacheManager.stop();
	}
}

```

### Shutdown Servers

Run the command `mvn wildfly:shutdown` to shutdown JDG servers running in Domain mode.
