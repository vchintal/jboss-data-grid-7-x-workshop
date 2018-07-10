# Security

Security in JBoss Data Grid comes in many flavors:

* Protecting access and operations on the Cache/CacheManager with Role Based Access Control \(RBAC\)
* In Client-Server Mode, securing the communication line between the JDG client and the server with TLS
* In Client-Server Mode, securing the cluster \(JGroups\) communication with TLS and ensuring that only authorized nodes can join the cluster

In this lab we will be focussing on how to secure Cache/CacheManager with RBAC

## Embedded Mode

In the embedded mode, since the cache resides within the application, the security of the cache is driven by the security around the app itself. Since there are numerous ways to protect a webapp with its container security than console based app, we will refer to a demo your presenter will run using the quickstart that can be found [here](https://github.com/vchintal/secure-embedded-cache-quickstart).

In this demo we will secure a simple webapp with authentication/authorization and use that to drive the operations on the cache.

## Client-Server Mode

There is a significant change on the both the client side and the server side \(configuration\) to enable authentication.

On the **server** side:

1. The cache container should have a security and authorization enabled that also lists all the possible roles and their cache permissions
2. The cache that needs to be secured should have security and authorization enabled that lists all the roles that can be applied.
3. The role names that need to be mentioned should be a subset of what was mentioned at the cache container level Authentication should be enabled on the **hotrod-connector** with applicable SASL mechanisms

On the **client** side:

1. A SASL callback handler needs to be defined
2. Security/authentication needs to be enabled on the RemoteCacheManager configuration along with an instance of callback handler with user credentials for authentication

To work on this lab, either use the project setup during the Initial Setup or create a new project based on the same **infinispan-server-client-archetype** archetype.

Follow the steps below to setup the project further:

Setup the JDG server in Domain mode

1. Create a new file `commands.cli` in `src/main/resources` folder and paste the contents as shown below. This will create thee caches: persistentCache \(no eviction\), persistentCacheEviction and persistentPassivatedCache

   ```bash
   /profile=clustered/subsystem=datagrid-infinispan/cache-container=clustered/security=SECURITY:add()
   /profile=clustered/subsystem=datagrid-infinispan/cache-container=clustered/security=SECURITY/authorization=AUTHORIZATION:add(mapper=org.infinispan.security.impl.IdentityRoleMapper)
   /profile=clustered/subsystem=datagrid-infinispan/cache-container=clustered/security=SECURITY/authorization=AUTHORIZATION/role=writer:add(name=writer,permissions=[WRITE,READ,BULK_WRITE,BULK_READ])
   /profile=clustered/subsystem=datagrid-infinispan/cache-container=clustered/security=SECURITY/authorization=AUTHORIZATION/role=reader:add(name=reader,permissions=[READ,BULK_READ])
   /profile=clustered/subsystem=datagrid-infinispan/cache-container=clustered/security=SECURITY/authorization=AUTHORIZATION/role=admin:add(name=admin,permissions=[ADMIN])
   /profile=clustered/subsystem=datagrid-infinispan/cache-container=clustered/configurations=CONFIGURATIONS/distributed-cache-configuration=secure-cache-configuration:add()
   /profile=clustered/subsystem=datagrid-infinispan/cache-container=clustered/configurations=CONFIGURATIONS/distributed-cache-configuration=secure-cache-configuration/security=SECURITY:add()
   /profile=clustered/subsystem=datagrid-infinispan/cache-container=clustered/configurations=CONFIGURATIONS/distributed-cache-configuration=secure-cache-configuration/security=SECURITY/authorization=AUTHORIZATION:add(roles=[reader,writer],enabled=true)
   /profile=clustered/subsystem=datagrid-infinispan/cache-container=clustered/distributed-cache=secureCache:add(configuration=secure-cache-configuration)
   /profile=clustered/subsystem=datagrid-infinispan-endpoint/hotrod-connector=hotrod-connector/authentication=AUTHENTICATION:add(security-realm=ApplicationRealm)
   /profile=clustered/subsystem=datagrid-infinispan-endpoint/hotrod-connector=hotrod-connector/authentication=AUTHENTICATION/sasl=SASL:add(mechanisms=[DIGEST-MD5],qop=[auth],server-name=hotrodserver)
   :reload-servers
   ```

2. Ensure that no JDG is running with jpsand run the command `mvn wildfly:start` in the root of the project
3. Run the command `mvn wildfly:execute-commands` to execute the CLI commands we placed in the file
4. Now leave the server running

### Rest of the code and execution {#rest-of-the-code-and-execution}

The steps for preparing the main class \(JDGRemoteClientConsoleApp.java\) is pretty straighforward:

1. Use the following code snippet and overwrite the part of the code that builds the configuration.

   ```java
   ConfigurationBuilder builder = new ConfigurationBuilder();
   builder.security().authentication()
    .enable()
    .serverName("hotrodserver")
    .saslMechanism("DIGEST-MD5")
   .callbackHandler(new MyCallbackHandler("dguser","ApplicationRealm", "dguser1!".toCharArray()));
   Configuration configuration = builder.build();
   ```

2. All that is left for us to do is to put together the missing class `MyCallbackHandler.java` ideally in the same package as the main class and we are good to go.

   For the sake of convenience the code for `MyCallbackHandler.java` is pasted below.

   ```java
   import javax.security.auth.callback.Callback;
   import javax.security.auth.callback.CallbackHandler;
   import javax.security.auth.callback.NameCallback;
   import javax.security.auth.callback.PasswordCallback;
   import javax.security.auth.callback.UnsupportedCallbackException;
   import javax.security.sasl.AuthorizeCallback;
   import javax.security.sasl.RealmCallback;

   public class MyCallbackHandler implements CallbackHandler {
       final private String username;
       final private char[] password;
       final private String realm;

       public MyCallbackHandler(String username, String realm, char[] password) {
           this.username = username;
           this.password = password;
           this.realm = realm;
       }

       @Override
       public void handle(Callback[] callbacks) throws IOException, UnsupportedCallbackException {
           for (Callback callback : callbacks) {
               if (callback instanceof NameCallback) {
                   NameCallback nameCallback = (NameCallback) callback;
                   nameCallback.setName(username);
               } else if (callback instanceof PasswordCallback) {
                   PasswordCallback passwordCallback = (PasswordCallback) callback;
                   passwordCallback.setPassword(password);
               } else if (callback instanceof AuthorizeCallback) {
                   AuthorizeCallback authorizeCallback = (AuthorizeCallback) callback;
                   authorizeCallback.setAuthorized(
                           authorizeCallback.getAuthenticationID().equals(authorizeCallback.getAuthorizationID()));
               } else if (callback instanceof RealmCallback) {
                   RealmCallback realmCallback = (RealmCallback) callback;
                   realmCallback.setText(realm);
               } else {
                   throw new UnsupportedCallbackException(callback);
               }
           }
       }
   }
   ```

3. We will now create the following users \(shown at the end of the page\) by updating the `.properties` files in `src/main/resources/domain/configuration`

   If you were to set these users by hand you would run the following commands:

   ```bash
   $JDG_HOME/bin/add-user.sh -a -u dgreader -p dgreader1! -g reader
   $JDG_HOME/bin/add-user.sh -a -u dgwriter -p dgwriter1! -g writer
   $JDG_HOME/bin/add-user.sh -u admin -p redhat1! -g admin
   ```

   Since we are not directly working with the JDG server we would update the following files in the folder `src/main/resources/domain/configuration`

   1. application-users.properties by adding the following lines

      ```text
      dgreader=ea5f219eb5157a4fe4b3579884ce00fd
      dgwriter=faea545a8451518ccef37ff983a40e18
      ```

   2. application-roles.properties by adding the following lines

      ```text
      dgreader=reader
      dgwriter=writer
      ```

   3. mgmt-users.properties by adding the following lines

      ```text
      admin=2851c15b7f819875fdb05f0bd8666564
      ```

   4. mgmt-groups.properties by adding the following lines

      ```text
      admin=admin
      ```

4. Restart the JDG server after your updates and then continue to update the main class such that you get a handle on the `secureCache` and run program by pumping in several entries into the cache
5. Try changing the username/password in the code for the CacheManager configuration to see the behavior of the client execution

| Username | Password | Type of User | Group |
| :--- | :--- | :--- | :--- |
| dgreader | dgreader1! | application | reader |
| dgwriter | dgwriter1! | application | writer |
| admin | redhat1! | management | admin |

