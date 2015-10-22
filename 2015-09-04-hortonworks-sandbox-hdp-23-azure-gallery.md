---
layout: post
title: Hortonworks Sandbox with HDP 2.3 is now available on Microsoft Azure Gallery
date: '2015-09-04T12:34:00.001-07:00'
author: Saptak Sen
tags:
- hadoop
- azure
- sandbox
modified_time: '2015-09-04T15:11:18.054-07:00'
---

We are excited to announce the general availability of [Hortonworks Sandbox](http://hortonworks.com/sandbox) with HDP 2.3 on [Microsoft Azure](http://azure.microsoft.com/en-us/) Gallery. Hortonworks Sandbox is already a very popular environment for developers, data scientists and administrators to learn and experiment with the latest innovations in [Hortonworks Data Platform](http://hortonworks.com/hdp).

The hundreds of innovations span across [Apache Hadoop](http://hortonworks.com/hadoop/), [Kafka](http://hortonworks.com/hadoop/kafka/), [Storm](http://hortonworks.com/hadoop/storm/), [Spark](http://hortonworks.com/hadoop/spark/), [Hive](http://hortonworks.com/hadoop/hive/), [Pig](http://hortonworks.com/hadoop/pig/), [YARN](http://hortonworks.com/hadoop/yarn/), [Ambari](http://hortonworks.com/hadoop/ambari/), [Falcon](http://hortonworks.com/hadoop/falcon/), [Ranger](http://hortonworks.com/hadoop/ranger/) and other components that make up HDP platform. We also provide tutorials to help you get a jumpstart on how to use HDP to implement an Open Enterprise Hadoop in &nbsp_place_holder;your organization.

Every component is updated, including some of the key technologies we added in HDP 2.3.

![](http://hortonworks.com/wp-content/uploads/2015/07/Asparagus-Chart.png)  


This guide walks you through using the Azure Gallery to quickly deploy [Hortonworks Sandbox](http://hortonworks.com/sandbox) on Microsoft Azure.

### Prerequisite:

  * A Microsoft Azure account – you can sign up for an evaluation account if you do not already have one.

### Guide

Start by logging into the Azure Portal with your Azure account: [https://portal.azure.com/](https://portal.azure.com/)

![](http://hortonassets.s3.amazonaws.com/tutorial/azure-sandbox/01.png)  


Navigate to the **MarketPlace** 

![](http://hortonassets.s3.amazonaws.com/tutorial/azure-sandbox/04.jpeg)  


Search for Hortonworks. Click on the Hortonworks Sandbox icon.

![](http://www.dropbox.com/s/ay9hehehqpdmyet/Screenshot%202015-09-03%2013.31.30.png?dl=1)  


To go directly to the Hortonworks Sandbox on Azure page navigate to [http://azure.microsoft.com/en-us/marketplace/partners/hortonworks/hortonworks-sandbox/](http://azure.microsoft.com/en-us/marketplace/partners/hortonworks/hortonworks-sandbox/)

This will launch the wizard to configure Hortonworks Sandbox for deployment.

![](http://www.dropbox.com/s/8t6ckv7iyac9v4c/Screenshot%202015-09-03%2013.31.58.png?dl=1)  

 You will need to note down the `hostname` and the `username`/`password` that you enter in the next steps to be able to access the Hortonworks Sandbox once deployed. Also ensure you select a Azure instance with size `A4` or more for optimal experience.

![](http://www.dropbox.com/s/jadtasw29wg42n6/Screenshot%202015-09-03%2013.32.52.png?dl=1)  
Click `Buy` if you agree with everything on this page.

![](http://www.dropbox.com/s/h7kc8i0wznm6v2a/Screenshot%202015-09-03%2013.33.10.png?dl=1)  

At this point it should take you back to the Azure portal home page where you can see the deployment in progress.

![](http://www.dropbox.com/s/8r9ucjjhrqew629/Screenshot%202015-09-03%2013.33.26.png?dl=1)  
You can see the progress in more details by clicking on `Audit`

![](http://www.dropbox.com/s/ky0iibwq6h73o80/Screenshot%202015-09-03%2013.36.06.png?dl=1)

Once the deployment completes you will see this page with configuration and status of you VM. Again it is important to note down the DNS name of your VM which you will use in the next steps.

![](http://www.dropbox.com/s/9niib0di7ucczs9/Screenshot%202015-09-03%2013.43.48.png?dl=1)  


If you scroll down you can see the Estimated spend and other metrics for your VM.

![](http://www.dropbox.com/s/ib277102ymza8zd/Screenshot%202015-09-03%2014.01.03.png?dl=1)  


Let’s navigate to the home page of your Sandbox by pointing your browser to the URL: `http://<hostname>.cloudapp.net:8888` , where `<hostname>` is the hostname you entered during configuration.


By navigating to port 8080 of your Hortonworks Sandbox on Azure you can access the Ambari interface for your Sandbox.

![](http://www.dropbox.com/s/l9u6iltfsuq1l13/Screenshot%202015-09-03%2014.03.46.png?dl=1)  


If you want a full list of tutorial that you can use with your newly minted Hortonworks Sandbox on Azure go to [http://hortonworks.com/tutorials](http://hortonworks.com/tutorials).

![](http://www.dropbox.com/s/4o9fx312v5qygn8/Screenshot%202015-09-03%2018.22.38.png?dl=1)  

HDP 2.3 leverages the Ambari Views Framework to deliver new user views and a breakthrough user experience for both cluster operators and data explorers.


Happy Hadooping with Hortonworks Sandbox!
