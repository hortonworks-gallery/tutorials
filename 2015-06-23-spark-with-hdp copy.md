---
layout: post
title: A Lap around Apache Spark 1.3.1 with HDP 2.3
date: '2014-06-24T12:34:00.001-07:00'
author: Saptak Sen
tags:
- spark
- hadoop
modified_time: '2014-06-24T15:11:18.054-07:00'
---
This Apache Spark 1.3.1 with HDP 2.3 guide walks you through many of the newer features of Apache Spark 1.3.1 on YARN.

Hortonworks recently announced general availability of Spark 1.3.1 on the HDP platform. Apache Spark is a fast moving community and Hortonworks plans frequent releases to allow evaluation and production use of the latest capabilities of Apache Spark on HDP for our customers.

With YARN, Hadoop can now support many types of data and application workloads; Spark on YARN becomes yet another workload running against the same set of hardware resources.

This guide describes how to:

  * Run Spark on YARN and run the canonical Spark examples: SparkPi and Wordcount.
  * Run Spark 1.3.1 on HDP 2.3.
  * Use Spark DataFrame API
  * Work with a built-in UDF, collect_list, a key feature of Hive 13. This release provides support for Hive 0.13.1 and instructions on how to call this UDF from Spark shell.
  * Use SparkSQL thrift JDBC/ODBC Server.
  * View history of finished jobs with Spark Job History.
  * Use ORC files with Spark, with examples.

When you are ready to go beyond these tasks, try the machine learning examples at Apache Spark.

### HDP Cluster Requirement

Spark 1.3.1 can be configured on any HDP 2.3 cluster whether it is a multi node cluster or a single node HDP Sandbox.

The instructions in this guide assumes you are using the latest Hortonworks Sandbox

### Run the Spark Pi Example

To test compute intensive tasks in Spark, the Pi example calculates pi by “throwing darts” at a circle. The example points in the unit square ((0,0) to (1,1)) and sees how many fall in the unit circle. The fraction should be pi/4, which is used to estimate Pi.

To calculate Pi with Spark:

  1. **Change to your Spark directory and become spark OS user:**

```bash
cd /usr/hdp/current/spark-client  
su spark
```

  2. **Run the Spark Pi example in yarn-client mode:**

```bash
./bin/spark-submit --class org.apache.spark.examples.SparkPi --master yarn-client --num-executors 3 --driver-memory 512m --executor-memory 512m --executor-cores 1 lib/spark-examples*.jar 10
```

**Note:** The Pi job should complete without any failure messages and produce output similar to below, note the value of Pi in the output message:
![](https://www.dropbox.com/s/i93qmcvjmzue1ho/Screenshot%202015-07-20%2014.48.48.png?dl=1)

### Using WordCount with Spark

#### Copy input file for Spark WordCount Example

Upload the input file you want to use in WordCount to HDFS. You can use any text file as input. In the following example, log4j.properties is used as an example:

As user spark:

```bash
hadoop fs -copyFromLocal /etc/hadoop/conf/log4j.properties /tmp/data
```

### Run Spark WordCount

To run WordCount:

#### Run the Spark shell:

```bash
./bin/spark-shell --master yarn-client --driver-memory 512m --executor-memory 512m
```

Output similar to below displays before the Scala REPL prompt, scala>:
![](https://www.dropbox.com/s/f5bq5biq88gc2bs/Screenshot%202015-07-20%2014.50.31.png?dl=1)

#### At the Scala REPL prompt enter:

```scala
val file = sc.textFile("/tmp/data")
val counts = file.flatMap(line => line.split(" ")).map(word => (word, 1)).reduceByKey(_ + _)
counts.saveAsTextFile("/tmp/wordcount")
```

##### Viewing the WordCount output with Scala Shell

To view the output in the scala shell:

```scala
counts.count()
```

![](https://www.dropbox.com/s/0j1yk2okljlqx3k/Screenshot%202015-07-20%2014.55.13.png?dl=1)

To print the full output of the WordCount job:

```scala
counts.toArray().foreach(println)
```

![](https://www.dropbox.com/s/942krdi729qbkzx/Screenshot%202015-07-20%2014.57.10.png?dl=1)

##### Viewing the WordCount output with HDFS

To read the output of WordCount using HDFS command:
Exit the scala shell.

```scala
exit
```

View WordCount Results:

```bash
hadoop fs -ls /tmp/wordcount
```

It should display output similar to:

![](https://www.dropbox.com/s/tg17aqu3kieb9wa/Screenshot%202015-07-20%2014.58.22.png?dl=1)

Use the HDFS cat command to see the WordCount output. For example,

```bash
hadoop fs -cat /tmp/wordcount/part-00000
```

![](https://www.dropbox.com/s/xed8ikl35jj8dx7/Screenshot%202015-07-20%2014.59.10.png?dl=1)

##### Using Spark DataFrame API

With Spark 1.3.1, DataFrame API is a new feature. DataFrame API provide easier access to data since it looks conceptually like a Table and a lot of developers from Python/R/Pandas are familiar with it.

Let's upload people text file to HDFS

```bash
cd /usr/hdp/current/spark-client

su spark
hdfs dfs -copyFromLocal examples/src/main/resources/people.txt people.txt

hdfs dfs -copyFromLocal examples/src/main/resources/people.json people.json
```

![](https://www.dropbox.com/s/g8igatmz9v7n4hy/Screenshot%202015-07-20%2015.01.49.png?dl=1)

Then let's launch the Spark Shell

```bash
cd /usr/hdp/current/spark-client
su spark
./bin/spark-shell --num-executors 2 --executor-memory 512m --master yarn-client
```
At the Spark Shell type the following:

```scala
val df = sqlContext.jsonFile("people.json")
```

This will produce and output such as

![](https://www.dropbox.com/s/7dk2nsnfrvkv4y4/Screenshot%202015-07-20%2015.04.33.png?dl=1)

**Note:** The highlighted output shows the inferred schema of the underlying people.json.

Now print the content of DataFrame with df.show

```scala
df.show
```

![](https://www.dropbox.com/s/keq8tjfkz4fjfjb/Screenshot%202015-07-20%2015.06.29.png?dl=1)

##### Data Frame API examples

```scala
import org.apache.spark.sql.functions._
// Select everybody, but increment the age by 1
df.select(df("name"), df("age") + 1).show()
```
![](https://www.dropbox.com/s/n8lmf0lg4ed6t54/Screenshot%202015-07-20%2015.13.59.png?dl=1)

```scala
// Select people older than 21
df.filter(df("age") > 21).show()
```

![](https://www.dropbox.com/s/2yjemit47m1eo3v/Screenshot%202015-07-20%2015.14.47.png?dl=1)

```scala
// Count people by age
df.groupBy("age").count().show()
```

![](https://www.dropbox.com/s/y5mk0zdxzi8si0t/Screenshot%202015-07-20%2015.15.41.png?dl=1)

##### Programmatically Specifying Schema

```scala
import org.apache.spark.sql._
val sqlContext = new org.apache.spark.sql.SQLContext(sc)
val people = sc.textFile("people.txt")
val schemaString = "name age"
import org.apache.spark.sql.types.{StructType,StructField,StringType}
val schema = StructType(schemaString.split(" ").map(fieldName => StructField(fieldName, StringType, true)))
```

![](https://www.dropbox.com/s/21m81un74ml65dd/Screenshot%202015-07-20%2015.18.02.png?dl=1)

```scala
val rowRDD = people.map(_.split(",")).map(p => Row(p(0), p(1).trim))
val peopleDataFrame = sqlContext.createDataFrame(rowRDD, schema)
peopleDataFrame.registerTempTable("people")
val results = sqlContext.sql("SELECT name FROM people")
results.map(t => "Name: " + t(0)).collect().foreach(println)
```
This will produce an output like

![](https://www.dropbox.com/s/t3x20c5fs2dh2o1/Screenshot%202015-07-20%2015.19.49.png?dl=1)

### Running Hive 0.13.1 UDF

Hive 0.13.1 provides a new built-in UDF collect_list(col) which returns a list of objects with duplicates.
The below example reads and write to HDFS under Hive directories. In a production environment one needs appropriate HDFS permission. However for evaluation you can run all this section as hdfs user.

Before running Hive examples run the following steps:

#### Launch Spark Shell on YARN cluster

```scala
su hdfs
./bin/spark-shell --num-executors 2 --executor-memory 512m --master yarn-client
```
#### Create Hive Context

```scala
val hiveContext = new org.apache.spark.sql.hive.HiveContext(sc)
```
You should see output similar to the following:

![](https://www.dropbox.com/s/aga459qghah9x15/Screenshot%202015-07-20%2015.24.12.png?dl=1)

#### Create Hive Table

```scala
hiveContext.sql("CREATE TABLE IF NOT EXISTS TestTable (key INT, value STRING)")
```

You should see output similar to the following:

![](https://www.dropbox.com/s/h23w2qf99arqfjk/Screenshot%202015-07-20%2015.25.34.png?dl=1)

#### Load example KV value data into Table

    scala> hiveContext.sql("LOAD DATA LOCAL INPATH 'examples/src/main/resources/kv1.txt' INTO TABLE TestTable")

You should see output similar to the following:

![](https://www.dropbox.com/s/pqu1u95ep91omna/Screenshot%202015-07-20%2015.26.53.png?dl=1)

#### Invoke Hive collect_list UDF

```scala
hiveContext.sql("from TestTable SELECT key, collect_list(value) group by key order by key").collect.foreach(println)
```
You should see output similar to the following:

![](https://www.dropbox.com/s/3j0qfu95gxdvg2u/Screenshot%202015-07-21%2010.40.04.png?dl=1)

### Read & Write ORC File Example

In this tech preview, we have implemented full support for ORC files with Spark. We will walk through an example that reads and write ORC file and uses ORC structure to infer a table.

### ORC File Support

#### Create a new Hive Table with ORC format

```scala
hiveContext.sql("create table orc_table(key INT, value STRING) stored as orc")
```

#### Load Data into the ORC table

```scala
hiveContext.sql("INSERT INTO table orc_table select * from testtable")
```

#### Verify that Data is loaded into the ORC table

```scala
hiveContext.sql("FROM orc_table SELECT *").collect().foreach(println)
```

![](https://www.dropbox.com/s/ufqqxnpq25uktt6/Screenshot%202015-07-21%2010.42.11.png?dl=1)

#### Read ORC Table from HDFS as HadoopRDD**

```scala
val inputRead = sc.hadoopFile("/apps/hive/warehouse/orc_table", classOf[org.apache.hadoop.hive.ql.io.orc.OrcInputFormat],classOf[org.apache.hadoop.io.NullWritable],classOf[org.apache.hadoop.hive.ql.io.orc.OrcStruct])
```
![](https://www.dropbox.com/s/ivc37b3vykknsfb/Screenshot%202015-07-21%2010.45.03.png?dl=1)

#### Verify we can manipulate the ORC record through RDD

```scala
val k = inputRead.map(pair => pair._2.toString)
val c = k.collect
```

You should see output similar to the following:

![](https://www.dropbox.com/s/yqk5a39iep23lby/Screenshot%202015-07-21%2010.46.20.png?dl=1)

#### **Copy example table into HDFS**

```bash
cd /usr/hdp/current/spark-client
su spark
hadoop dfs -put examples/src/main/resources/people.txt people.txt
```

#### Run Spark-Shell

```bash
./bin/spark-shell --num-executors 2 --executor-memory 512m --master yarn-client
```

on Scala prompt type the following, except for the comments

```scala
import org.apache.spark.sql.hive.orc._
import org.apache.spark.sql._
import org.apache.spark.sql.types.{StructType,StructField,StringType}
val hiveContext = new org.apache.spark.sql.hive.HiveContext(sc)
```
![](https://www.dropbox.com/s/6qinxmfpwdmvzon/Screenshot%202015-07-21%2010.54.29.png?dl=1)

Load and register the spark table

```scala
val people = sc.textFile("people.txt")
val schemaString = "name age"
val schema = StructType(schemaString.split(" ").map(fieldName => StructField(fieldName, StringType, true)))
val rowRDD = people.map(_.split(",")).map(p => Row(p(0), new Integer(p(1).trim)))
```
![](https://www.dropbox.com/s/xqmvdgs14b08yak/Screenshot%202015-07-21%2013.35.47.png?dl=1)

Infer table schema from RDD

```scala
val peopleSchemaRDD = hiveContext.applySchema(rowRDD, schema)
```
![](https://www.dropbox.com/s/n5heod5udrwsxm3/Screenshot%202015-07-21%2013.37.06.png?dl=1)

Create a table from schema

```scala
peopleSchemaRDD.registerTempTable("people")
val results = hiveContext.sql("SELECT * FROM people")
results.map(t => "Name: " + t.toString).collect().foreach(println)
```
![](https://www.dropbox.com/s/fngxh0pmxmcwdvb/Screenshot%202015-07-21%2013.43.49.png?dl=1)

Save Table to ORCFile

```scala
peopleSchemaRDD.saveAsOrcFile("people.orc")
```

Create Table from ORCFile

```scala
val morePeople = hiveContext.orcFile("people.orc")
morePeople.registerTempTable("morePeople")
```

![](https://www.dropbox.com/s/ub7o8isy6b0gfqk/Screenshot%202015-07-21%2013.46.51.png?dl=1)

Query from the table

```scala
hiveContext.sql("SELECT * from morePeople").collect.foreach(println)
```

![](https://www.dropbox.com/s/fo6ryfluwqn323i/Screenshot%202015-07-21%2013.47.58.png?dl=1)

### SparkSQL Thrift Server for JDBC/ODBC access

With this release SparkSQL’s thrift server provides JDBC access to SparkSQL.

  1. **Start Thrift Server**
From SPARK_HOME, start SparkSQL thrift server, Note the port value of the thrift JDBC server

```scala
su spark
./sbin/start-thriftserver.sh --master yarn-client --executor-memory 512m --hiveconf hive.server2.thrift.port=10001
```

  * **Connect to Thrift Server over beeline**
Launch beeline from SPARK_HOME

```scala
su spark
./bin/beeline
```

  * **Connect to Thrift Server & Issue SQL commands**
On beeline prompt

```sql
!connect jdbc:hive2://localhost:10001
```

Note this is example is without security enabled, so any username password should work.

Note, the connection may take a few second to be available and try show tables after a wait of 10-15 second in a Sandbox env.

```sql
show tables;
```
![](https://www.dropbox.com/s/cwgmi3dbhm6566y/Screenshot%202015-07-21%2013.53.04.png?dl=1)

type `Ctrl+C` to exit beeline.

  * **Stop Thrift Server**

```bash
./sbin/stop-thriftserver.sh
```
![](https://www.dropbox.com/s/b6mvjepkfuadkxi/Screenshot%202015-07-21%2013.55.40.png?dl=1)

### Spark Job History Server

Spark Job history server is integrated with YARN’s Application Timeline Server(ATS) and publishes job metrics to ATS. This allows job details to be available after the job finishes.

  1. **Start Spark History Server**

```bash
./sbin/start-history-server.sh
```

You can let the history server run, while you run examples and go to YARN resource manager page at [http://127.0.0.1:8088/cluster/apps](http://127.0.0.1:8088/cluster/apps) and see the logs of finished application with the history server.

  2. **Stop Spark History Server**

```bash
./sbin/stop-history-server.sh
```
![](https://www.dropbox.com/s/5p14qxkuqw6r9hg/Screenshot%202015-07-21%2014.00.10.png?dl=1)

Visit [http://hortonworks.com/tutorials](http://hortonworks.com/tutorials) for more tutorials on Apache Spark.
