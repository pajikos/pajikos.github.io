---
title: "How To Create Maven Dependencies using DSL in Groovy"
date: "2015-09-04"
categories: 
  - "groovy"
---

This post shows how to create Maven's dependency elements (used in  pom.xml file) programmatically, using DSL (Domain-Specific Language).

You may find it useful, when you need to convert e.g. an old Java project (with dozens of required dependency jar files in one folder) into Maven project, i.e. create a new pom.xml file with all necessary dependency elements.

Of cause, it exists a lot of different ways how to do it, but I solved this task using dynamic nature of Groovy and its metaprogramming capabilities, which makes it attractive for building DSLs.

> The basic idea of a domain specific language (DSL) is a computer language that's targeted to a particular kind of problem,... M. Fowler

If you are not familiar either with DSL (Domain-Specific Language) or its support in Groovy, learn from the following links:

- [DomainSpecificLanguage by Martin Fowler](http://martinfowler.com/bliki/DomainSpecificLanguage.html)
- [Domain-Specific Languages - Groovy Doc](http://docs.groovy-lang.org/docs/latest/html/documentation/core-domain-specific-languages.html)

Here is a core of our DSL implementation:

```groovy
def xml = new MyDependencyBuilder(out:System.out)
 
//print dependency - using DSL
xml.dependency {
    groupId "org.springframework"
    artifactId "spring-core"
    version "4.2.1.RELEASE"
}
```

The core class consist of one variable to output a result (`def out`) and a `methodMissing` implementation. Customizing the behavior of `methodMissing` (and `propertyMissing` in addition) is the heart of Groovy runtime metaprogramming, so we can create methods on the fly.

Every time you call any missing method, the runtime of Groovy routes the call to our `methodMissing`, just before throwing a `MissingMethodException` exception.

In a case of `Closure` params (line 5), we need to delegate all its calls back, to our `methodMissing` method using a change in Closure's [delegate property](http://docs.groovy-lang.org/docs/latest/html/documentation/core-domain-specific-languages.html#_delegating_to_an_arbitrary_type).

Here is an example of usage of this class: 
```groovy
def xml = new MyDependencyBuilder(out:System.out)
 
//print dependency - using DSL
xml.dependency {
    groupId "org.springframework"
    artifactId "spring-core"
    version "4.2.1.RELEASE"
}
```

All of the above should print something like this (without whitespaces): 

```xml
<dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-core</artifactId>
    <version>4.2.1.RELEASE</version>
</dependency>
```
