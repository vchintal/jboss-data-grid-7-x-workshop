# JDG on OpenShift

## Prepare OpenShift Environment

TODO: List all the ways OpenShift can be started locally

1. Start the OpenShift Cluster: The the following command to start the OpenShift Container Platform locally

   ```text
   oc cluster up --image registry.access.redhat.com/openshift3/ose  --version v3.7.23-3
   ```

2. Re-login as the system admin

   ```text
   oc login -u system:admin
   ```

3. Run the following command to deploy the JBoss Middleware image streams

   ```text
   oc create -f https://raw.githubusercontent.com/jboss-openshift/application-templates/master/jboss-image-streams.json -n openshift
   ```

4. Run the following command to deploy the **JBoss Data Grid** application template

   ```text
   oc create -f https://raw.githubusercontent.com/jboss-openshift/application-templates/master/datagrid/datagrid71-basic.json -n openshift
   ```

5. Using the **default** service account in the myproject namespace

   ```text
   oc policy add-role-to-user view system:serviceaccount:$(oc project -q):default -n $(oc project -q)
   ```

6. Use the eap-service-account in the myproject namespace

   ```text
   oc policy add-role-to-user view system:serviceaccount:$(oc project -q):eap-service-account -n $(oc project -q)
   ```

7. Log back in as **developer**

   ```text
   oc login -u developer
   ```

8. Create a new JDG app in the _myproject_ namespace

   ```text
   oc new-app --template=datagrid71-basic
   ```

9. Deploy the JDG client application which puts 100 entries into the `default` cache

   ```text
   oc new-app vchintal/s2i-java~https://github.com/vchintal/openshift-hotrod-console-client.git
   ```

   > NOTE: The last step runs a bit long as its doing Source-to-Image in OpenShift. Once built the image is pushed onto your local docker repository. So the next time you can use the command like `oc new-app --docker-image=172.30.1.1:5000/myproject/openshift-hotrod-console-client:latest` to deploy the image if for some reason you had to restart the Openshift server \(locally\)
   >
   > What would be ideal is that image be build locally and then pushed to OpenShift so that there is no need for a build in OpenShift. Hence this step will updated soon.

10. Verify the cache statistcs by visting the JDG POD and looking at the **Java Console** in the Details tab. Navigate to jboss.datagrid-infinispan 🡒 Cache 🡒 default\(dist\_sync\) 🡒 Clustered 🡒 Statistics and look at the _Number of entries_

