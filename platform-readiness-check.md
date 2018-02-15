# Prerequisites

## **Java Development Kit 8 ** installed

* Environment variable `JAVA_HOME` is set to the path where JDK is installed.

  * On Linux \(feel free to put this in your .bashrc or .kshrc\) this could be done with the following command:

    ```
    # Shown as an example, please use the right JDK installation path
    export JAVA_HOME=/usr/java/jdk1.8.0_144
    ```

  * On Windows

    * Search for **Environment Variables** from the Desktop. If that cannot be done, navigate to Control Panel 🡒 System and Security 🡒 System 🡒 \(click on\) Advanced System Settings 🡒 Advanced \(tab\) 🡒 \(click on\) Environment Variables 
    * In the window that came up, under the section_ _**User variables for**, click on the New and add **JAVA\_HOME **as the the variable and the path as the value

* `bin` folder of Java home \(JAVA\_HOME\) is on the path.

  * On Linux \(feel free to put this in your .bashrc or .kshrc\) :

    ```
    export PATH=$JAVA_HOME/bin:$PATH
    ```

  * On Windows

    * Following the same steps as above for JAVA\_HOME but this time as the last step in the window that got launched, work with the section titled **System Variables**. In that section find the variable by the name **Path** and click open it

    * Edit the value by prepending `%JAVA_HOME%\bin;` to the beginning of the line

* Verify the setup by running the command `java -version` on the command-line. On Windows, you have to launch a new command-line session.

## **Maven **installed

* Environment variable `M2_HOME` set to path where Maven is installed.

  * On Linux this could be done with the following command:

    ```bash
    # Shown as an example, please use the right Maven installation path
    export M2_HOME=/usr/share/maven
    ```

  * On Windows, follow the same steps as for JAVA_\_\_HOME and this time use the varibale_ _M2_\_\_HOME and set value to the maven's installed path

* `bin` folder of Maven's home \(M2\_HOME\) is on the path:

  * On Linux:

    ```bash
    export PATH=$M2_HOME/bin:$PATH
    ```

  * On Windows, follow the same steps as you followed to put the `%JAVA_HOME%\bin` on the path

* Verify the setup by running the command `mvn -version` on the command-line. On Windows, you have to launch a new command-line session.

## **JBoss Developer Studio ** installed

* Download the binary as `.jar` from [here](https://developers.redhat.com/products/devstudio/download/)
* Run the installation by executing the following command:
  ```bash
  java -jar /path/to/devstudio-11.2.0.GA-installer-standalone.jar
  ```
* Launch the JBoss Developer Studio and as a ideal optional step, create a new worspace by clicking on the menu File 🡒 Switch Workspace and type in the desired path and name of the workspace. This is a way to protect your current projects if any in the current workspace.



