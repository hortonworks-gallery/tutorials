---
layout: post
title: Securing HDFS, Hive and HBase with Knox and Ranger
date: '2015-09-08T12:34:00.001-07:00'
author: Saptak Sen
tags:
- ranger
- knox
- hive
- hbase
- security
- hadoop
modified_time: '2015-09-08T15:11:18.054-07:00'
---
### Introduction

Apache Ranger delivers a comprehensive approach to security for a Hadoop cluster. It provides central security policy administration across the core enterprise security requirements of authorization, accounting and data protection.

Apache Ranger already extends baseline features for coordinated enforcement across Hadoop workloads from batch, interactive SQL and real–time in Hadoop.

In this tutorial, we cover using Apache Ranger for HDP 2.3 to secure your Hadoop environment. We will walkthrough the following topics:

  1. Support for Knox authorization and audit
  2. Command line policies in Hive
  3. Command line policies in HBase
  4. REST APIs for policy manager

### Prerequisite

The only prerequisite for this tutorial is that you have [Hortonworks Sandbox](http://hortonworks.com/sandbox).

Once you have Hortonworks Sandbox, login through SSH:

![](http://www.dropbox.com/s/5xcvt1ymxee82yi/Screenshot%202015-09-02%2014.59.03.png?dl=1)

### Starting Knox Service and Demo LDAP Service

From the Ambari console at [http://localhost:8080/](http://localhost:8080/) (username and password is `admin` and `admin` respectively), select Knox from the list of `Services` on the left-hand side of the page.
![](http://www.dropbox.com/s/0vfvvobjunmzagg/Screenshot%202015-09-08%2010.17.05.png?dl=1)

Then click on `Service Actions` from the top right hand side of the page click on `Start`

![]http://www.dropbox.com/s/jhb30dgey8m30n6/Screenshot%202015-09-08%2010.27.06.png?dl=1)

From the following you can track the start of the Knox service to completion:

![](http://www.dropbox.com/s/wklkcfva0hml4m1/Screenshot%202015-09-08%2010.20.01.png?dl=1)

Then go back to the `Service Actions` button on the `Knox` service and click on `Start Demo LDAP`

![](https://www.dropbox.com/s/s8fj7rhcx6vb5ax/Screenshot%202015-09-08%2010.37.09.png?dl=1)

You can track the start of the `Demo LDAP Service` from the following screen:

![](http://www.dropbox.com/s/58n8ykihilijf43/Screenshot%202015-09-08%2010.39.01.png?dl=1)

### Knox access scenarios

Check if Ranger Admin console is running, at [http://localhost:6080/](http://localhost:6080/)from your host machine. The username is `admin` and the password is `admin`

![](http://www.dropbox.com/s/a7a5syeuzvww76s/Screenshot%202015-09-08%2010.11.01.png?dl=1)

If it is not running you can start from the command line using the command

```bash
sudo service ranger-admin start
```
![](https://www.dropbox.com/s/le04q38rvztcn0i/Screenshot%202015-09-02%2015.06.00.png?dl=1)

Click on sandbox_knox link under Knox section in the main screen of Ranger Administration Portal

![](https://www.dropbox.com/s/890vp59sfbmvqiq/Screenshot%202015-09-08%2010.30.58.png?dl=1)

You can review policy details by a clicking on the policy name.

![](http://www.dropbox.com/s/jqxlvtl1k7qjeu8/Screenshot%202015-09-08%2010.31.31.png?dl=1)

To start testing Knox policies, we would need to turn off the “global knox allow” policy.

![](http://www.dropbox.com/s/hpfd4vegvktk8i7/Screenshot%202015-09-08%2010.33.53.png?dl=1)

Locate `Sandbox for Guest` policy on the Ranger Admin console and edit the policy

![](http://www.dropbox.com/s/j4ovjilbkcxq7q4/Screenshot%202015-09-08%2010.45.17.png?dl=1)

and enable policy named “Sandbox for Guest”

![](http://www.dropbox.com/s/fbv85udmimjrn8t/Screenshot%202015-09-08%2010.47.28.png?dl=1)

From your local SSHd terminal (not directly on the Sandbox), run this CURL command to access WebHDFS

`curl -k -u admin:admin-password 'https://127.0.0.1:8443/gateway/knox_sample/webhdfs/v1?op=LISTSTATUS'`

![](http://www.dropbox.com/s/aezb4fat1v9whyx/Screenshot%202015-09-08%2010.50.43.png?dl=1)

Go to Ranger Policy Manager tool → Audit screen and check the knox access (denied) being audited.

![](http://www.dropbox.com/s/1al1gdd8q2ku17o/Screenshot%202015-09-08%2010.53.47.png?dl=1)

Now let us try the same CURL command using “guest” user credentials from the terminal

`curl -k -u guest:guest-password 'https://127.0.0.1:8443/gateway/knox_sample/webhdfs/v1?op=LISTSTATUS'`

![](http://www.dropbox.com/s/2vodca4dawz1ex5/Screenshot%202015-09-08%2010.55.29.png?dl=1)

    {"FileStatuses":{"FileStatus":[{"accessTime":0,"blockSize":0,"childrenNum":0,"fileId":16393,"group":"hadoop","length":0,"modificationTime":1439987528048,"owner":"yarn","pathSuffix":"app-logs","permission":"777","replication":0,"storagePolicy":0,"type":"DIRECTORY"},{"accessTime":0,"blockSize":0,"childrenNum":4,"fileId":16389,"group":"hdfs","length":0,"modificationTime":1439987809562,"owner":"hdfs","pathSuffix":"apps","permission":"755","replication":0,"storagePolicy":0,"type":"DIRECTORY"},{"accessTime":0,"blockSize":0,"childrenNum":1,"fileId":17000,"group":"hdfs","length":0,"modificationTime":1439989173392,"owner":"hdfs","pathSuffix":"demo","permission":"755","replication":0,"storagePolicy":0,"type":"DIRECTORY"},{"accessTime":0,"blockSize":0,"childrenNum":1,"fileId":16398,"group":"hdfs","length":0,"modificationTime":1439987529660,"owner":"hdfs","pathSuffix":"hdp","permission":"755","replication":0,"storagePolicy":0,"type":"DIRECTORY"},{"accessTime":0,"blockSize":0,"childrenNum":1,"fileId":16394,"group":"hdfs","length":0,"modificationTime":1439987528532,"owner":"mapred","pathSuffix":"mapred","permission":"755","replication":0,"storagePolicy":0,"type":"DIRECTORY"},{"accessTime":0,"blockSize":0,"childrenNum":2,"fileId":16396,"group":"hadoop","length":0,"modificationTime":1439987538099,"owner":"mapred","pathSuffix":"mr-history","permission":"777","replication":0,"storagePolicy":0,"type":"DIRECTORY"},{"accessTime":0,"blockSize":0,"childrenNum":1,"fileId":16954,"group":"hdfs","length":0,"modificationTime":1439988741413,"owner":"hdfs","pathSuffix":"ranger","permission":"755","replication":0,"storagePolicy":0,"type":"DIRECTORY"},{"accessTime":0,"blockSize":0,"childrenNum":3,"fileId":16386,"group":"hdfs","length":0,"modificationTime":1440165443820,"owner":"hdfs","pathSuffix":"tmp","permission":"777","replication":0,"storagePolicy":0,"type":"DIRECTORY"},{"accessTime":0,"blockSize":0,"childrenNum":8,"fileId":16387,"group":"hdfs","length":0,"modificationTime":1439988397561,"owner":"hdfs","pathSuffix":"user","permission":"755","replication":0,"storagePolicy":0,"type":"DIRECTORY"}]}}

We can check the auditing in the Ranger Policy Manager → Audit screen

![](http://www.dropbox.com/s/bo9i7o5nvbl8gv3/Screenshot%202015-09-08%2010.56.45.png?dl=1)

Ranger plugin for Knox intercepts any request made to Knox and enforces policies which are retrieved from the Ranger Administration Portal

You can configure the Knox policies in Ranger to restrict to a specific service (WebHDFS, WebHCAT etc) and to a specific user or a group and you can even bind user/group to an ip address

![](http://www.dropbox.com/s/ejdjh8z9k3e755u/Screenshot%202015-09-08%2010.59.43.png?dl=1)

### Hive grant/revoke permission scenarios

Ranger can support import of grant/revoke policies set through command line or Hue for Hive. Ranger can store these policies centrally along with policies created in the  administration portal and enforce it in Hive using its plugin.

![](http://www.dropbox.com/s/zl1v3se7rklb30h/Screenshot%202015-09-08%2011.01.23.png?dl=1)

As a first step, disable the global access policy for Hive in Ranger Administration Portal

![](http://www.dropbox.com/s/a3a1enxxs8s0a6x/Screenshot%202015-09-08%2011.03.30.png?dl=1)

Let us try running a Grant operation using user Hive from the command line. Login into beeline tool using the following command

`beeline -u "jdbc:hive2://sandbox.hortonworks.com:10000/default" -n it1 -p it1-d org.apache.hive.jdbc.HiveDriver`

![](http://www.dropbox.com/s/67mmhiczob5d581/Screenshot%202015-09-08%2011.05.44.png?dl=1)

Then issue the `GRANT` command

`grant select, update on table xademo.customer_details to user network1;`

You should see the following error:

![](http://www.dropbox.com/s/hyyfvmyib65ya2w/Screenshot%202015-09-08%2011.07.14.png?dl=1)

Let us check the audit log in the Ranger Administration Portal → Audit

![](http://www.dropbox.com/s/v3msdbodlnr7568/Screenshot%202015-09-08%2011.08.52.png?dl=1)

You can see that access was denied for an admin operation for user it1.

We can create a policy in Ranger for user ‘it1’ to be an admin. Create a new policy from the Ranger Admin console and ensure the configuration matches the illustration below

![](http://www.dropbox.com/s/b85fnw29td9rufb/Screenshot%202015-09-08%2012.57.48.png?dl=1)

We can try the beeline command again, once the policy has been saved.

`GRANT select, update on table xademo.customer_details to user network1;`

![](http://www.dropbox.com/s/6r9x08jwl52dwcc/Screenshot%202015-09-08%2013.00.35.png?dl=1)

If the command goes through successfully, you will see the policy created/updated in Ranger Admin Portal → Policy Manager. It checks if there is an existing relevant policy to update, else it creates a new one. 
![](http://www.dropbox.com/s/u0iulr8c9z4avia/Screenshot%202015-09-08%2013.02.33.png?dl=1)

What happened here?

Ranger plugin intercepts GRANT/REVOKE commands in Hive and creates corresponding policies in Admin portal. The plugin then uses these policies for enforcing Hive authorization (Hiveserver2)

Users can run further GRANT commands to update permissions and REVOKE commands to take away permissions.

### HBase grant/revoke permission scenarios

Ranger can support import of grant/revoke policies set through command line in Hbase. Similar to Hive, Ranger can store these policies as part of the Policy Manager and enforce it in Hbase using its plugin.

Before you go further, ensure HBase is running from Ambari – http://127.0.0.1:8080 (username and password are `admin`).

If it is not go to `Service Actions` button on top right and `Start` the service

![](http://www.dropbox.com/s/sil03yzmc5mfw9i/Screenshot%202015-09-08%2013.10.32.png?dl=1)

As a first step, let us try running a Grant operation using user Hbase.

Disable the public access policy “HBase Global Allow” in Ranger Administration Portal – policy manager

![](http://www.dropbox.com/s/2wentbjpxwx8i2x/Screenshot%202015-09-08%2013.13.08.png?dl=1)

Login into HBase shell as ‘it1’ user

    su - it1

    [it1@sandbox ~]$ hbase shell

![](http://www.dropbox.com/s/txhuqq9kihwgcy3/Screenshot%202015-09-08%2013.14.33.png?dl=1)


Run a grant command to give “Read”, “Write”, “Create” access to user mktg1 in table ‘iemployee’

`hbase(main):001:0> grant 'mktg1', 'RWC', 'iemployee'`

you should get a Acess Denied as below:

![](http://www.dropbox.com/s/jjlhj1wdh8juqnj/Screenshot%202015-09-08%2014.10.21.png?dl=1)

Go to Ranger Administration Portal → Policy Manager and create a new policy to assign “admin” rights to user it1

![](http://www.dropbox.com/s/j0j7e8fczyeh13f/Screenshot%202015-09-08%2014.13.46.png?dl=1)

Save the policy and rerun the HBase command again

    hbase(main):006:0> grant 'mktg1', 'RWC', 'iemployee'

    0 row(s) in 0.8670 seconds

![](http://www.dropbox.com/s/z8d8b4pugwjs8c1/Screenshot%202015-09-08%2014.14.27.png?dl=1)

Check HBase policies in the Ranger Policy Administration portal. The grant permissions were added to an existing policy for table ‘iemployee’ that we created in previous step

![](http://www.dropbox.com/s/wviys0tdrhe87yr/Screenshot%202015-09-08%2014.15.44.png?dl=1)

You can revoke the same permissions and the permissions will be removed from Ranger admin. Try this in the same HBase shell


    hbase(main):007:0> revoke 'mktg1', 'iemployee'

    0 row(s) in 0.4330 seconds


You can check the existing policy and see if it has been changed

![](http://www.dropbox.com/s/u9u56n8ra94q220/Screenshot%202015-09-08%2014.16.34.png?dl=1)

What happened here?

Ranger plugin intercepts GRANT/REVOKE commands in Hbase and creates corresponding policies in the Admin portal. The plugin then uses these policies for enforcing authorization

Users can run further GRANT commands to update permissions and REVOKE commands to take away permissions.

### REST APIs for Policy Administration

Ranger policies administration can be managed through REST APIs. Users can use the APIs to create or update policies, instead of going into the Administration Portal.

####Running REST APIs from command line

From your local command line shell, run this CURL command. This API will create a policy with the name “hadoopdev-testing-policy2” within the HDFS repository “sandbox_hdfs”

    curl -i --header "Accept:application/json" -H "Content-Type: application/json" --user admin:admin -X POST http://127.0.0.1:6080/service/public/api/policy -d '{ "policyName":"hadoopdev-testing-policy2","resourceName":"/demo/data/test","description":"Testing policy for /demo/data/test","repositoryName":"sandbox_hdfs","repositoryType":"HDFS","permMapList":[{"userList":["mktg1"],"permList":["Read"]},{"groupList":["IT"],"permList":["Read"]}],"isEnabled":true,"isRecursive":true,"isAuditEnabled":true,"version":"0.1.0","replacePerm":false}'

![](http://www.dropbox.com/s/8bgjlmu2762m2hp/Screenshot%202015-09-08%2014.18.43.png?dl=1)

the policy manager and see the new policy named “hadoopdev-testing-policy2”

![](http://www.dropbox.com/s/jmdpelawvh642u5/Screenshot%202015-09-08%2014.21.07.png?dl=1)

Click on the policy and check the permissions that has been created

![](http://www.dropbox.com/s/w7h4n1xh0p2pd9x/Screenshot%202015-09-08%2014.21.51.png?dl=1)

The policy id is part of the URL of this policy detail page[http://127.0.0.1:6080/index.html#!/hdfs/1/policy/26](http://127.0.0.1:6080/index.html#!/hdfs/1/policy/26)

We can use the policy id to retrieve or change the policy.

Run the below CURL command to get policy details using API

`curl -i --user admin:admin -X GET http://127.0.0.1:6080/service/public/api/policy/26`

![](http://www.dropbox.com/s/es3mf7nvgiqrnis/Screenshot%202015-09-08%2014.24.23.png?dl=1)

What happened here?

We created a policy and retrieved policy details using REST APIs. Users can now manage their policies using API tools or applications integrated with the Ranger REST APIs

Hopefully, through this whirlwind tour of Ranger, you were introduced to the simplicity and power of Ranger for security administration.
