# Initial Setup

Git clone the repo [https://github.com/vchintal/jboss-middleware-archetypes.git](https://github.com/vchintal/jboss-middleware-archetypes) either from command-line or use JBoss Developer Studio to clone the project into your workspace.

### From Command-Line

```bash
 git clone https://github.com/vchintal/jboss-middleware-archetypes.git
```

### From JBoss Developer Studio

1. Using the menu, choose File 🡒 Import 🡒 Maven 🡒 Checkout Maven Projects from SCM
2. Copy-paste the above URL, click next till the end
3. Right-click on **jboss-middleware-archetypes**, choose Run As 🡒 6 Maven Install

Follow the instructions below to further run setup instructions based on the operational mode of your choice.

### Catalogging the Archetypes

In some circumstances, the archetypes are installed but not cataloged and hence the JBoss Developer Studio \(JBDS\) cannot find them. If JBDS cannot find the archetypes, you cannot create a project from them using it. So here is some workaround:

1. Run the command `mvn archetype:crawl` on the command line
2. It should create a file `~/.m2/repository/archetype-catalog.xml` . That is on my system it created `/home/vchintal/.m2/repository/archetype-catalog.xml`. Make a note of it and move to next step
3. Using JBDS, navigate to, Window 🡒 Preferences 🡒 Maven 🡒 Archetypes 🡒 Add Local Catalog. Paste the full path to your archetype-catalog.xml and save it

![](/assets/local-archetype.png)

## Embedded Mode {#embedded-mode}

Using **JBoss Developer Studio **\(JBDS\) menu:

1. Choose File 🡒 New 🡒 Other 🡒 Maven 🡒 Maven Project and click **Next** two times
2. In the **Filter** type in `org.everythingjboss.mw` and ensure that **Include snapshot archetypes ** checkbox is selected
3. Select the **infinispan-embedded-archetype** and click **Next**
4. Fill in the groupId, artifactId and version for your new project and click on **Finish**
5. Right click on the only Java class name, **JDGConsoleApp**, in the package explorer and click Run As 🡒 Java Application
6. If the execution in \#5 didn't fail for any reason, then the project worked. Your instructor will help you set the expectation around the output

## Client-Server Mode {#client-server-mode}

### Download/Install the JDG server

Download the JDG server binary from the either the [Customer Portal](https://access.redhat.com/jbossnetwork/restricted/listSoftware.html?product=data.grid&downloadType=distributions) or from [developers.redhat.com](https://developers.redhat.com/download-manager/file/jboss-datagrid-7.1.0-server.zip)

1. Unzip the folder to a chosen path. Let us refer to the unzipped path ending with `jboss-datagrid-7.1.0-server` as `$JDG_HOME`. On Linux this can be achieved with the following command:

   ```
   export JDG_HOME=/path/to/jboss-datagrid-7.1.0-server
   ```

2. If using the **Customer Portal**, download and save the [Red Hat JBoss Data Grid 7.1.2 Server Update](https://access.redhat.com/jbossnetwork/restricted/softwareDownload.html?softwareId=56221). Run the server update by following the steps below

   ```
   # Start the JDG server in one terminal with the following command
   $JDG_HOME/bin/standalone.sh 

   # In another terminal with JDG_HOME environment variable set, run 
   # the CLI command 
   $JDG_HOME/bin/cli.sh --connect --controller=127.0.0.1:9990

   # Once logged into CLI run the following command
   [standalone@127.0.0.1:9990 /] patch apply /path/to/jboss-datagrid-7.1.1-server-patch.zip

   # Then shutdown the JDG instance 
   [standalone@127.0.0.1:9990 /] shutdown
   ```

Using **JBoss Developer Studio **\(JBDS\) menu:

1. Choose File 🡒 New 🡒 Other 🡒 Maven 🡒 Maven Project and click **Next** two times
2. In the **Filter** type in `org.everythingjboss.mw` and ensure that **Include snapshot archetypes ** checkbox is selected
3. Select the **infinispan-server-client-archetype** and click **Next**
4. Fill in the groupId, artifactId and version for your new project and click on **Finish**
5. Right click on the only Java class name, **JDGConsoleApp**, in the package explorer and click Run As 🡒 Java Application
6. If the execution in \#5 didn't fail for any reason, then the project worked. Your instructor will help you set the expectation around the output



