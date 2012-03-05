---
layout: post
title: "Groovy primitive type cast gotcha when upgrading to Grails 2.0"
date: 2012-02-23 18:13
comments: true
published: true
categories: 
- Grails
- Groovy
---

When upgrading our Grails applications from 1.3.x to 2.0, I got burned by this slight change of semantics in Groovy 1.8.x:

Groovy 1.7.9:

``` java
groovy:000> null as int
===> null
```

Groovy 1.8.0:

``` java
groovy:000> null as float         
ERROR org.codehaus.groovy.runtime.typehandling.GroovyCastException:
Cannot cast object 'null' with class 'null' to class 'float'. Try 'java.lang.Float' instead
```

Turns out this can bite you in quite many places. A very common pattern occurring in our taglibs was to cast attributes, even if they were optional. Like this:

``` java
def myTag = { attrs, body ->
	def height = attrs.remove("height") as int
}
```

Which would've simply assigned _null_ to _height_ in Grails 1.3.x. Luckily you can just change your casts to boxed versions of the primitives (i.e. _as Integer_, _as Long_ etc) and be done with it.

``` java
def myTag = { attrs, body ->
	def height = attrs.remove("height") as Integer
}
```
