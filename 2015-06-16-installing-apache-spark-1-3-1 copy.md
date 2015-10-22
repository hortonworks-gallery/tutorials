---
layout: post
title: Installing Apache Spark 1.3.1 on HDP 2.2.4.2
date: '2014-06-16T12:34:00.001-07:00'
author: Saptak Sen
tags:
- spark
- hadoop
modified_time: '2014-06-16T15:11:18.054-07:00'
---
In this section we will configure Spark 1.3.1 on Hortonworks Sandbox with HDP 2.2.

Login as root to your Sandbox and add the repo that has Spark 1.3.1 using the following command

```bash
wget -nv http://public-repo-1.hortonworks.com/HDP/centos6/2.x/updates/2.2.4.4/hdp.repo -O /etc/yum.repos.d/HDP-TP.repo
```

![](https://www.dropbox.com/s/j60xo6twvt20aju/Screenshot%202015-06-07%2016.00.25.png?dl=1)

With the following command install the Spark package

```bash
yum install spark_2_2_4_4_16-master
```
It will take a few minutes to complete the installation of Spark and all the dependencies:

![](https://www.dropbox.com/s/2gi21haz2l99obq/Screenshot%202015-06-07%2016.13.47.png?dl=1)

Lets's also install pyspark with the command

```bash
yum install spark-python
```
![](https://www.dropbox.com/s/et4wbmz6walrl59/Screenshot%202015-06-07%2016.16.55.png?dl=1)

Use the `hdp-select` command to configure history server and client to point to the version we just installed:

```bash
hdp-select set spark-historyserver 2.2.4.4-16
hdp-select set spark-client 2.2.4.4-16
```

![](https://www.dropbox.com/s/8ye09a1wos2p04i/Screenshot%202015-06-07%2016.24.19.png?dl=1)

Let's check if all is well by running a sample.

First let's as the `spark` user that was created during the RPM install and then cd to the $SPARK_HOME directory.

```
su spark
cd /usr/hdp/current/spark-client
```
![](https://www.dropbox.com/s/hew5n8056maa60b/Screenshot%202015-06-07%2016.31.12.png?dl=1)


Now let's run the sample to calculate properties

```
./bin/spark-submit --class org.apache.spark.examples.SparkPi --master yarn-client --num-executors 3 --driver-memory 512m --executor-memory 512m --executor-cores 1 lib/spark-examples*.jar 10
```

![](https://www.dropbox.com/s/o31shit8i04xbxs/Screenshot%202015-06-08%2007.59.16.png?dl=1)
