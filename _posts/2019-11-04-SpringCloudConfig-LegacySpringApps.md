---
title: Spring Cloud Config Server and Legacy Spring Apps
tags: 
- java
- Spring Boot
- Spring Cloud
- Hybris
categories: developer
desc: Integrating Spring Cloud Config Server with Legacy Spring Apps
layout: post
---

* TOC
{:toc}

Below steps explain, how to integrate Spring Config Client with a legacy Spring Web/MVC application. Same steps fundamentally can be extended to any Spring based application, may be with minor modifications and(or) additional changes.

For example:

Spring Config Client can even be integrated with packaged solutios as SAP-Hybris eComm platform.  The implementation for this is little more  complex, but can be done seamlessly and can be integrated with Hybris Platform Context’s HAC by overriding the default web/spring context all together as it is underneath is a java/spring based framerowrk. At the bottom of this blog, the section "Spring Cloud Config for Hybris" shows how below generic 6 steps can slightly be modfified (and with couple of more hybris specifc changes).

# Step-0: Dependencies

Add SpringCloudConfigClient jar (spring-cloud-config-client) as the dependency to your project (all other it’s dependencies will be pulled by itself if using maven/gradle. In case of Ant, you have to manually copy all jars to lib)

Note: In addition to the above you would need spring security outh2 client, if connecting to a OAUTH2 backed config server.

# Step-1 : Cloud Web Context Class

1. Override default contextClass  (which by default in case of MVC typically is
XmlWebApplicationContext) with your custom class, in the web.xml.  
2. This param typically is fed into Spring Dispatcher Servlet.
3. Below is a sample web.xml 
~~~xml
       <context-param>
          <description> Override default context with Spring Cloud Context for Config Client</description>
          <param-name>contextClass</param-name>
          <param-value>com.cardinalhealth.chh.storefront.config.SpringCloudConfigAppContext</param-value>
       </context-param>
~~~
4. Below is sample implementation of SpringCloudConfigAppContext class

~~~java

public class SpringCloudConfigContext extends XmlWebApplicationContext {
  @Override
  protected ConfigurableEnvironment createEnvironment() {
    System.out.println("##################### loaded my comfigurable context");
    return new CHAHCloudConfigEnvironment();
  }
}
~~~


# Step-2:  Override Property Locator

~~~java
int currentLevel = INITIAL_CHUNK_LEVEL;
List<Future<Chunk>> mergeFutureList = new ArrayList<>();
while (true) {
    //从队列中获取一组chunk
    List<Chunk> pollChunks = pollChunks(chunkQueue, currentLevel);
    //未取到同级chunk, 表示此级别应合并完成
    if (CollectionUtils.isEmpty(pollChunks)) {
        mergeFutureList.stream().map(this::get).forEach(chunkQueue::add);
        mergeFutureList.clear();
        //chunkQueue 中只有一个元素，表示此次合并是最终合并
        if (chunkQueue.size() == 1) {
            break;
        } else {
            currentLevel++;
            continue;
        }
    }
    Future<Chunk> chunk = threadPoolExecutor.submit(() -> merge(pollChunks, original));
    mergeFutureList.add(chunk);
}
~~~

# Step-3:  Config Server Authentication
# Step-4:  Environment properties
# Step-5:  All set
# Hybris specific changes

