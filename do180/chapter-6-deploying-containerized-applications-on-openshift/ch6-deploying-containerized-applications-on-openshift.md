# Creating Kubernetes Resources


# Guided Exercise

0. Prepare the lab

`$ lab openshift-resources start`

`$ source /usr/local/etc/ocp4.config`


1. Login to the cluster and create a new project

`$ oc login -u ${RHT_OCP4_DEV_USER} -p ${RHT_OCP4_DEV_PASSWORD} ${RHT_OCP4_MASTER_API}`

`$ oc new-project ${RHT_OCP4_DEV_USER}-mysql-openshift`


2. Create a new application from the **mysql-persistent** template using the `oc-new app` command

`$ oc new-app --template=mysql-persistent -p MYSQL_USER=user -p MYSQL_PASSWORD=pass -p MYSQL_DATABASE=testdb -p MYSQL_ROOT_PASSWORD=rootpass -p VOLUME_CAPACITY=10Gi`

3. Verify that the MySQL pod was created successfully

`$ oc status`

`$ oc get pods`

`$ oc describe pod mysql-1-5vfn4`

`$ oc get svc`

`$ oc describe service mysql`

`$ oc get pvc`

`$ oc describe pvc/mysql`


4. Connect to the MySQL database server and verify that the database was created successfully.

`$ oc port-forward mysql-1-5vfn4 3306:3306`

`$ mysql -uuser -ppass --protocol tcp -h localhost`

`mysql> SHOW DATABASES;`
```
+--------------------+
| Database           |
+--------------------+
| information_schema |
| testdb             |
+--------------------+
```

`mysql> exit`

`Ctrl + C `


5. Delete the project to remove all the resources inside it.


`$ oc delete project ${RHT_OCP4_DEV_USER}-mysql-openshift`


6. Finish the lab






