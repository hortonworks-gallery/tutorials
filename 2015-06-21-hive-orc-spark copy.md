---
layout: post
title: Using Hive and ORC with Apache Spark
date: '2014-06-21T12:34:00.001-07:00'
author: Saptak Sen
tags:
- hive
- hadoop
- spark
modified_time: '2014-06-21T15:11:18.054-07:00'
---
In this tutorial, we will explore how you can access and analyze data on Hive from Spark.

Spark SQL uses the Spark engine to execute SQL queries either on data sets persisted in HDFS or on existing RDDs. It allows you to manipulate data with SQL statements within a Spark program.

### Prerequisite

The only prerequiste for this tutorial is the latest [Hortonworks Sandbox](http://hortonworks.com/sandbox) installed on your computer or on the [cloud](http://hortonworks.com/blog/hortonworks-sandbox-azure/).

In case you are running an Hortonworks Sandbox with an earlier version of Apache Spark, for the instruction in [this]() tutorial to install the Apache Spark 1.3.1.

### Getting the dataset

To begin, login in to Hortonworks Sandbox through SSH:

![](https://www.dropbox.com/s/tzsxvsnxfo26jn7/Screenshot_2015-04-13_07_58_43.png?dl=1)

The default password is `hadoop`.


Now let's download the dataset with the command below:

```bash
wget http://hortonassets.s3.amazonaws.com/tutorial/data/yahoo_stocks.csv
```

![](https://www.dropbox.com/s/x5s6hwx1tin4wbu/Screenshot%202015-05-28%2008.49.00.png?dl=1)

and copy the downloaded file to HDFS:

```bash
hadoop fs -put ./yahoo_stocks.csv /tmp/
```
![](https://www.dropbox.com/s/pjv0zrrg775yy5w/Screenshot%202015-05-28%2008.49.55.png?dl=1)


### Starting the Spark shell

Use the command below to launch the Scala REPL for Apache Spark:

```bash
./bin/spark-shell --master yarn-client --driver-memory 512m --executor-memory 512m
```
![](https://www.dropbox.com/s/zjipmm9q9viltrt/Screenshot%202015-05-28%2008.53.08.png?dl=1)

Notice it is already starting with Hive integration as we have preconfigured it on the Hortonworks Sandbox.

Before we get started with the actual analytics lets import some of the libraries we are going to use below.

```scala
import org.apache.spark.sql.hive.orc._
import org.apache.spark.sql._
```

![](https://www.dropbox.com/s/qupxygona3u1720/Screenshot%202015-05-21%2011.43.56.png?dl=1)

### Creating HiveContext

HiveContext is an instance of the Spark SQL execution engine that integrates with data stored in Hive. The more basic SQLContext provides a subset of the Spark SQL support that does not depend on Hive. It reads the configuration for Hive from hive-site.xml on the classpath.

```scala
val hiveContext = new org.apache.spark.sql.hive.HiveContext(sc)
```
![](https://www.dropbox.com/s/evc1qupcrlebnu9/Screenshot%202015-05-21%2011.47.33.png?dl=1)

### Creating ORC tables

ORC is a self-describing type-aware columnar file format designed for Hadoop workloads. It is optimized for large streaming reads and with integrated support for finding required rows fast. Storing data in a columnar format lets the reader read, decompress, and process only the values required for the current query. Because ORC files are type aware, the writer chooses the most appropriate encoding for the type and builds an internal index as the file is persisted.

Predicate pushdown uses those indexes to determine which stripes in a file need to be read for a particular query and the row indexes can narrow the search to a particular set of 10,000 rows. ORC supports the complete set of types in Hive, including the complex types: structs, lists, maps, and unions.

Specifying `as orc` at the end of the SQL statement below ensures that the Hive table is stored in the ORC format.

```scala
hiveContext.sql("create table yahoo_orc_table (date STRING, open_price FLOAT, high_price FLOAT, low_price FLOAT, close_price FLOAT, volume INT, adj_price FLOAT) stored as orc")
```

![](https://www.dropbox.com/s/pddrk6ga15wtr2e/Screenshot%202015-05-28%2009.33.34.png?dl=1)

### Loading the file and creating a RDD

**Resilient Distributed Dataset**(RDD), is an immutable collection of objects that is partitioned and distributed across multiple physical nodes of a YARN cluster and that can be operated in parallel.

Once an RDD is instantiated, you can apply a [series of operations](https://spark.apache.org/docs/1.2.0/programming-guide.html#rdd-operations). All operations fall into one of two types:[transformations](https://spark.apache.org/docs/1.2.0/programming-guide.html#transformations) or [actions](https://spark.apache.org/docs/1.2.0/programming-guide.html#actions). **Transformation** operations, as the name suggests, create new datasets from an existing RDD and build out the processing DAG that can then be applied on the partitioned dataset across the YARN cluster. An **Action** operation, on the other hand, executes DAG and returns a value.

Normally, we would have directly loaded the data in the ORC table we created above and then created an RDD from the same, but in this to cover a little more surface of Spark we will create an RDD directly from the CSV file on HDFS and then apply Schema on the RDD and write it back to the ORC table.

With the command below we instantiate an RDD:

```scala
val yahoo_stocks = sc.textFile("hdfs://sandbox.hortonworks.com:8020/tmp/yahoo_stocks.csv")
```
![](https://www.dropbox.com/s/lddom9ac9jftgbx/Screenshot%202015-05-21%2012.08.16.png?dl=1)

###Separating the header from the data

Let's assign the first row of the RDD above to a new variable
```scala
val header = yahoo_stocks.first
```
![](https://www.dropbox.com/s/3zlxwalwx5tlxml/Screenshot%202015-05-28%2010.14.21.png?dl=1)

Let's dump this new RDD in the console to see what we have here:

```scala
header
```
![](https://www.dropbox.com/s/36w14xmh7218e1s/Screenshot%202015-05-28%2010.22.10.png?dl=1)

Now we need to separate the data into a new RDD where we do not have the header above and :

```scala
val data = yahoo_stocks.filter(_(0) != header(0))
```
check the first row to seen it's indeed only the data in the RDD

```scala
data.first
```





### Creating a schema

There's two ways of doing this.

```scala
case class YahooStockPrice(date: String, open: Float, high: Float, low: Float, close: Float, volume: Integer, adjClose: Float)
```

![](https://www.dropbox.com/s/rto78tzekm8pum3/Screenshot%202015-05-28%2011.54.06.png?dl=1)

### Attaching the schema to the parsed data
Create an RDD of Yahoo Stock Price objects and register it as a table.

```scala
val stockprice = data.map(_.split(",")).map(row => YahooStockPrice(row(0), row(1).trim.toFloat, row(2).trim.toFloat, row(3).trim.toFloat, row(4).trim.toFloat, row(5).trim.toInt, row(6).trim.toFloat)).toDF()
```

![](https://www.dropbox.com/s/ma418p8gnx3uekr/Screenshot%202015-05-28%2011.59.33.png?dl=1)

Let's verify that the data has been correctly parsed by the statement above by dumping the first row of the RDD containing the parsed data:

```scala
stockprice.first
```

![](https://www.dropbox.com/s/ynbm95rsi4oyto6/Screenshot%202015-05-28%2014.02.58.png?dl=1)

if we want to dump more all the rows, we can use

```scala
stockprice.show
```

![](https://www.dropbox.com/s/nh48ic4un84ct2g/Screenshot%202015-05-28%2014.08.33.png?dl=1)

To verify the schema, let's dump the schema:

```scala
stockprice.printSchema
```
![](https://www.dropbox.com/s/hr49pyn8jccvlwi/Screenshot%202015-05-28%2014.12.38.png?dl=1)

### Registering a temporary table

Now let's give this RDD a name, so that we can use it in Spark SQL statements:

```scala
stockprice.registerTempTable("yahoo_stocks_temp")
```
![](https://www.dropbox.com/s/x3hc5u39v9cr332/Screenshot%202015-05-28%2014.19.30.png?dl=1)

### Querying against the table

Now that our schema'd RDD with data has a name we can use Spark SQL commands to query it. Remember the table below is not a Hive table, it is just a RDD we are querying with SQL.

```scala
val results = sqlContext.sql("SELECT * FROM yahoo_stocks_temp")
```
![](https://www.dropbox.com/s/tyhykbc2e51xrom/Screenshot%202015-05-28%2016.24.14.png?dl=1)

The resultset returned from the Spark SQL query is now loaded in the `results` RDD. Let's pretty print it out on the command line.

```scala
results.map(t => "Stock Entry: " + t.toString).collect().foreach(println)
```
![](https://www.dropbox.com/s/nm4zqg7ouzcwku7/Screenshot%202015-05-21%2013.08.32.png?dl=1)

### Saving as an ORC file

Now let's persist back the RDD into the Hive ORC table we created before.

```scala
results.saveAsOrcFile("yahoo_stocks_orc")
```
![](https://www.dropbox.com/s/a6d1ydhtw9kg4fi/Screenshot%202015-05-28%2016.52.44.png?dl=1)

### Reading the ORC file

Let's now try to read back the ORC file, we just created back into an RDD. But before we do so, we need a hiveContext:

```scala

val hiveContext = new org.apache.spark.sql.hive.HiveContext(sc)
```
![](https://www.dropbox.com/s/sb99lpm8jtyyicu/Screenshot%202015-05-28%2017.23.06.png?dl=1)

now we can try to read the ORC file with:

```scala
val yahoo_stocks_orc = hiveContext.orcFile("yahoo_stocks_orc")
```
![](https://www.dropbox.com/s/xu0aygiph6ntdbu/Screenshot%202015-05-28%2017.24.05.png?dl=1)

Let's register it as a temporary in-memory table mapped to the ORC table:

```scala
yahoo_stocks_orc.registerTempTable("orcTest")
```
![](https://www.dropbox.com/s/6nx8a7axy8qdgo4/Screenshot%202015-05-28%2017.24.53.png?dl=1)

Now we can verify weather we can query it back:

```scala
hiveContext.sql("SELECT * from orcTest").collect.foreach(println)
```
![](https://www.dropbox.com/s/1f92sc2yclqovhd/Screenshot%202015-05-28%2017.26.08.png?dl=1)

Voila! We just did a round trip of persisting and reading data to and from Hive ORC using Spark SQL.

Hope this tutorial illustrated some of the ways you can integrate Hive and Spark.
