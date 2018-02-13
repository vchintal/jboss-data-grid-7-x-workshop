# Prerequisites

## **Java Development Kit 8 ** installed

* Environment variable `JAVA_HOME` is set to the path where JDK is installed. On Linux this could be done with the following command:

  ```
  # Shown as an example, please use the right JDK installation path
  export JAVA_HOME=/usr/java/jdk1.8.0_144
  ```

* `bin` folder of Java home \(JAVA\_HOME\) is on the path. On Linux:
  ```
  export PATH=$JAVA_HOME/bin:$PATH
  ```
* Verify the setup by running the command `java -version` on the command-line

## **Maven **installed

* Environment variable `M2_HOME` set to path where Maven is installed. On Linux this could be done with the following command:

  ```
  # Shown as an example, please use the right Maven installation path
  export M2_HOME=/usr/share/maven
  ```

* `bin` folder of Maven's home \(M2\_HOME\) is on the path On Linux:
  ```
  export PATH=$M2_HOME/bin:$PATH
  ```
* Verify the setup by running the command `mvn -version` on the command-line

## **JBoss Developer Studio ** installed

* Download the binary as `.jar` from [here](https://developers.redhat.com/products/devstudio/download/)
* Run the installation by executing the following command:
  ```
  java -jar /path/to/devstudio-11.2.0.GA-installer-standalone.jar
  ```



