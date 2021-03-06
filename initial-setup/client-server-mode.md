# Client-Server Mode

We will be creating a new JDG \(server\) client application based on one of the archetypes we installed in the last section. But before could run the application we need to setup the JDG server. Follow the steps shown below in order to finish the setup.

### Download/Install the JDG server {#download-install-the-jdg-server}

1. Download the JDG server binary from the either the [Customer Portal](https://access.redhat.com/jbossnetwork/restricted/listSoftware.html?product=data.grid&downloadType=distributions) or from [developers.redhat.com](https://developers.redhat.com/download-manager/file/jboss-datagrid-7.2.0-server.zip)​
2. Unzip the folder to a chosen path. Let us refer to the unzipped path ending with `jboss-datagrid-7.2.0-server` as `$JDG_HOME`. On Linux this can be achieved with the following command:
   ```bash
   export JDG_HOME=/path/to/jboss-datagrid-7.2.0-server
   ```

### Create a new JDG Client app {#create-a-new-jdg-client-app}

Using **JBoss Developer Studio** \(JBDS\) menu:

1. Choose File 🡒 New 🡒 Other 🡒 Maven 🡒 Maven Project and click Next two times
2. In the **Filter** type in `org.everythingjboss.mw` and ensure that **Include snapshot archetypes** checkbox is selected
3. Select the **infinispan-server-client-archetype** and click Next
4. Fill in the groupId, artifactId and version for your new project and click on Finish

### Run the client app {#run-the-client-app}

For the client app to be successful we must test against an running server, so let's follow the steps below in order to run the client application:

1. In the project folder, using command-line, ensure that the **JDG\_HOME** environment variable is set as shown above and run the command : `mvn wildfly:start` to start the JDG server
2. Once the above command finishes, run the Java main class **JDGRemoteClientConsoleApp** either from the JBDS by right-clicking on its name in package explorer and clicking Run As 🡒 Java Application or by running the command `mvn compile exec:exec` in the project's root folder.
3. If the command went thru without exceptions, then the project setup worked. Your instructor will help you set the expectation around the output
4. Shutdown the JDG servers running in domain mode by running the command `mvn wildfly:shutdown`  



