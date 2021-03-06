# Introduction and Agenda

JBoss Data Grid \(JDG in short\) is an In-Memory Enterprise Caching solution from Red Hat, Inc. JDG is :

* A distributed &lt;K,V&gt; \(key-value\) based store where the contents live in-memory of JDG instances  
* Allows for large volumes of data \(hundreds of GBs\) to be stored in a cluster
* Elastic scaling and redistribution of data as more instances are added or removed from the cluster
* Querying functionality to query stored values based on their attributes when key\(s\) are unavailable
* Role Based Access Control \(RBAC\) on cache entities which clearly define _who are allowed to do what operations on the cache_
* Distributed execution of tasks \(such a Map/Reduce\) over cluster content ensuring the tasks are migrated and executed on JDG instances where the data resides
* and much more ...

With this workshop you will gain hands-on experience developing applications which use **JDG** for its caching needs. This workshop covers most of the common and major features of **JDG**, however it doesn't cover all of them and the content can change without notice.

## Workshop Agenda {#workshop-agenda}

1. Platform readiness check 
2. Initial setup 
3. Eviction and Expiration 
4. Clustering
5. Listeners and notification 
6. Persistence 
7. Security 
8. Querying 
9. Apache Camel with JDG
10. JDG on Openshift 3.x

