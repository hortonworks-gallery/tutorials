---
layout: post
title: Basics of programming Apache Spark
date: '2014-06-18T12:34:00.001-07:00'
author: Saptak Sen
tags:
- spark
- hadoop
modified_time: '2014-06-18T15:11:18.054-07:00'
---

###Concepts

**RDD** or Resilient Distributed Dataset is an immutable collection of objects that is usually partitioned and distributed across multiple physical nodes of the YARN cluster. So once a RDD is instantiated, it cannot be changed.

Typically, RDDs are instantiated by loading data from HDFS, HBASE, etc on a YARN cluster.

Once a RDD is instantiated you can apply a series of operations.  All operations fall into one of two types, **transformations** or **actions**. **Transformation** operations build out the processing graph which can then be applied on the partitioned dataset across the YARN cluster once the **Action** operation is invoked. Each transformation creates a new RDD.

Let's try it out.

###Hands-On

Let's open a shell to our Sandbox through SSH:

![](https://www.dropbox.com/s/tzsxvsnxfo26jn7/Screenshot_2015-04-13_07_58_43.png?dl=1)

The default password is `hadoop` or in case of Sandbox on Azure, what ever you've set it to.



Then let's get some data with the command below in your shell prompt:

```bash
wget http://en.wikipedia.org/wiki/Hortonworks
```
![](https://www.dropbox.com/s/p6v9f2garljdpoj/Screenshot_2015-04-13_08_11_41.png?dl=1)

Copy the data over to HDFS on Sandbox:

```bash
hadoop fs -put ~/Hortonworks /user/guest/Hortonworks
```


Let's start the PySpark shell and work through a simple example of counting the lines in a file. PySpark shell let's us interact with out data using Spark and Python:

```bash
pyspark
```
![](https://www.dropbox.com/s/vr5syq682z8usla/Screenshot%202015-04-13%2007.59.59.png?dl=1)

In case you do not want such verbose logging, you can change the verbosity in the `$SPARK_HOME/conf/log4j.properties` file.

First exit `pyspark` console by pressing `CTRL+D`.

![](https://www.dropbox.com/s/wkxkddlkwuv94sb/Screenshot%202015-06-08%2007.16.04.png?dl=1)

In case you do not already have the `log4j.properties` file, make a copy of the file using the command below

```
cp /usr/hdp/current/spark-client/conf/log4j.properties.template /usr/hdp/current/spark-client/conf/log4j.properties
```

![](https://www.dropbox.com/s/dzvu6xu0yfb8bcy/Screenshot%202015-06-08%2007.09.28.png?dl=1)

Then edit the file `/usr/hdp/current/spark-client/conf/log4j.properties` to change the line

```
log4j.rootCategory=INFO, console
```
to

```
log4j.rootCategory=WARN, console
```
![](https://www.dropbox.com/s/x5z0wxkljwk1khb/Screenshot%202015-06-08%2007.17.29.png?dl=1)

Now relaunch `pyspark`

![](https://www.dropbox.com/s/vjayu75m5g6n2db/Screenshot%202015-06-08%2007.19.16.png?dl=1)

Now it's much cleaner. Let's start programming Spark.

As discussed above, the first step is to instantiate the RDD using the Spark Context `sc` with the file `Hortonworks` on HDFS.

```python
myLines = sc.textFile('hdfs://sandbox.hortonworks.com/user/guest/Hortonworks')
```
![](https://www.dropbox.com/s/a2d7v61acgozid7/Screenshot%202015-04-13%2009.10.32.png?dl=1)

Now that we have instantiated the RDD, it's time to apply some transformation operations on the RDD. In this case, I am going to apply a simple transformation operation `filter(f)` using a Python lambda expression to filter out all the empty lines. The `filter(f)` method is a data-parallel operation that creates a new RDD from the input RDD by applying filter function `f` to each item in the parent RDD and only passing those elements where the filter function returns `True`. Elements that do not return `True` will be dropped. Like map(), filter can be applied individually to each entry in the dataset, so is easily parallelized using Spark.


```python
myLines_filtered = myLines.filter( lambda x: len(x) > 0 )
```
![](https://www.dropbox.com/s/0m0wg35a89p3rrj/Screenshot%202015-04-13%2009.17.52.png?dl=1)

Note that the previous Python statement returned without any output. This lack of output signifies, that the transformation operation did not touch the data in any way so far, but has only modified the processing graph.

Let's make this transformation real, with an Action operation like 'count()', which will execute all the transformation actions before and apply this aggregate function.

```python
myLines_filtered.count()
```
![](https://www.dropbox.com/s/q42679pbo8m2hf1/Screenshot%202015-04-13%2009.19.07.png?dl=1)

The final result of this little Spark Job is the number you see at the end. In this case it is `341`.

Hope this little example whets up your appetite for more ambitious data science projects on the Hortonworks Data Platform.
