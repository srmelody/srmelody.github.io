---
layout: post
title:  "When Java and Groovy Collide"
date:   2017-06-29 12:25:00 -0400
categories: groovy grails debugging hacking
---

*Originally published 2015, Rally Engineering Blog*
## One Fine Day ...

One day, while we were working on stories to add some cool new features that we could ship to our customers, our operations team pinged us. Our analytics service, the Lookback API, was alerting on high CPU load. Lo and behold, we saw that there were a ton of very sad graphs. Oracle connections exhausted! Response times reaching multiple seconds! Five hundred series response codes! JVM heap completely allocated!
Gathering Forensics

So we started digging. Our first step whenever something like this happens is to gather forensics by taking thread dumps by sending a jstack command or the SIGQUIT signal via kill to the Java process. This gives us insight into what the JVM is doing.
Could it be Oracle?

Over the years, we've done tuning in Oracle and our connection pool to ensure we have the optimal connection config. We speculated that perhaps something was going on in Oracle. Several threads in our thread dumps looked like this:

````
<threadID>" prio=10 tid=0x0000000002137800 nid=0x2ccf waiting for monitor entry [0x00007fc3352cd000]
   java.lang.Thread.State: BLOCKED (on object monitor)
	at com.mchange.v2.resourcepool.BasicResourcePool.prelimCheckoutResource(BasicResourcePool.java:573)
   -  waiting to lock <0x0000000673d92218> (a com.mchange.v2.resourcepool.BasicResourcePool)
	at com.mchange.v2.resourcepool.BasicResourcePool.checkoutResource(BasicResourcePool.java:526)
	at com.mchange.v2.c3p0.impl.C3P0PooledConnectionPool.checkoutAndMarkConnectionInUse(C3P0PooledConnectionPool.java:755)
	at com.mchange.v2.c3p0.impl.C3P0PooledConnectionPool.checkoutPooledConnection(C3P0PooledConnectionPool.java:682)
	at com.mchange.v2.c3p0.impl.AbstractPoolBackedDataSource.getConnection(AbstractPoolBackedDataSource.java:140)
	at org.springframework.orm.hibernate3.LocalDataSourceConnectionProvider.getConnection(LocalDataSourceConnectionProvider.java:81)
````
We talked to the DBAs and dug in a little, and Oracle itself was fine. So the application was the culprit. Threads were waiting to acquire Oracle connections but request threads were taking too long and weren't releasing their connections to the pool.

We also saw several threads waiting in log4j.

````
java.lang.Thread.State: BLOCKED (on object monitor)
at org.apache.log4j.Category.callAppenders(Category.java:205)
- waiting to lock <0x00000006705e8c88> (a org.apache.log4j.Logger)
at org.apache.log4j.Category.forcedLog(Category.java:391)
at org.apache.log4j.Category.log(Category.java:856)
at org.slf4j.impl.GrailsLog4jLoggerAdapter.logMessage(GrailsLog4jLoggerAdapter.java:192)
````

Ugh, blocked in the log library?
````groovy
public void callAppenders(LoggingEvent event) {
  int writes = 0;
 
  for(Category c = this; c != null; c=c.parent) {
    // Protected against simultaneous call to addAppender, removeAppender,...
    synchronized(c) {
      if(c.aai != null) {
        writes += c.aai.appendLoopOnAppenders(event);
      }
      if(!c.additive) {
        break;
      }
    }
  }
 
  if(writes == 0) {
    repository.emitNoAppenderWarning(this);
  }
}
````

That smells bad and there's an ancient bug in the log4j bugzilla. You can solve this problem with a simple upgrade to log4j 2.0 which doesn't implement the callAppenders function in Category.Java and therefore doesn't synchronize on the code block that existed in log4j 1.2.

But it wasn't the only contention we saw.

## Wait, We See This All the Time!

In the thread dumps, we saw tons of these stack traces:

java.lang.Thread.State: BLOCKED (on object monitor)
at org.codehaus.groovy.runtime.MetaClassHelper.convertToTypeArray(MetaClassHelper.java:604)

Here's what was happening: when Groovy interacts with a Java object, it does so by using a metaclass object and if a metaclass for the Java class doesn't exist, it creates a new MetaClassImpl. Groovy uses metaclasses to add dynamic and functional features to Java (especially pre Java8 versions of Java). It's similar to dynamic object proxying or the way AspectJ weaves aspect code into bytecode. But when under load, the contention for the class lock causes threads to wait to obtain the lock to create new metaclasses. By default, Groovy will construct metaclasses as WeakReferences, which then turns those objects into candidates for garbage collection. But each of these metaclasses are the same! We don't change code or the metaclasses when we use our Java libraries (like log4j, spring, mongo driver) inside of the Grails app.

If we use strong metaclasses, we will end up making Groovy short circuit the code in ClassInfo.getMetaClassUnderLock (comments are mine).

````groovy
private MetaClass getMetaClassUnderLock() {
    MetaClass answer = getStrongMetaClass();
    if (answer!=null) return answer;
 
    // The weak metaclass will be garbage collected, even though the
    // metaclass is needed and will be re-constructed the next time
    // our Groovy code calls this Java code.
    answer = getWeakMetaClass();
 
    // Does a bunch of stuff in the case of Weak metaclasses,
    // re-creating the same object over and over each time this method
    // is called.
  }
````

## Solving the Problem

So, in Bootstrap.groovy's init function, we added:
````groovy
GroovySystem.setKeepJavaMetaClasses(true)
````

 

After changing this, we deployed our code to our test cluster. On our test cluster, we used JMeter to hammer our service with production-shaped traffic and use jstack, jmap, htop, the GC log, and host charts to monitor what was going on. We saw less lock contention at 10x our peak production load. Success!

## Extra Reading

Learn more about Groovy meta programming [here](http://www.groovy-lang.org/metaprogramming.html) and [here](/http://igor.kupczynski.info/2013/12/07/groovy-method-resolution.html).

*Author's note - this post originally appeared in the Rallydev.com Engineering blog, which has been lost in the internet.  But thanks to the [wayback machine](https://web.archive.org), it is back online!*
