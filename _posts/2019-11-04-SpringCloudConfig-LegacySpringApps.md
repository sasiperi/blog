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
1. Override method createEnvironment() method of the default context class, to return your custom configurable environment. So that we can override the Locate Property Source. (the property location and population happens during “locate” in the default environment class (incase of Web Mvc, which would be StandardServletEnvironment)

~~~java
public class MyCloudConfigEnvironment extends StandardServletEnvironment
{
    @Override
    protected void customizePropertySources(MutablePropertySources propertySources)
    {
~~~

# Step-3:  Config Server Authentication
## 1. Open Source Spring Config Server (BASIC Auth)
Basic authorization is simple and is automatically handled (by default) by Congig Client itself either when spring cloud config user/password environment vars are configured. Or by simply configuring them in the URI itself. Actual names of the env vars is shown in the screen shot, under step-4.
## 2. PCF Config Server from market place (OAUTH2)
Unlike Open Source Spring Config server (which by default supports basic auth only), the one from PCF market place OAUTH2 (I think pcf works with server side decrypt only, for props that are encrypted, though)

Spring Config Client by default does not support OAUTH2 our of the box. So little more work need to be done in the above CHAHCloudConfigEnvironment, to manually create a RestTemplate and inject that into “ConfigServicePropertySourceLocator” so that it will use OAUTH2 headers instead of BASIC-AUTH headers.

## 3. Sample code for MyCloudConfigEnvironment.  
Below sample code does OAUTH2, as there is nothing needs to be done specially for BASIC
~~~java
public class MyCloudConfigEnvironment extends StandardServletEnvironment
{   
    public CHAHCloudConfigEnvironment()
    {
        super();
    }
    @Override
    protected void customizePropertySources(MutablePropertySources propertySources)
    {
        super.customizePropertySources(propertySources);
        try
        {
            PropertySource<?> source = initConfigServicePropertySourceLocator(this);
            propertySources.addLast(source);
        }
        catch (Exception ex)
        {
            logger.warn("failed to initialize cloud config environment", ex);
        }
    }
    
    private OAuth2ProtectedResourceDetails fullAccessresourceDetailsClientOnly(String accessTokenUri, String clientId, String clientSecret) 
    {
            ClientCredentialsResourceDetails resource = new ClientCredentialsResourceDetails();
            resource.setAccessTokenUri(accessTokenUri);
            resource.setClientSecret(clientSecret);
            resource.setClientId(clientId);
            resource.setGrantType("client_credentials");
            resource.setAuthenticationScheme(AuthenticationScheme.header);
            return resource;
    }
    private PropertySource<?> initConfigServicePropertySourceLocator(Environment environment)
    {
        ConfigClientProperties configClientProperties = new ConfigClientProperties(environment);
        System.out.println(environment.toString());
        configClientProperties.setUri(environment.getProperty("SPRING_CLOUD_CONFIG_URI", "http://localhost:8888"));
        System.out.println("##################### will load the client configuration");
        System.out.println(configClientProperties);
        ConfigServicePropertySourceLocator configServicePropertySourceLocator = new ConfigServicePropertySourceLocator(configClientProperties);
        OAuth2ProtectedResourceDetails resource = fullAccessresourceDetailsClientOnly(environment.getProperty("SPRING_CLOUD_CONFIG_TOKEN_URI"),environment.getProperty("SPRING_CLOUD_CONFIG_CLIENT_ID"),environment.getProperty("SPRING_CLOUD_CONFIG_CLIENT_SECRET"));
        OAuth2RestTemplate restTemplate = new OAuth2RestTemplate(resource);
        configServicePropertySourceLocator.setRestTemplate(restTemplate);
        return configServicePropertySourceLocator.locate(environment);
    }
}
 
~~~

# Step-4:  Environment properties
The following environment properties need to be configured, which will be pulled via Environment class, and will be overridden in the Config Client. (as shown in the below screen shot)

Note: For basic auth you can just make id/pwd part of URL (e.g. http://id:pwd@URI-path)

![eclipse-props]

# Step-5:  All set
# Hybris specific changes

