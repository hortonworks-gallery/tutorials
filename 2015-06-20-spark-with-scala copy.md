---
layout: post
title: Exploring Spark with Scala
date: '2014-06-20T12:34:00.001-07:00'
author: Saptak Sen
tags:
- spark
- hadoop
modified_time: '2014-06-20T15:11:18.054-07:00'
---
In this section we are going to walk through the process of using Scala and Apache Spark to interactively analyze data on a Apache Hadoop Cluster.

By the end of this tutorial, you will have learned:

  1. How to interact with Apache Spark through an interactive Spark shell
  2. How to read a text file from HDFS and create a RDD
  3. How to interactively analyze a data set through a rich set of Spark API operations

Let’s open a shell to our Sandbox through SSH:

![](https://www.dropbox.com/s/tzsxvsnxfo26jn7/Screenshot_2015-04-13_07_58_43.png?dl=1)

The default password is `hadoop` if you are login in into a Sandbox running on your machine.

Now let’s start the Spark Shell

```bash
spark-shell --master yarn-client --driver-memory 512m --executor-memory 512m
```
![](https://www.dropbox.com/s/nmuwjfn7i6j0jia/Screenshot%202015-06-08%2008.06.32.png?dl=1)

There is a :sh command in the Spark shell that lets you run linux commands:

```scala
:sh sudo jps
```

![](https://www.dropbox.com/s/wypitay56i5xpwy/Screenshot%202015-06-08%2008.08.42.png?dl=1)

The res0 output that you see in my case stands for ‘result #0’.

Now, print the output of result #0, which is the output of the commandline `jps` in our case:

```scala
res0.show
```
![](https://www.dropbox.com/s/elzn77ewphy8eir/Screenshot%202015-06-08%2008.10.58.png?dl=1)

When we launched the Spark shell, more JVMs has been instantiated to support the Shell, namely the SparkSubmit and CoarseGrainedExecutorBackend.

The SparkSubmit is the driver for the 'Spark shell' application and the CoarseGrainedExecutorBackend is the Executor running to support our application.

You can always exit the Spark shell by pressing `CTRL+D`.

Next, let's get some data into the Sandbox by copy-pasting the following into a new file called `littlelog.csv`, and then save it on your sandbox in the hdfs `/tmp` directory:

    20120315 01:17:06,99.122.210.248,[http://www.acme.com/SH55126545/VD55170364,{7AAB8415-E803-3C5D-7100-E362D7F67CA7},homestead,fl,usa](http://www.acme.com/SH55126545/VD55170364,{7AAB8415-E803-3C5D-7100-E362D7F67CA7},homestead,fl,usa)

    20120315 01:34:46,69.76.12.213,[http://www.acme.com/SH55126545/VD55177927,{8D0E437E-9249-4DDA-BC4F-C1E5409E3A3B},coeur d alene,id,usa](http://www.acme.com/SH55126545/VD55177927,{8D0E437E-9249-4DDA-BC4F-C1E5409E3A3B},coeur d alene,id,usa)

    20120315 17:23:53,67.240.15.94,[http://www.acme.com/SH55126545/VD55166807,{E3FEBA62-CABA-11D4-820E-00A0C9E58E2D},queensbury,ny,usa](http://www.acme.com/SH55126545/VD55166807,{E3FEBA62-CABA-11D4-820E-00A0C9E58E2D},queensbury,ny,usa)

    20120315 17:05:00,67.240.15.94,[http://www.acme.com/SH55126545/VD55149415,{E3FEBA62-CABA-11D4-820E-00A0C9E58E2D},queensbury,ny,usa](http://www.acme.com/SH55126545/VD55149415,{E3FEBA62-CABA-11D4-820E-00A0C9E58E2D},queensbury,ny,usa)

    20120315 01:27:53,98.234.107.75,[http://www.acme.com/SH55126545/VD55179433,{49E0D2EE-1D57-48C5-A27D-7660C78CB55C},sunnyvale,ca,usa](http://www.acme.com/SH55126545/VD55179433,{49E0D2EE-1D57-48C5-A27D-7660C78CB55C},sunnyvale,ca,usa)

    20120315 02:09:38,75.85.165.38,[http://www.acme.com/SH55126545/VD55179433,{F6F8B460-4204-4C26-A32C-B93826EDCB99},san diego,ca,usa](http://www.acme.com/SH55126545/VD55179433,{F6F8B460-4204-4C26-A32C-B93826EDCB99},san diego,ca,usa)


![](https://www.dropbox.com/s/3djm8kuxtt3mri4/Screenshot%202015-06-08%2008.21.53.png?dl=1)

Put the file `littlelog.csv` into /tmp directory in hadoop:

```bash
hadoop fs -put ./littlelog.csv /tmp/
```
![](https://www.dropbox.com/s/kt2ee75ytn3kmfp/Screenshot%202015-06-08%2008.25.17.png?dl=1)

Now we have our data in HDFS, let's launch `spark-shell`

```
spark-shell --master yarn-client --driver-memory 512m --executor-memory 512m
```
![](https://www.dropbox.com/s/ry9ygu7c61ilcz7/Screenshot%202015-06-08%2008.33.54.png?dl=1)

In Spark, datasets are represented as a list of entries, where the list is broken up into many different partitions that are each stored on a different machine. Each partition holds a unique subset of the entries in the list. Spark calls datasets that it stores "Resilient Distributed Datasets" (RDDs).

So let's create a RDD from our `littlelog.csv`:

```scala
val file = sc.textFile("hdfs://sandbox.hortonworks.com:8020/tmp/littlelog.csv")
```
Now we have a freshly created RDD. We have to use an action operation like `collect()` to gather up the data into the drivers memory and then to print out the contents of the file:

```
file.collect().foreach(println)
```
![](https://www.dropbox.com/s/cclf7ttf45i7xtq/Screenshot%202015-06-08%2008.58.24.png?dl=1)

Remember doing a `collect()` action operation on a very large distributed RDD can cause your driver program to run out of memory and crash. So, do not use `collect()` except for when you are prototyping your Spark program on a small dataset.

Another way to print the content of the RDD is

```
file.toArray.foreach(println)
```
![](https://www.dropbox.com/s/odczf5d1ipjw2fw/Screenshot%202015-06-10%2007.33.44.png?dl=1)

In fact you can easily discover other methods that apply to this RDD by tab auto-completion.

Type the name of the RDD followed by a `.`, in our case it's `file.` and the press the `<TAB>` key.

![](https://www.dropbox.com/s/lvg7fcmj7a728ni/Screenshot%202015-06-10%2007.37.45.png?dl=1)

Now let’s extract some information from this data.

Let’s create a map where the state is the key and the number of visitors is the value.


Since state is the 6th element in each row of our text in `littlelog.csv` (index 5), we need to use a map operator to pass in the lines of text to a function that will parse out the 6th element and store it in a new RDD containing two elements as the key, then count the number of times it appears in the set and provide that number as the value in the second element of this new RDD.

By using the Spark API operator map, we have created or transformed our original RDD into a newer one.

So let’s do it step by step. First let’s filter out the blank lines.

```scala
val fltr = file.filter(_.length > 0)
```
![](https://www.dropbox.com/s/edl1v0bb04bde5s/Screenshot%202015-06-08%2009.22.54.png?dl=1)

WAIT! What is that _ doing there? _ is a shortcut or wildcard in Scala that essentially means ‘whatever happens to be passed to me’. So in the above code the _ stands for each row of our file RDD and we are saying fltr equals a new RDD that is composed of each row with a length > 0.

So, we are invoking the method length on an unknown ‘whatever’ and trusting that Scala will figure out that the thing in each row of the file RDD is actually a String that supports the length operator.

So, in other words within the parenthesis of our filter method we are defining the argument: ‘whatever’, and the logic to be applied to it.

This pattern of constructing a function within the argument to a method is one of the fundamental characteristics of Scala and once you get used to it, it will make sense and speed up your programming a lot.

Then let’s split the line into individual columns seperated by space and then let’s grab the 5th columns

```scala
val keys = fltr.map(_.split(",")).map(a => a(5))
```
![](https://www.dropbox.com/s/3kl2r73fj3hex33/Screenshot%202015-06-08%2009.25.24.png?dl=1)

Notice that we are using the ‘whatever’ shortcut again. This time each row of the fltr RDD is having the split(“,”) method called on it, resulting in an anonymous RDD which we are then invoking map on and defining a function with the strange syntax => which stands for, ‘what is before me is the variable name (the type is inferred), what is after me is what you do to it’. In this case, each row (an array) in the anonymous RDD created by split is, in turn, assigned to the variable ‘a’ and then we extract the 5th element from it, which ends up being added to the named RDD called ‘keys’ we declared at the start of the line of code.

Then let’s print out the values of the key.

```scala
keys.collect().foreach(println)
```
![](https://www.dropbox.com/s/38czyv8k6z5knqt/Screenshot%202015-06-08%2009.27.35.png?dl=1)

Notice that some of the states are not unique and repeat. We need to count how many times each key (state) appears in the log.

Now let’s generate a key-value pair for each state as the key and the corresponding value as 1.

```scala
val stateCnt = keys.map(key => (key,1))
```
![](https://www.dropbox.com/s/2ydp5z7ndm8h3gh/Screenshot%202015-06-08%2009.29.05.png?dl=1)

Next, we will iterate through each row of the stateCnt RDD and pass their contents to a utility method available to our RDD that counts the distinct number of rows containing each key

```scala
val lastMap = stateCnt.countByKey
```
![](https://www.dropbox.com/s/wg8fojy5x5zem84/Screenshot%202015-06-08%2009.33.56.png?dl=1)

Now, let’s print out the result.

```scala
lastMap.foreach(println)
```

Result: a listing of state abbreviations and the count of how many times visitors from that state hit our website.

    (ny,2)
    (ca,2)
    (fl,1)
    (id,1)

![](https://www.dropbox.com/s/fu4n9h6u257d3ge/Screenshot%202015-06-08%2009.34.58.png?dl=1)

Note that at this point you still have access to all the RDDs you have created during this session. You can reprocess any one of them, for instance, again printing out the values contained in the keys RDD:

```scala
keys.collect().foreach(println)
```
![](https://www.dropbox.com/s/6t85jvmyt7ud1xr/Screenshot%202015-06-08%2009.35.58.png?dl=1)

I hope this has proved informative and that you have enjoyed this simple example of how you can interact with Data on HDP using Scala and Apache Spark.
