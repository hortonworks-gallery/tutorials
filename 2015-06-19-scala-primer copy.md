---
layout: post
title: A short primer on Scala
date: '2014-06-19T12:34:00.001-07:00'
author: Saptak Sen
tags:
- scala
- hadoop
modified_time: '2014-06-19T15:11:18.054-07:00'
---
Scala is relatively new language based on the JVM. The main difference between other “Object Oriented Languages” and Scala is that everything in Scala is an object. The primitive types that are defined in Java, such as int or boolean, are objects in Scala. Functions are treated as objects, too. As objects, they can be passed as arguments, allowing a functional programming approach to writing applications for Apache Spark.

If you have programmed in Java or C#, you should feel right at home with Scala with very little effort.

You can also run or compile Scala programs from commandline or from IDEs such as Eclipse.

To learn and experiment with data, I prefer the interactivity of the Scala shell. Let's launch the Spark shell as it is a fully capable Scala shell.

```bash
spark-shell --master yarn-client --driver-memory 512m --executor-memory 512m
```

![](https://www.dropbox.com/s/d21chg1rsetlimq/Screenshot%202015-06-08%2013.20.55.png?dl=1)

###Values

Values can be either mutable or immutable. Mutable values are expressed by the `var` keyword.

```scala
var a: Int = 5
a = a + 1
println(a)
```
![](https://www.dropbox.com/s/89ly7aj7yg6ssdb/Screenshot%202015-06-08%2012.26.39.png?dl=1)

Immutable values are expressed by the `val` keyword.

```scala
val b: Int = 7
b = b + 1 //Error
println(b)
```
![](https://www.dropbox.com/s/48flcm9q1zlg165/Screenshot%202015-06-08%2012.29.13.png?dl=1)

###Type Inference

Scala like Java is a strongly typed language. But, it is very smart in type inference, which alleviates the onus of specifying types from the programmers.

```scala
var c = 9
println(c.getClass)

val d = 9.9
println(d.getClass)

val e = "Hello"
println(e.getClass)
```

![](https://www.dropbox.com/s/cdhlibeiu41c1ww/Screenshot%202015-06-08%2012.52.36.png?dl=1)

###Functions

Functions are defined with the `def` keyword. In Scala, the last expression in the function body is returned without the need of the `return` keyword.

```scala
def cube(x: Int): Int = {
  val x2 = x * x * x
  x2
}

cube(3)
```
![](https://www.dropbox.com/s/0w9xcwfe180ylom/Screenshot%202015-06-08%2013.24.04.png?dl=1)

You can write the function more succinctly by leaving out the braces and the return type as return type can be easily inferred.

```scala
def cube(x: Int) = x*x*x
```
![](https://www.dropbox.com/s/nh66ybashw7exqf/Screenshot%202015-06-08%2013.27.58.png?dl=1)

###Anonymous Functions

Anonymous functions can be assigned as a `var` or `val` value. It can also be passed to and returned from other functions.

```scala
val sqr: Int => Int = x => x * x
sqr(2)
```
![](https://www.dropbox.com/s/00axm7yxhi4tfx0/Screenshot%202015-06-08%2013.50.40.png?dl=1)

Or, anonymous functions can be further shortened to

```scala
val thrice: Int => Int = _ * 3
thrice(15)
```
![](https://www.dropbox.com/s/e37sbhm7ie9j2il/Screenshot%202015-06-08%2013.58.41.png?dl=1)

where `_` is a the shorthand for whatever the input was.

###Collections

Scala has very convenient set of collections including Lists, Sets, Tuples, Iterators and Options. When you combine these data structures with anonymous functions and closures it is very expressive.

```
val strs = Array("This", "is", "happening")

strs.map(_.reverse)
```
![](https://www.dropbox.com/s/2b8f3vz68usndmu/Screenshot%202015-06-08%2014.38.48.png?dl=1)

```
strs.reduce(_+" "+_)
```

![](https://www.dropbox.com/s/vc17p0t06a5wfmy/Screenshot%202015-06-08%2014.28.31.png?dl=1)

This is far from comprehensive. To learn more visit [http://scala-lang.org](http://scala-lang.org)
