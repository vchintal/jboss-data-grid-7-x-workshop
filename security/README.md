# Security

Security in JBoss Data Grid comes in many flavors:

* Protecting access and operations on the Cache/CacheManager with Role Based Access Control \(RBAC\)
* In Client-Server Mode, securing the communication line between the JDG client and the server with TLS
* In Client-Server Mode, securing the cluster \(JGroups\) communication with TLS and ensuring that only authorized nodes can join the cluster

In this lab we will be focussing on how to secure Cache/CacheManager with RBAC