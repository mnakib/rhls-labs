# Guided Exercise

0. Prepare the lab

`$ lab openshift-resources start`

`$ source /usr/local/etc/ocp4.config`


1. Login to the cluster and create a new project

`$ oc login -u ${RHT_OCP4_DEV_USER} -p ${RHT_OCP4_DEV_PASSWORD} ${RHT_OCP4_MASTER_API}`

`$ oc new-project ${RHT_OCP4_DEV_USER}-mysql-openshift`


2. Create a new application from the **mysql-persistent** template using the `oc-new app` command

`$ oc new-app --template=mysql-persistent -p MYSQL_USER=user -p MYSQL_PASSWORD=pass -p MYSQL_DATABASE=testdb -p MYSQL_ROOT_PASSWORD=rootPa$$ -p VOLUME_CAPACITY=10Gi`

`$ oc status`

`$ oc get pods`
```
NAME             READY   STATUS      RESTARTS   AGE
mysql-1-deploy   0/1     Completed   0          120s
**mysql-1-5vfn4 **   1/1     Running     0          100s
```


