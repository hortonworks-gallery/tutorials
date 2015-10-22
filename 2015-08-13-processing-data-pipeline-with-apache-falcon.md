---
title: Processing Data Pipeline on Hadoop clusters with Apache Falcon
layout: post
date: '2015-08-13T12:34:00.001-07:00'
author: Saptak Sen
tags:
- falcon
- hadoop
modified_time: '2015-08-13T15:11:18.054-07:00'
---
Apache Falcon is a framework to simplify data pipeline processing and management on Hadoop clusters.

It makes it much simpler to onboard new workflows/pipelines, with support for late data handling and retry policies. It allows you to easily define relationship between various data and processing elements and integrate with metastore/catalog such as Hive/HCatalog. Finally it also lets you capture lineage information for feeds and processes.In this tutorial we are going walk the process of:

* Defining the feeds and processes
* Defining and executing a job to mirror data between two clusters
* Defining and executing a data pipeline to ingest, process and persist data continuously

###Prerequisite

[Download Hortonworks Sandbox](http://hortonworks.com/products/hortonworks-sandbox/#install)

Once you have download the Hortonworks sandbox and run the VM, navigate to the Ambari interface on the port `8080` of the IP address of your Sandbox VM. Login with the username and password of `admin` and `admin` respectively:  

![](http://www.dropbox.com/s/ddpzb9y5n3hav6o/Screenshot%202015-08-19%2016.28.48.png?dl=1)

### Scenario

In this tutorial we will walk through a scenario where email data lands hourly on a cluster. In our example:

  * This cluster is the primary cluster located in the Oregon data center.
  * Data arrives from all the West Coast production servers. The input data feeds are often late for up to 4 hrs.

The goal is to clean the raw data to remove sensitive information like credit card numbers and make it available to our marketing data science team for customer churn analysis.

To simulate this scenario, we have a pig script grabbing the freely available Enron emails from the internet and feeding it into the pipeline.

![](http://hortonassets.s3.amazonaws.com/tutorial/falcon/images/arch.png)


###Starting Falcon

By default, Falcon is not started on the sandbox. You can click on the Falcon icon on the left hand bar:

![](http://www.dropbox.com/s/z5zx9r37p67y4q3/Screenshot%202015-08-19%2016.29.22.png?dl=1)

Then click on the `Service Actions` button on the top right:

![](http://www.dropbox.com/s/6wrbaqb29ak2d2j/Screenshot%202015-08-19%2016.29.44.png?dl=1)

Then click on `Start`:

![](http://www.dropbox.com/s/1tbgz6t69ydqdwa/Screenshot%202015-08-19%2016.30.07.png?dl=1)

Once, Falcon starts, Ambari should clearly indicate as below that the service has started:

![](http://www.dropbox.com/s/zotukjfp01qyn8v/Screenshot%202015-08-19%2016.34.32.png?dl=1)

###Downloading and staging the dataset

Now let's stage the dataset using the commandline. Although we perform many of these file operations below using the command line, you can also do the same with the `HDFS Files View` in Ambari.

First SSH into the Hortonworks Sandbox with the command:


![](http://www.dropbox.com/s/tzsxvsnxfo26jn7/Screenshot_2015-04-13_07_58_43.png?dl=1)

The default password is `hadoop`

Then login as user `hdfs`

```bash
su - hdfs
```

Then download the file falcon.zip with the following command"

```
wget http://hortonassets.s3.amazonaws.com/tutorial/falcon/falcon.zip
```

and then unzip with the command

```
unzip falcon.zip
```

![](http://www.dropbox.com/s/3g0dnj9pdrp7c07/Screenshot%202015-08-10%2018.39.50.png?dl=1)

Now let's give ourselves permission to upload files

```
hadoop fs -chmod -R 777 /user/ambari-qa
```

then let's create a folder `falcon` under `ambari-qa` with the command

```
hadoop fs -mkdir /user/ambari-qa/falcon
```

![](http://www.dropbox.com/s/lnzlibnqqijcjms/Screenshot%202015-08-25%2018.24.59.png?dl=1)

Now let's upload the decompressed folder with the command

```
hadoop fs -copyFromLocal demo /user/ambari-qa/falcon/
```
![](http://www.dropbox.com/s/v29ls1ab0d00mhb/Screenshot%202015-08-11%2015.05.45.png?dl=1)

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


Next we will need to create the `working` directories for `primaryCluster` and `backupCluster`

```bash
hadoop fs -mkdir /apps/falcon/primaryCluster/working
hadoop fs -mkdir /apps/falcon/backupCluster/working
```

![](http://www.dropbox.com/s/midzw0tr3rs7eov/Screenshot%202015-08-07%2010.36.12.png?dl=1)

Finally you need to set the proper permissions on the staging/working directories:

```bash
hadoop fs -chmod 777 /apps/falcon/primaryCluster/staging
hadoop fs -chmod 755 /apps/falcon/primaryCluster/working
hadoop fs -chmod 777 /apps/falcon/backupCluster/staging
hadoop fs -chmod 755 /apps/falcon/backupCluster/working
hadoop fs –chown –R falcon /apps/falcon/*
```

Let's open the Falcon Web UI. You can easily launch the Falcon Web UI from Ambari:

![](http://www.dropbox.com/s/ypv88ruir0djx5e/Screenshot%202015-08-19%2016.31.12.png?dl=1)

You can also navigate to the Falcon Web UI directly on our browser. The Falcon UI is by default at port 15000. The default username is `ambari-qa` and the password is `admin`.

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

###Defining the rawEmailFeed entity

To create a feed entity click on the `Feed` button on the top of the main page on the Falcon Web UI.

Then click on the edit button over XML Preview area on the right hand side of the screen and replace the XML content with the XML document below:

```xml
<?xml version="1.0" encoding="UTF-8" standalone="yes"?>
<feed name="rawEmailFeed" description="Raw customer email feed" xmlns="uri:falcon:feed:0.1">
    <tags>externalSystem=USWestEmailServers</tags>
    <groups>churnAnalysisDataPipeline</groups>
    <frequency>hours(1)</frequency>
    <timezone>UTC</timezone>
    <late-arrival cut-off="hours(1)"/>
    <clusters>
        <cluster name="primaryCluster" type="source">
            <validity start="2015-07-22T01:00Z" end="2015-07-22T10:00Z"/>
            <retention limit="days(90)" action="delete"/>
        </cluster>
    </clusters>
    <locations>
        <location type="data" path="/user/ambari-qa/falcon/demo/primary/input/enron/${YEAR}-${MONTH}-${DAY}-${HOUR}"/>
        <location type="stats" path="/"/>
        <location type="meta" path="/"/>
    </locations>
    <ACL owner="ambari-qa" group="users" permission="0x755"/>
    <schema location="/none" provider="/none"/>
</feed>
```

Click `Finish` on the top of the XML Preview area

![](http://www.dropbox.com/s/ne2877qvxl7apl0/Screenshot%202015-08-11%2015.09.14.png?dl=1)

Falcon UI should have automatically parsed out the values from the XML and populated in the right fields. Once you have verified that these are the correct values press `Next`.

![](http://www.dropbox.com/s/wxq9b5gbk0crgly/Screenshot%202015-08-11%2015.14.21.png?dl=1)

On the Clusters page ensure you modify the validity to a time slice which is in the very near future.

Click `Next`

![](http://www.dropbox.com/s/tdit1iys7od1lva/Screenshot%202015-08-11%2015.15.35.png?dl=1)

Save the feed

![](http://www.dropbox.com/s/k55ijdij8i6avso/Screenshot%202015-08-11%2015.16.01.png?dl=1)

###Defining the rawEmailIngestProcess entity

Now lets define the `rawEmailIngestProcess`.

To create a process entity click on the `Process` button on the top of the main page on the Falcon Web UI.

Then click on the edit button over XML Preview area on the right hand side of the screen and replace the XML content with the XML document below:

```xml
<?xml version="1.0" encoding="UTF-8" standalone="yes"?>
<process name="rawEmailIngestProcess" xmlns="uri:falcon:process:0.1">
    <tags>email=testemail</tags>
    <clusters>
        <cluster name="primaryCluster">
            <validity start="2015-07-22T01:00Z" end="2015-07-22T10:00Z"/>
        </cluster>
    </clusters>
    <parallel>1</parallel>
    <order>FIFO</order>
    <frequency>hours(1)</frequency>
    <timezone>UTC</timezone>
    <outputs>
        <output name="output" feed="rawEmailFeed" instance="now(0,0)"/>
    </outputs>
    <workflow name="emailIngestWorkflow" version="4.0.1" engine="oozie" path="/user/ambari-qa/falcon/demo/apps/ingest/fs"/>
    <retry policy="exp-backoff" delay="minutes(3)" attempts="3"/>
    <ACL owner="ambari-qa" group="users" permission="0x755"/>
</process>
```
Click `Finish` on the top of the XML Preview area

![](http://www.dropbox.com/s/k9cm0fveefdrfgk/Screenshot%202015-08-11%2015.17.01.png?dl=1)

Accept the default values and click next

![](http://www.dropbox.com/s/g7dlc0lijvybkam/Screenshot%202015-08-11%2015.17.19.png?dl=1)

On the Clusters page ensure you modify the validity to a time slice which is in the very near future and then click next

![](http://www.dropbox.com/s/b4udackmnu8m5a9/Screenshot%202015-08-11%2015.18.02.png?dl=1)

Accept the default values and click Next

![](http://www.dropbox.com/s/3bf4dhk8nonssst/Screenshot%202015-08-11%2015.18.15.png?dl=1)

Let's `Save` the process.

![](http://www.dropbox.com/s/0cgbrb8gh306ccz/Screenshot%202015-08-11%2015.18.37.png?dl=1)

###Defining the cleansedEmailFeed

Again, to create a feed entity click on the `Feed` button on the top of the main page on the Falcon Web UI.

Then click on the edit button over XML Preview area on the right hand side of the screen and replace the XML content with the XML document below:

```xml
<?xml version="1.0" encoding="UTF-8" standalone="yes"?>
<feed name="cleansedEmailFeed" description="Cleansed customer emails" xmlns="uri:falcon:feed:0.1">
    <tags>cleanse=cleaned</tags>
    <groups>churnAnalysisDataPipeline</groups>
    <frequency>hours(1)</frequency>
    <timezone>UTC</timezone>
    <late-arrival cut-off="hours(4)"/>
    <clusters>
        <cluster name="primaryCluster" type="source">
            <validity start="2015-07-22T01:00Z" end="2015-07-22T10:00Z"/>
            <retention limit="hours(90)" action="delete"/>
            <locations>
                <location type="data" path="/user/ambari-qa/falcon/demo/primary/processed/enron/${YEAR}-${MONTH}-${DAY}-${HOUR}"/>
                <location type="stats" path="/"/>
                <location type="meta" path="/"/>
            </locations>
        </cluster>
        <cluster name="backupCluster" type="target">
            <validity start="2015-07-22T01:00Z" end="2015-07-22T10:00Z"/>
            <retention limit="hours(90)" action="delete"/>
            <locations>
                <location type="data" path="/falcon/demo/bcp/processed/enron/${YEAR}-${MONTH}-${DAY}-${HOUR}"/>
                <location type="stats" path="/"/>
                <location type="meta" path="/"/>
            </locations>
        </cluster>
    </clusters>
    <locations>
        <location type="data" path="/user/ambari-qa/falcon/demo/processed/enron/${YEAR}-${MONTH}-${DAY}-${HOUR}"/>
        <location type="stats" path="/"/>
        <location type="meta" path="/"/>
    </locations>
    <ACL owner="ambari-qa" group="users" permission="0x755"/>
    <schema location="/none" provider="/none"/>
</feed>
```
Click `Finish` on the top of the XML Preview area

![](http://www.dropbox.com/s/eibbg47a2jw5t1s/Screenshot%202015-08-11%2015.35.10.png?dl=1)

Accept the default values and click Next

![](http://www.dropbox.com/s/8xbise44h40imtv/Screenshot%202015-08-11%2015.35.49.png?dl=1)

Accept the default values and click Next

![](http://www.dropbox.com/s/3y7kyu0ngiamwep/Screenshot%202015-08-11%2015.35.58.png?dl=1)

On the Clusters page ensure you modify the validity to a time slice which is in the very near future and then click Next

![](http://www.dropbox.com/s/5ahsbuvuzs24crv/Screenshot%202015-08-11%2015.36.35.png?dl=1)

![](http://www.dropbox.com/s/nf3ym257dn04851/Screenshot%202015-08-11%2015.37.05.png?dl=1)

Accept the default values and click Save

![](http://www.dropbox.com/s/mvd9olqxst6cfzl/Screenshot%202015-08-11%2015.37.21.png?dl=1)

### Defining the cleanseEmailProcess

Now lets define the `cleanseEmailProcess`.

Again, to create a process entity click on the `Process` button on the top of the main page on the Falcon Web UI.

Then click on the edit button over XML Preview area on the right hand side of the screen and replace the XML content with the XML document below:

```xml
<?xml version="1.0" encoding="UTF-8" standalone="yes"?>
<process name="cleanseEmailProcess" xmlns="uri:falcon:process:0.1">
    <tags>cleanse=yes</tags>
    <clusters>
        <cluster name="primaryCluster">
            <validity start="2015-07-22T01:00Z" end="2015-07-22T10:00Z"/>
        </cluster>
    </clusters>
    <parallel>1</parallel>
    <order>FIFO</order>
    <frequency>hours(1)</frequency>
    <timezone>UTC</timezone>
    <inputs>
        <input name="input" feed="rawEmailFeed" start="now(0,0)" end="now(0,0)"/>
    </inputs>
    <outputs>
        <output name="output" feed="cleansedEmailFeed" instance="now(0,0)"/>
    </outputs>
    <workflow name="emailCleanseWorkflow" version="pig-0.13.0" engine="pig" path="/user/ambari-qa/falcon/demo/apps/pig/id.pig"/>
    <retry policy="exp-backoff" delay="minutes(5)" attempts="3"/>
    <ACL owner="ambari-qa" group="users" permission="0x755"/>
</process>
```
Click `Finish` on the top of the XML Preview area

![](http://www.dropbox.com/s/67euk40jojsmioz/Screenshot%202015-08-11%2015.39.34.png?dl=1)

Accept the default values and click Next

![](http://www.dropbox.com/s/dpmpfhdfwkyb5dc/Screenshot%202015-08-11%2015.39.53.png?dl=1)

On the Clusters page ensure you modify the validity to a time slice which is in the very near future and then click Next

![](http://www.dropbox.com/s/80gtpe4ov99ebyf/Screenshot%202015-08-11%2015.40.24.png?dl=1)

Select the Input and Output Feeds as shown below and Save

![](http://www.dropbox.com/s/w3jdrtu2o4krxhk/Screenshot%202015-08-11%2015.40.40.png?dl=1)

###Running the feeds

From the Falcon Web UI home page search for the Feeds we created

![](http://www.dropbox.com/s/wbo7xboeu596fwf/Screenshot%202015-08-11%2015.41.34.png?dl=1)

Select the rawEmailFeed by clicking on the checkbox

![](http://www.dropbox.com/s/djgbw9o6i9jbfh2/Screenshot%202015-08-11%2015.41.56.png?dl=1)

Then click on the Schedule button on the top of the search results

![](http://www.dropbox.com/s/pq8inbov0vuho3x/Screenshot%202015-08-11%2015.42.04.png?dl=1)

Next run the cleansedEmailFeed in the same way

![](http://www.dropbox.com/s/rkqmzqhjxsp353k/Screenshot%202015-08-11%2015.42.30.png?dl=1)

####Running the processes

From the Falcon Web UI home page search for the Process we created

![](http://www.dropbox.com/s/m9t309isyw9fme5/Screenshot%202015-08-11%2015.42.55.png?dl=1)

Select the cleanseEmailProcess by clicking on the checkbox

![](http://www.dropbox.com/s/o0f6uiywmb4hdzs/Screenshot%202015-08-11%2015.43.07.png?dl=1)

Then click on the Schedule button on the top of the search results

![](http://www.dropbox.com/s/5wrpsl7e46ov563/Screenshot%202015-08-11%2015.43.31.png?dl=1)

Next run the rawEmailIngestProcess in the same way

![](http://www.dropbox.com/s/afsxyy215xcnanf/Screenshot%202015-08-11%2015.43.41.png?dl=1)

If you visit the Oozie process page, you can seen the processes running

![](http://www.dropbox.com/s/mlleewsrrn5u0ty/Screenshot%202015-08-11%2015.44.23.png?dl=1)

###Input and Output of the pipeline

Now that the feeds and processes are running, we can check the dataset being ingressed and the dataset egressed on HDFS.

![](http://www.dropbox.com/s/fusow13whgtd7oa/Screenshot%202015-08-11%2015.45.48.png?dl=1)

Here is the data being ingressed

![](http://www.dropbox.com/s/lpq8sxywi47tg37/Screenshot%202015-08-11%2016.31.37.png?dl=1)

and here is the data being egressed from the pipeline

![](http://www.dropbox.com/s/b442uvtrjtuzr6o/Screenshot%202015-08-11%2017.13.05.png?dl=1)

### Summary

In this tutorial we walked through a scenario to clean the raw data to remove sensitive information like credit card numbers and make it available to our marketing data science team for customer churn analysis by defining a data pipeline with Apache Falcon.
