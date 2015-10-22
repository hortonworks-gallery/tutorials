---
title: Mirroring Datasets between Hadoop clusters with Apache Falcon
layout: post
date: '2015-08-11T12:34:00.001-07:00'
author: Saptak Sen
tags:
- falcon
- hadoop
modified_time: '2015-08-11T15:11:18.054-07:00'
---
Apache Falcon is a framework to simplify data pipeline processing and management on Hadoop clusters.

It provides data management services such as retention, replications across clusters, archival etc. It makes it much simpler to onboard new workflows/pipelines, with support for late data handling and retry policies. It allows you to easily define relationship between various data and processing elements and integrate with metastore/catalog such as Hive/HCatalog. Finally it also lets you capture lineage information for feeds and processes.

In this tutorial we are going walk the process of:

* Defining the cluster entities
* Defining and executing a job to mirror data between two clusters

###Prerequisite

[Download Hortonworks Sandbox](http://hortonworks.com/products/hortonworks-sandbox/#install)

Once you have download the Hortonworks sandbox and run the VM, open a commandline shell to our Sandbox through SSH:  

![](https://www.dropbox.com/s/tzsxvsnxfo26jn7/Screenshot_2015-04-13_07_58_43.png?dl=1)

The default password is `hadoop`

###Starting Falcon

By default, Falcon is not started on the sandbox. You can use the following commands on the shell command line to start Apache Falcon:

 First login as user `falcon`

```bash
su - falcon
```
Change directory to Falcon install directory:

```bash
cd /usr/hdp/2.3.0.0-2557/falcon/
```

Use `falcon-stop` to stop any existing falcon instances

```bash
./bin/falcon-stop
```
Now, use `falcon-start` to start Falcon

```bash
./bin/falcon-start
```

![](http://www.dropbox.com/s/r889tzu4qszyxyr/Screenshot%202015-08-07%2010.26.03.png?dl=1)

You can check the status of falcon using the `falcon-status` command

```bash
./bin/falcon-status
```

![](http://www.dropbox.com/s/n9l9zvntwjw7a3o/Screenshot%202015-08-07%2010.26.38.png?dl=1)

###Creating the cluster entities

Before creating the cluster entities, we need to create the directories on HDFS representing the two clusters that we are going to define, namely `primaryCluster` and `backupCluster`.

Use `hadoop fs -mkdir` commands to create the directories `/apps/falcon/primaryCluster` and `/apps/falcon/backupCluster` directories on HDFS.

```bash
hadoop fs -mkdir /apps/falcon/primaryCluster
hadoop fs -mkdir /apps/falcon/backupCluster
```

![](http://www.dropbox.com/s/d9ktkj2em5xmyqe/Screenshot%202015-08-07%2010.29.58.png?dl=1)

Further create directories called `staging` inside each of the directories we created above:

```
hadoop fs -mkdir /apps/falcon/primaryCluster/staging
hadoop fs -mkdir /apps/falcon/backupCluster/staging
```

![](http://www.dropbox.com/s/v0ydcb0him66uui/Screenshot%202015-08-07%2010.31.37.png?dl=1)

Now we need to change the permission recursively on the `falcon` directory on HDFS

```bash
hadoop fs -chmod -R 777 /apps/falcon/*
```
![](http://www.dropbox.com/s/5g0xo5rfx4h8poy/Screenshot%202015-08-07%2010.33.32.png?dl=1)

Next we need to change the owner of the directories to user `falcon`

```bash
hadoop fs -chown -R falcon /apps/falcon/*
```


![](http://www.dropbox.com/s/fme73f0pbitvcdp/Screenshot%202015-08-07%2010.34.25.png?dl=1)

Next we will need to create the `working` directories for `primaryCluster` and `backupCluster`

```bash
hadoop fs -mkdir /apps/falcon/primaryCluster/working
hadoop fs -mkdir /apps/falcon/backupCluster/working
```

![](http://www.dropbox.com/s/midzw0tr3rs7eov/Screenshot%202015-08-07%2010.36.12.png?dl=1)

As before we have to set the owner and permission of the directories

```bash
hadoop fs -chown -R falcon /apps/falcon/*
hadoop fs -chmod -R 755 /apps/falcon/primaryCluster/working /apps/falcon/backupCluster/working
```

![](http://www.dropbox.com/s/i05cdsdegc6z9g8/Screenshot%202015-08-07%2010.43.50.png?dl=1)

Now let's navigate to the Falcon UI on our browser. The Falcon UI is by default at port 15000. The default username is `ambari-qa` and the password is `admin`.

![](http://www.dropbox.com/s/a428livn5qj2q9t/Screenshot%202015-08-07%2010.45.40.png?dl=1)

This UI allows us to create and manage the various entities like Cluster, Feed, Process and Mirror. Each of these entities are represented by a XML file which you either directly upload or generate by filling up the various fields.

You can also search for existing entities and then edit, change state, etc.

![](http://www.dropbox.com/s/9ttkhjdmzhtwyzf/Screenshot%202015-08-07%2010.46.23.png?dl=1)

Let's first create a couple of cluster entities. To create a cluster entity click on the `Cluster` button on the top.

Then click on the `edit` button over XML Preview area on the right hand side of the screen and replace the XML content with the XML document below:

```xml
<?xml version="1.0" encoding="UTF-8" standalone="yes"?>
<cluster name="primaryCluster" description="this is primary cluster" colo="primaryColo" xmlns="uri:falcon:cluster:0.1">
    <tags>primaryKey=primaryValue</tags>
    <interfaces>
        <interface type="readonly" endpoint="hftp://sandbox.hortonworks.com:50070" version="2.2.0"/>
        <interface type="write" endpoint="hdfs://sandbox.hortonworks.com:8020" version="2.2.0"/>
        <interface type="execute" endpoint="sandbox.hortonworks.com:8050" version="2.2.0"/>
        <interface type="workflow" endpoint="http://sandbox.hortonworks.com:11000/oozie/" version="4.0.0"/>
        <interface type="messaging" endpoint="tcp://sandbox.hortonworks.com:61616?daemon=true" version="5.1.6"/>
    </interfaces>
    <locations>
        <location name="staging" path="/apps/falcon/primaryCluster/staging"/>
        <location name="temp" path="/tmp"/>
        <location name="working" path="/apps/falcon/primaryCluster/working"/>
    </locations>
    <ACL owner="ambari-qa" group="users" permission="0x755"/>
    <properties>
        <property name="test" value="value1"/>
    </properties>
</cluster>
```
Click `Finish` on top of the XML Preview area to save the XML.
![](http://www.dropbox.com/s/uvkbmrgsmrgpn9b/Screenshot%202015-08-07%2010.49.25.png?dl=1)

Falcon UI should have automatically parsed out the values from the XML and populated in the right fields. Once you have verified that these are the correct values press `Next`.

![](http://www.dropbox.com/s/ebizakk740y7bte/Screenshot%202015-08-07%2010.50.01.png?dl=1)

Click `Save` to persist the entity.

![](http://www.dropbox.com/s/1htzp8ljm4gy9ik/Screenshot%202015-08-07%2010.50.18.png?dl=1)

Similarly, we will create the `backupCluster` entity. Again click on `Cluster` button on the top to open up the form to create the cluster entity.

Then click on the `edit` button over XML Preview area on the right hand side of the screen and replace the XML content with the XML document below:

```xml
<?xml version="1.0" encoding="UTF-8" standalone="yes"?>
<cluster name="backupCluster" description="this is backup colo" colo="backupColo" xmlns="uri:falcon:cluster:0.1">
    <tags>backupTag=backupTagValue</tags>
    <interfaces>
        <interface type="readonly" endpoint="hftp://sandbox.hortonworks.com:50070" version="2.2.0"/>
        <interface type="write" endpoint="hdfs://sandbox.hortonworks.com:8020" version="2.2.0"/>
        <interface type="execute" endpoint="sandbox.hortonworks.com:8050" version="2.2.0"/>
        <interface type="workflow" endpoint="http://sandbox.hortonworks.com:11000/oozie/" version="4.0.0"/>
        <interface type="messaging" endpoint="tcp://sandbox.hortonworks.com:61616?daemon=true" version="5.1.6"/>
    </interfaces>
    <locations>
        <location name="staging" path="/apps/falcon/backupCluster/staging"/>
        <location name="temp" path="/tmp"/>
        <location name="working" path="/apps/falcon/backupCluster/working"/>
    </locations>
    <ACL owner="ambari-qa" group="users" permission="0x755"/>
    <properties>
        <property name="key1" value="val1"/>
    </properties>
</cluster>
```
Click `Finish` on top of the XML Preview area to save the XML and then the `Next` button to verify the values.

![](http://www.dropbox.com/s/ohr0egsj7rmi9nq/Screenshot%202015-08-07%2010.51.14.png?dl=1)

Click `Save` to persist the `backupCluster` entity.

![](http://www.dropbox.com/s/u14draatq25sr3s/Screenshot%202015-08-07%2010.51.33.png?dl=1)

Now let's go back to the SSH terminal and create the directory `/user/ambari-qa/falcon` on HDFS and then the directories `mirrorSrc` and `mirrorTgt` as the source and target of the mirroring job we are about to create.

```bash
hadoop fs -mkdir /user/ambari-qa/falcon
hadoop fs -mkdir /user/ambari-qa/falcon/mirrorSrc
hadoop fs -mkdir /user/ambari-qa/falcon/mirrorTgt
```

![](http://www.dropbox.com/s/gnxxkc9p7x9qz45/Screenshot%202015-08-10%2013.36.49.png?dl=1)

Now we need to set a permission to allow access:

```bash
hadoop fs -chmod -R 777 /user/ambari-qa/falcon
```

![](http://www.dropbox.com/s/v0nzza61guwm2nk/Screenshot%202015-08-10%2013.38.05.png?dl=1)

###Setting up the Mirroring Job

To create the mirroring job, Go back the Falcon UI on your browser and click on the `Mirror` button on the top to create a mirroring job

![](http://www.dropbox.com/s/z2q57euxt34olee/Screenshot%202015-08-10%2013.41.39.png?dl=1)

Provide a name of your choice. We named the Mirror Job `MirrorTest`

![](http://www.dropbox.com/s/ywyugxyv27epwx1/Screenshot%202015-08-10%2013.50.22.png?dl=1)

Select the appropriate Source and Target. In our case the source cluster is `primaryCluster` and that HDFS path on the cluster is `/user/ambari-qa/falcon/mirrorSrc`.

The target cluster is `backupCluster` and that HDFS path on the cluster is `/user/ambari-qa/falcon/mirrorTgt`.

Also set the validity of the job to your current time, so that when you attempt to run tthe job in a few minutes, the job is still within the validity period.

![](http://www.dropbox.com/s/30tvzf4c5ph3w31/Screenshot%202015-08-10%2013.52.07.png?dl=1)

###Running the Job

Before we can run the job we need some data to test on HDFS. Let's give us permission to upload some data using the HDFS View in Ambari.

```bash
hadoop fs -chmod -R 775 /user/ambari-qa
```


![](http://www.dropbox.com/s/f530p25vco8g3iu/Screenshot%202015-08-10%2013.55.08.png?dl=1)

Open Ambari from your browser at port 8080.

Then launch the HDFS view from the top right hand corner.


From the view on the Ambari console navigate to the directory `/user/ambari-qa/falcon/mirrorSrc`

![](http://www.dropbox.com/s/94y0req4b4yyx3f/Screenshot%202015-08-10%2014.00.22.png?dl=1)

Upload any file

![](http://www.dropbox.com/s/mgvikjdx8bhgrbq/Screenshot%202015-08-10%2014.00.06.png?dl=1)

Once uploaded the file should appear in the directory

![](http://www.dropbox.com/s/oddhh0m79ofsu03/Screenshot%202015-08-10%2014.01.36.png?dl=1)

Now navigate to the Falcon Ui and search for the job we created. The name of the Mirro job we had created was `MirrorTest`

![](http://www.dropbox.com/s/ww54dc0b9w9kqm0/Screenshot%202015-08-10%2014.02.57.png?dl=1)

Select the `MirrorTest` job my checking the checkbox and thebclick on `Schedule`

![](http://www.dropbox.com/s/9ro49x5za0fpi9c/Screenshot%202015-08-10%2014.03.17.png?dl=1)

The state of the job should change to `RUNNING`

![](http://www.dropbox.com/s/fq90cky0ho4na40/Screenshot%202015-08-10%2014.03.40.png?dl=1)

After a few minutes, use the HDFS View in the Ambari console to check the `/user/ambari-qa/falcon/mirrorTgt` directory and you should notice your data mirrored

![](http://www.dropbox.com/s/vpbine4pzyu4vo2/Screenshot%202015-08-10%2014.41.20.png?dl=1)

In this tutorial we walked through the process of defining the cluster entities representing two different clusters and then mirroring the datasets between them. In the next tutorial we will work through defining various data feeds and processing them to refine the data.
