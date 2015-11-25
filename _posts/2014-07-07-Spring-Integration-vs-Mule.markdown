---
layout: post
title: Spring Integration vs. Mule
date:   2014-07-07 14:14:21
---
Back in November of 2013, I performed an initial comparison of Spring Integration and Mule and found many advantages of using SI when compared to Mule.  At the time, I had a lot of Mule experience but very little SI experience.  The advantages of SI became immediately apparent, with the major advantages coming in the areas of ease of development and automated unit and functional testing.  I revisited my comparison in July of 2014 to see if this was still the case.  With a few minor modifications, the comparison was for the most part the same result; SI in a landslide!

## Feature Comparison

### Deploying
  * Mule: All projects deploy to a running Mule Server instance
  * SI: No restrictions when deploying, i.e. you can build a Spring Boot executable jar and run anywhere

_Key Difference:_
Using gradle and spring boot, you can create a lightweight executable jar file that runs a standalone java application that creates a simple spring context.  This gives a great amount of flexibility for testing and deploying flows.

### Spring Version
  * Mule: Tied to Mule server's version of spring
  * SI: No restrictions

_Key Difference:_
With SI, we can use any version of Spring we need, since you have full control over the spring environment.  Mule ships tied to a specific version of spring and since Mule runs on this version, it makes it impossible to deviate from it (or if not impossible, impractical).

### IDE
  * Mule: Mule Studio, currently based on Eclipse version 3.8
  * SI: STS, currently based on Eclipse version 4.3

_Key Difference:_
Spring's STS IDE seems to be on a much more aggressive development cycle than Mule Studio.  A big advantage in tooling considering all the bugs that were in Mule Studio.  Mule's IDE had more functionality, especially on the graphical component side in designing flows, but I didn't find that to be enough of an advantage given all the classpath issues, as well as the freezing issues which at times made Studio unuseable.

### Creating Flows
  * Mule: Mule Studio, using the GUI or pure XML
  * SI: Use pure XML, Java config, or the new Java DSL

_Key Difference:_
The GUI piece of Mule's Studio is more robust when creating flows, but it is also very buggy, auto-editing certain components at times.  Creating flows using XML in SI was very easy and even though you couldn't use the GUI to create the flow, you could use it to view the flow once created.  The Java config in SI allows better typing for defending against refactoring risk than using XML in SI or either of Mule's methods of flow creation. The Java DSL is great for strongly typing your flow components but it is rather clunky and I shied away from using it.

### Client Communication with a Flow Endpoint
  * Mule: JMS client, HTTP client, etc... I've implemented the Observer pattern to decouple these details from busness logic
  * SI: Interface-driven using any POJO interface.  All implementation details are abstracted away into the config (which can be environment specific, using spring profiles)

_Key Difference:_
What makes this interesting is the details of how a message enters a flow are completely abstracted away, when using an SI Gateway config.  This means a flow could run in-process, or use JMS or HTTP or any other protocol to send the message to an external system.  All of the details are handled through the configuration, so if you decide to change your transport mechanism (or run a flow in-process in a development environment), no code changes in your client application.  You are just invoking a method on a POJO interface.  We've achieved a similar strategy using the Observer pattern when sending messages in our current code in order to abstract some of the protocol-specific details away from our business logic, but this has much less flexibility then Spring Integration. 

### Clustering Flows
  * Mule: None with the community edition
  * SI: Yes, but seems to be in its infancy. Also, there exists a Hazelcast plugin which would give you clustering. Not sure how mature it is at this point.

_Key Difference:_
Hazelcast, an in-memory data grid, has developed a plugin for Spring Integration.
http://www.hazelcast.com/docs/2.1/manual/multi_html/ch14.html 

https://github.com/garyrussell/spring-integration-ha-demo

### Monitoring
  * Mule: MMC, with the paid version. JMX – Seems to be some basic stats.  Seems like you need to create a lot of custom Mbeans to make it useful
  * SI: JMX monitoring for advanced stats of performance of messages in flows, including easy custom configurations for exporting data to JMX.

_Key Difference:_
SI Mbeans expose a bunch of information about endpoints, message channels, and messages.  It also supports lifecycle operations on endpoints to start, stop, etc...  which means we would have full control over temporarily stopping flows and such in production.  Has a bunch of statistics for detecting problems, error channel stats, time spent in processing.  Also has a notification panel where you can publish custom application notifications to an MBean, but I have yet to get that to work.

Also in SI, you can create managed resources for your custom beans using annotations.  Any custom component you create can be exposed as an Mbean.

Samples for monitoring
https://github.com/spring-projects/spring-integration-samples/tree/master/intermediate/monitoring

### Stack Overflow Search Hits
  * Mule
    * JMS: 2640
    * Email: 3810
    * Transformer: 1320
    * Multithread: 3470
  * SI 
    * JMS: 2350
    * Email: 4860
    * Transformer: 1580
    * Multithread: 2000

_Key Difference:_
Ran some cursory searches on Stack Overflow just to get an idea of the community behind both tools.  Numbers are fairly similar so I concluded there are active communities behind both tools.  Larger numbers aren't necessarily better, since that could mean buggier code as opposed to simply meaning a larger user base.

### Unit testing
  * Mule: Functional tests that need to run inside a pared-down Mule container.  Can't mock out everything, only spring beans
  * SI: Similar conceptually to the Mule testing, mocking out pieces of flows.  Seems a bit easier since it's pure Spring beans to mock.  And absolutely everything is a spring bean so mocking is more robust.

_Key Difference:_
Mule Functional tests involved running a heavyweight mule server in-process which made tests bulky and slow.  The SI functional tests were far more lightweight and also much easier to develop in a timely fashion.

### Components
  * Mule: Uses some custom component names, but pretty intuitive and loosely based on Enterprise Integration Patterns book
  * SI: Modeled after Enterprise Integration Patterns book.  Names of components follow the patterns

_Key Difference:_
None worth mentioning.

### Other SI links used for research
Enterprise Integration Patterns: http://www.eaipatterns.com/ 
http://projects.spring.io/spring-integration/ 

Docs – were equally as maddening as the Mule docs
http://docs.spring.io/spring-integration/docs/4.0.0.BUILD-SNAPSHOT/reference/html/ 

Official Code Samples link on github
https://github.com/spring-projects/spring-integration-samples 
