---
layout: post
title: Deploying a Hadoop cluster on EC2
date: '2014-02-04T12:34:00.001-07:00'
author: Saptak Sen
tags:
- ec2
- hadoop
modified_time: '2014-02-04T15:11:18.054-07:00'
---

In this post, we’ll walk through the process of deploying an Apache Hadoop 2 cluster on the EC2 cloud service offered by Amazon Web Services (AWS), using [Hortonworks Data Platform](http://hortonworks.com/products/hdp).

Both EC2 and HDP offer many knobs and buttons to cater to your specific, performance, security, cost, data size, data protection and other requirements. I will not discuss most of these options in this blog as the goal is to walk through one particular path of deployment to get started.

Let’s go!

## Prerequisites

  * Amazon Web Services account with the ability to launch 7 large instances of EC2 nodes.
  * A Mac or a Linux machine. You could also use Windows but you will have to install additional software such as SSH clients and SCP clients, etc.
  * Lastly, we assume that you have basic familiarity with EC2 to the extent that you have created EC2 instances and SSH’d in.

## Step 1: Creating a Base AMI with all the OS level configuration common to all nodes

Navigate to your EC2 console from the AWS Dashboard and then click on ‘Launch Instance':  
![EC2 Dashboard](http://hortonassets.s3.amazonaws.com/ec2/ec2-2.jpg)

Let’s select the RHEL 64bit and go to the next step:

![SelectBaseImage](http://hortonassets.s3.amazonaws.com/ec2/ec2-3.jpg)

Let’s select a large instance with adequate processing power and memory:

![ImageSize](http://hortonassets.s3.amazonaws.com/ec2/ec2-4.jpg)

Here we adjust storage as required:

![Instance_Storage](http://hortonassets.s3.amazonaws.com/ec2/ec2-70.jpg)

We are ready for Review and Launch:

![ReviewAndLaunch](http://hortonassets.s3.amazonaws.com/ec2/ec2-7.jpg)

But, before you Launch the instance, make sure you have downloaded the private key. Keep the private key safe and Launch:

![PrivateKey](http://hortonassets.s3.amazonaws.com/ec2/ec2-8.jpg)

Everything looks good. Let’s view the instances.

![<Display Name>](http://hortonassets.s3.amazonaws.com/ec2/ec2-9.jpg)

Now that we have instance up and running, we will need the public DNS name to connect to it:

![<Display Name>](http://hortonassets.s3.amazonaws.com/ec2/ec2-10.jpg)

Let’s SSH in:

![SSH](http://hortonassets.s3.amazonaws.com/ec2/ec2-11.jpg)

Now let’s prep the instance:

![Prep](http://hortonassets.s3.amazonaws.com/ec2/ec2-15.jpg)

That was all the prep we need, so we are going to create a private AMI. Go to the EC2 console, select the instance and from the action menu select “Create Image”:

![AMI](http://hortonassets.s3.amazonaws.com/ec2/ec2-16.jpg)

Make sure you check ‘No reboot’ before you click Create Image, as we will like to continue to work on this instance:

![NoReboot](http://hortonassets.s3.amazonaws.com/ec2/ec2-82.jpg)

Wait for the creation of the AMI to be complete:

![<Display Name>](http://hortonassets.s3.amazonaws.com/ec2/ec2-19.jpg)

Let’s configure this instance for password-less SSH to all the other nodes in the cluster. The first step is to have the private key on this instance.

![<Display Name>](http://hortonassets.s3.amazonaws.com/ec2/ec2-21.jpg)

We will need to move the private key to .ssh folder and rename it to id_rsa:

![<Display Name>](http://hortonassets.s3.amazonaws.com/ec2/ec2-24.jpg)

Let’s provision the other nodes now:

![<Display Name>](http://hortonassets.s3.amazonaws.com/ec2/ec2-25.jpg)

Select the size of the node instances:

![<Display Name>](http://hortonassets.s3.amazonaws.com/ec2/ec2-26.jpg)

I will select 6 more nodes here with 3 nodes dedicated for all the management daemons and 4 nodes dedicated to data nodes. Then click on ‘Review and Launch':

![<Display Name>](http://hortonassets.s3.amazonaws.com/ec2/ec2-27.jpg)

Click on the “Launch” button:

![<Display Name>](http://hortonassets.s3.amazonaws.com/ec2/ec2-28.jpg)

Ensure, you are using the same key as before for the passwordless SSH to work between the Ambari node and the rst of the new nodes. Click on the ‘Launch Instance':

![<Display Name>](http://hortonassets.s3.amazonaws.com/ec2/ec2-29.jpg)

As the instances are getting launched, we will copy down to a text file the Private DNS names of all the instances we have launched so far:

![<Display Name>](http://hortonassets.s3.amazonaws.com/ec2/ec2-31.jpg)

We will end up with a list like below:

![<Display Name>](http://hortonassets.s3.amazonaws.com/ec2/ec2-32.jpg)

## Step 2: Customize the security groups to minimize attack surface area while not blocking essential communication channels

We have have to add rules to the security groups which was created by default when we launched the instances.

The first security group should have been created when we launched the first instance. We are running the Ambari server on this instance, so we have to ensure we can get to it and it can communicate with the rest of the instances that we launched later:

![<Display Name>](http://hortonassets.s3.amazonaws.com/ec2/ec2-56.jpg)

Then we also need to open up the ports for IPs internal to the datacenter:

![<Display Name>](http://hortonassets.s3.amazonaws.com/ec2/ec2-57.jpg)

## Step 3: Setting up Ambari

Get the bits of HDP and add it to the repo:

![<Display Name>](http://hortonassets.s3.amazonaws.com/ec2/ec2-35.jpg)

next we will refresh the repo:

![<Display Name>](http://hortonassets.s3.amazonaws.com/ec2/ec2-36.jpg)

Then we will install the Ambari server:

![<Display Name>](http://hortonassets.s3.amazonaws.com/ec2/ec2-37.jpg)

Agree to download bits:

![<Display Name>](http://hortonassets.s3.amazonaws.com/ec2/ec2-38.jpg)

Agree to download the key:

![<Display Name>](http://hortonassets.s3.amazonaws.com/ec2/ec2-39.jpg)

Ambari Server bits are installed:

![<Display Name>](http://hortonassets.s3.amazonaws.com/ec2/ec2-40.jpg)

Now, we will configure the bits:

![<Display Name>](http://hortonassets.s3.amazonaws.com/ec2/ec2-41.jpg)

Just accept all the all the default options for all the prompts by pressing Enter:

![<Display Name>](http://hortonassets.s3.amazonaws.com/ec2/ec2-42.jpg)

Let’s start the Ambari Server:

![<Display Name>](http://hortonassets.s3.amazonaws.com/ec2/ec2-47.jpg)

That’s it we are all set to use Ambari to bring up the cluster.

## Step 4: Using Ambari to deploy the cluster

Copy the public DNS name of the Ambari:

![<Display Name>](http://hortonassets.s3.amazonaws.com/ec2/ec2-48.jpg)

Navigate to port 8080 of the public DNS from your browser. You should see the login page of Ambari. The default username and password is ‘admin’ and ‘admin’ respectively:

![<Display Name>](http://hortonassets.s3.amazonaws.com/ec2/ec2-49.jpg)

This is where we start creating the cluster. Enter any cluster name of your choosing:

![<Display Name>](http://hortonassets.s3.amazonaws.com/ec2/ec2-50.jpg)

We are going to create a HDP 2.0 cluster:

![<Display Name>](http://hortonassets.s3.amazonaws.com/ec2/ec2-51.jpg)

Remember the list of private DNS names that you had copied down to a text file. We will pull out the list and paste it in the Target host input box. We will also upload the private key that we have been using on this page:

![<Display Name>](http://hortonassets.s3.amazonaws.com/ec2/ec2-52.jpg)

We are all set to go. These should all come back as green with no warnings:

![<Display Name>](http://hortonassets.s3.amazonaws.com/ec2/ec2-54.jpg)

At this stage, we need to decide what services we need:

![<Display Name>](http://hortonassets.s3.amazonaws.com/ec2/ec2-58.jpg)

For this demonstration, I will select everything, although in real life you want to be more judicious and select the bare minimum needed for your requirement:

![<Display Name>](http://hortonassets.s3.amazonaws.com/ec2/ec2-58.jpg)

After we are done selecting the services, it’s time to determine where they will run. Ambari is smart enough to suggest you reasonable suggestions, but if you have specific topology in mind you might want move these around:

![<Display Name>](http://hortonassets.s3.amazonaws.com/ec2/ec2-59.jpg)

Next step is to configure which nodes do you want to Data nodes and Clients to be. I like to have clients on multiple instances just for the convenience.

![<Display Name>](http://hortonassets.s3.amazonaws.com/ec2/ec2-61.jpg)

In the next step we will have to configure the credentials for some of the services. the ones where you will need to populate the credentials are marked by a number in the red background mark:

![<Display Name>](http://hortonassets.s3.amazonaws.com/ec2/ec2-63.jpg)

Once we are done with all the inputs, we are ready to review and then start the deployment:

![<Display Name>](http://hortonassets.s3.amazonaws.com/ec2/ec2-65.jpg)

At this point it will take a while ( ~ 30 mins) to complete the deployment and test the services:

![<Display Name>](http://hortonassets.s3.amazonaws.com/ec2/ec2-67.jpg)

Voila!! We now have a fully functional and tested cluster on EC2. Happy Hadooping!!!
