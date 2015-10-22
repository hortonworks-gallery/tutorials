---
layout: post
title: Installing Apache Spark 1.2.0 on HDP 2.2
date: '2014-06-17T12:34:00.001-07:00'
author: Saptak Sen
tags:
- spark
- hadoop
modified_time: '2014-06-17T15:11:18.054-07:00'
---
After SSH'ing into your Sandbox, use `su` to login as root.
Now let's get the bits using the command

```
wget http://public-repo-1.hortonworks.com/HDP-LABS/Projects/spark/1.2.0/spark-1.2.0.2.2.0.0-82-bin-2.6.0.2.2.0.0-2041.tgz
```

![](https://www.dropbox.com/s/a25v1m9mryyxwoh/Screenshot%202015-06-07%2021.52.30.png?dl=1)

Then uncompress the archive with

```
tar xvfz spark-1.2.0.2.2.0.0-82-bin-2.6.0.2.2.0.0-2041.tgz
```

![](https://www.dropbox.com/s/i3kpjeenqku213n/Screenshot%202015-06-07%2022.03.56.png?dl=1)

Move the resulting folder to `/opt` as just `spark`

```
mv spark-1.2.0.2.2.0.0-82-bin-2.6.0.2.2.0.0-2041 /opt/spark
```

also for good measure create a symbolic link from `/usr/hdp/current/spark-client` into `/opt/spark`

```
ln -s /opt/spark /usr/hdp/current/spark-client
```
![](https://www.dropbox.com/s/cpid455mhbrwxmu/Screenshot%202015-06-07%2022.15.28.png?dl=1)

Create a bash script called spark.sh in the `/etc/profile.d` with the following lines

```
export SPARK_HOME=/usr/hdp/current/spark-client
export YARN_CONF_DIR=/etc/hadoop/conf
export PATH=/usr/hdp/current/spark-client/bin:$PATH
```


Then run the script with

```
source /etc/profile.d/spark.sh
```


Create a file `SPARK_HOME/conf/spark-defaults.conf` and add the following settings:

```  
spark.driver.extraJavaOptions -Dhdp.version=2.2.0.0–2041
spark.yarn.am.extraJavaOptions -Dhdp.version=2.2.0.0–2041
```

Now before we can test run a sample, use your browser to navigate to `http://<hostname>:8080` for the Ambari interface. Use `admin` and `admin` as username and password respectively to login.

![](https://www.dropbox.com/s/bw92iidkfhj124r/Screenshot%202015-06-07%2023.12.03.png?dl=1)

After you login ensure necessary services like Datanode, Nodemanager, ResourceManager, Namenode, HiveServer2, etc are running. If they are stopped, start them and wait for them to start before the next step.

![](https://www.dropbox.com/s/gnjxv5yjdmdqs65/Screenshot%202015-06-07%2023.15.49.png?dl=1)

Now we can test the validity of our installation by running the sample Pi calculator

```
cd /opt/spark
./bin/spark-submit --verbose --class org.apache.spark.examples.SparkPi --master yarn-cluster --num-executors 3 --driver-memory 512m --executor-memory 512m --executor-cores 1 lib/spark-examples*.jar 10
```

![](https://www.dropbox.com/s/a6hb26bdd65gc32/Screenshot%202015-06-07%2023.18.31.png?dl=1)

You can check the value of the result by navigating to the URL above. Make sure you replace `sandbox.hortonworks.com` with the actual DNS name of your Sandbox VM. Also ensure that you have opened the port 8088 either on Azure or your Virtualization software.
