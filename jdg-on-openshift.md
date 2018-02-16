# JDG on OpenShift

## Prepare OpenShift Environment

TODO: List all the ways OpenShift can be started locally

1. Start the OpenShift Cluster: The the following command to start the OpenShift Container Platform locally

   ```
   oc cluster up --image registry.access.redhat.com/openshift3/ose  --version v3.7.23-3
   ```

2. Re-login as the system admin 
   ```
   oc login -u system:admin
   ```
3. Run the following command to deploy the JBoss Middleware image streams
   ```
   oc create -f https://raw.githubusercontent.com/jboss-openshift/application-templates/master/jboss-image-streams.json -n openshift
   ```

4. Run the following command to deploy the **JBoss Data Grid** application template
   ```
   oc create -f https://raw.githubusercontent.com/jboss-openshift/application-templates/master/datagrid/datagrid71-basic.json -n openshift
   ```
   
5. Using the **default** service account in the myproject namespace
   ```
   oc policy add-role-to-user view system:serviceaccount:$(oc project -q):default -n $(oc project -q)
   ```
7. Use the eap-service-account in the myproject namespace
   ```
   oc policy add-role-to-user view system:serviceaccount:$(oc project -q):eap-service-account -n $(oc project -q)
   ```
8. Log back in as **developer** 
   ```
   oc login -u developer
   ```
9. Create a new JDG app in the _myproject_ namespace
   ```
   oc new-app --template=datagrid71-basic
   ```


   
   


