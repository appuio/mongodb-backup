# APPUiO Simple MongoDB Backup

## Overview

The MongoDB backup uses the 'scheduledjob' functionality of OpenShift to start a pod in regular intervals. The pod dumps the database to a persistent volume and exits.

This tool uses an existing container (RedHat MongoDB container) and overrides its command. There is no need to build a container.

## How to deploy the MongoDB backup

To set up the backup:
* Log in using `oc login`
* Switch to the right project using `oc project <yourproject>`

To create the scheduledjob, use
```
oc process -f mongodb-backup-template.yaml MONGODB_ADMIN_PASSWORD=passw0rd <more parameters> | oc create -f -
```

To get a list of parameters, use

```
oc process --parameters -f mongodb-backup-template.yaml
```

You can also store the template in the OpenShift project using
```
oc create -f mongodb-backup-template.yaml
```
After you did that, you can use
```
oc process mongodb-backup-template MONGODB_ADMIN_PASSWORD=passw0rd <more parameters> | oc create -f -
```

To disable the backup, you can simply remove the scheduledjob:
```
oc delete scheduledjob mongodb-backup
```

## Restore Database

To restore databases, copy it to the persistent volume of the database pod and use the following command:
```
mongorestore -u admin -p <mongodb-admin-password> --authenticationDatabase admin --gzip /var/lib/mongodb-backup/dump-YYYY-MM-DD-HH:MM:SS/<db-to-restore> -d <db-to-restore-into>
```

For most use cases 'db-to-restore' and 'db-to-restore-into' will be the same. You need to run the command multiple times with different arguments if you want to restore multiple databases. The command will restore all collections of the given database.

You should probably not restore the admin database, because that would change the password of the MongoDB 'admin' user, which is something you probably don't want to do.

A known issue is that some applications create invalid index definition files in the dump. You may need to manually remove the string ',"w":1' from the .json files that define the indexes of the corresponding collection.
