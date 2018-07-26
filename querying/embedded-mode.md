# Embedded Mode

To work on this lab, either use the project setup during the Initial Setup or create a new project based on the same **infinispan-embedded-archetype** archetype.

For this lab we will use the programmatic approach of defining the cache configuration. 

### Add Embedded Query API to the project

For this step, we will replace `infinispan-core` dependency in **pom.xml** ofthe project with `infinispan-embedded-query`. Once done you should the following entry in the **pom.xml** file.

```xml
<dependency>
    <groupId>org.infinispan</groupId>
    <artifactId>infinispan-embedded-query</artifactId>
</dependency> 
```

### Add a Domain entity class to the project

Since we will be working with a cache where the entities will be of type **Person**, lets create a Java class for it with all the required annotations that will facilitate indexing/searching of the entities based on it. 

Please the definition below in the same package structure as you currently have for convenience.

```java
import org.hibernate.search.annotations.Analyze;
import org.hibernate.search.annotations.Field;
import org.hibernate.search.annotations.Index;
import org.hibernate.search.annotations.Indexed;

import com.google.gson.Gson;

@Indexed
public class Person {
    private Long id;
    private String firstName;
    private String lastName;
    
    private int age;

    public Person() {
        
    }
    
    public Person(Long id, String firstName, String lastName, int age) {
        this.age = age;
        this.firstName = firstName;
        this.lastName = lastName;
        this.id = id;
    }
    
    @Field
    public Long getId() {
        return id;
    }

    public void setId(Long id) {
        this.id = id;
    }

    @Field(index=Index.YES, analyze=Analyze.YES)
    public String getFirstName() {
        return firstName;
    }

    public void setFirstName(String firstName) {
        this.firstName = firstName;
    }

    @Field(index=Index.YES, analyze=Analyze.NO)
    public String getLastName() {
        return lastName;
    }

    public void setLastName(String lastName) {
        this.lastName = lastName;
    }

    @Field(index=Index.YES)
    public int getAge() {
        return age;
    }

    public void setAge(int age) {
        this.age = age;
    }

    public String toString() {
        Gson gson = new Gson();
        return gson.toJson(this);
    }
}
```


### Define cache configuration

#### Programatically

Copy-Paste the entire definition of the configuration and overwrite the one in the class file

```java
        // Build Configuration for DefaultCacheManager via Fluent API
        ConfigurationBuilder configurationBuilder = new ConfigurationBuilder();
        configurationBuilder.indexing()
            .index(Index.ALL)
                .addProperty("default.directory_provider", "ram")
                .addProperty("lucene_version", "LUCENE_CURRENT");
        Configuration configuration =  configurationBuilder.build();
```

### Create cache entries and execute queries on them

Copy-Paste the following snippet between the cache manager instantiation and before its stoppage at the last line. For the sake of convience we are providing three different types of queries for this lab. Toggle the commented queries and test it out.

```java
        // Get the cache instance
        Cache<Long, Person> cache = cacheManager.getCache();
        cache.put(new Long(1), new Person(new Long(1), "Hillary", "Clinton", 70));
        cache.put(new Long(2), new Person(new Long(2), "Bill", "Clinton", 71));
        cache.put(new Long(3), new Person(new Long(3), "Chelsea", "Clinton", 38));
        
        QueryFactory qf = Search.getQueryFactory(cache);
        
        // Field >, <, =, >=, <=, !=
        //Query q = qf.create("from org.everythingjboss.simple_embedded_query_app.Person where age > 50");
        
        // Range Query
        //Query q = qf.create("from org.everythingjboss.simple_embedded_query_app.Person where age :  [20 to 70]");
        
        // Wildcard Query
        Query q = qf.create("from org.everythingjboss.simple_embedded_query_app.Person where firstName : '*ill*'");
        
        List<Person> persons = q.list();
        persons.stream().forEach(p -> {logger.info(p.toString());});
```

### Expected Outcome

For each query you should be able to verify from the generated output that the results only include the filtered objects based on the criteria defined.

