---
layout: post
title: Covering the HBase
date: '2014-03-12T12:34:00.001-07:00'
author: Saptak Sen
tags:
- hbase
- hadoop
modified_time: '2014-03-12T15:11:18.054-07:00'
---

Apache HBase was initially developed by Powerset, a natural language search engine startup in 2006. Then in 2008 they contributed the code base to the Apache Software Foundation so that the broad community could develop and extend this key piece of software. Since then, HBase has seen major changes and massive adoption and is now firmly established as the defacto standard NoSQL database for Hadoop.

Lets explore why organizations choose HBase over other options and how they are gaining value from it.

###Why HBase?

HBase provides near real-time, random read and write access to tables (or to be more accurate "maps") storing billions of rows and millions of columns. It provides low latency access to a LOT of data, but how?

HBase is a columnar database, which means it pivots storage of data to a column as opposed to the traditional row based storage found in an RDBMS.  This approach not only saves storage space but also reduces the amount of disk that needs to be scanned during a query, getting us close to near real time access.  

With the advent of the web and ARM processor, the rate at which data is being created around us today is astounding and the billion row database is not that rare anymore.  Apache Hadoop has emerged as the platform of choice for many to store and process this data because of its efficient and low cost linear scale.  HBase builds upon this platform to provide a columnar data store for Hadoop.  

Making sense of this data in a single platform makes sense. With Hadoop 2 and YARN, HBase can now share this single platform with other processing  engines, so that you may run a pig script or analyze a stream of data at the same time (on the same data) as you are feeding online applications with real time access to HBase data.  A singular system not only optimizes use of data but also presents new opportunities.  

###How is HBase delivering value today?

There's a long list of organizations like Adobe, Meetup, Pacific Northwest National Laboratory, Stumbleupon, Twitter, Yahoo that have publicly and voluntarily [acknowledged](http://wiki.apache.org/hadoop/Hbase/PoweredBy) their use of HBase for variety of scenarios.

One of the most common use of Hbase is to provide a recommendations to website visitors. While the recommendations may be calculated using Pig or some other processing engine, serving this data up to thousands of visitors per second needs a store like HBase.  For instance, Groupon uses HBase to personalize deals to users to be more accurate and relevant. Pinterest uses HBase to deliver the right set of "Pins" to their users depending what who they follow.

[Facebook's](http://sites.computer.org/debull/A12june/facebook.pdf) has used HBase to modernize their entire messaging system, which combines messages, chat and email into a real-time conversation for Facebook users.

Facebook shared the following benefits of choosing HBase over numerous other NoSQL options at their disposal:

  * High write throughput

  * Low latency random reads

  * Elasticity

  * Cheap and fault tolerant

  * Strong consistency within a data center

  * Experience with HDFS

Many other organizations have this exact same set of bases to cover when they require an enterprise grade massively scalable data platform.

###HBase: The future is written

As with any broad community open source project, the pace of innovation within the HBase community is astounding.  Here are few of the many improvements to HBase in just last six months:

####High Availability

  * Improvements in Mean-Time-To-Recovery (MTTR) for even quicker failover for even better High Availability. In HBase, the process of detecting a failure and the subsequent recovery is automatic.

  * HBase Snapshots, which allows users to take snapshots of an HBase table, clone it, copy it to a different cluster and restore it, etc.

####Performance

  * Compaction improvements to reduce the stress on the cluster during compactions by taking into account application's write & read patterns.

####Security

  * Cell level security allows HBase to secure data at the cell level in addition to tables and column families that was available before.

####Developer Experience

  * Data Type improvements providing a uniform way to represent data in HBase. These types can safely be used in HBase row keys, eliminating a source of bugs in HBase schema design.

  * Modularization of HBase for easy consumption in downstream projects. That means your applications that depend on the HBase client jar no longer bring along all the dependencies of the server module.

####Manageability

  * Metrics framework overhaul to allow monitoring HBase in real world deployments. The new system helps better integrate with management tools such as Apache Ambari for monitoring.

####Compatibility

  * Wire compatibility with releases after 0.96. to make it easy to do rolling upgrades and transparent upgrades for client applications.

  * Native HBase for Microsoft Windows.

The future of HBase based on all the work the community is doing is very exciting. Here are a few projects which hint the breadth and ambition of the work going on:

  * [Hoya](https://github.com/hortonworks/hoya) is a tool that is used to provision and flex HBase clusters on demand in a[ YARN](http://hortonworks.com/hadoop/yarn) cluster.

  * [Apache Phoenix](http://phoenix.incubator.apache.org/) is an effort to provide SQL support for HBase.

  * [Apache Ambari](http://hortonworks.com/hadoop/ambari) can now deploy and manage HBase in addition to all the other Hadoop components.

  * There are also platforms built on top of HBase like[ Kiji](http://www.kiji.org/) that provides user easy ways of developing applications against HBase.

The ecosystem of tools and software to complement HBase has been growing with it. This makes consumption of HBase easier, drives further HBase adoption, and encourages investment back into it.

In summary, HBase has come a long way and can be used reliably for a variety of enterprise scenarios. It has an exciting roadmap in front of it ensuring it's popularity for a long time.
