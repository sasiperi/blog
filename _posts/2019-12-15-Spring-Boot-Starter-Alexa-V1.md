---
title: Spring Boot Starter for Alexa (Alexa v1 SDK)
tags: 
- java
- Spring Boot
- Spring Cloud
- Alexa
- AWS
categories: developer
desc: Spring Boot Starter for Alexa
excerpt: Spring Boot Starter Alexa (v1) is licenced under Apache V2. This guide walks you through the process of building an application using the starter lib.
layout: post
---

* TOC
{:toc}

# Spring Boot Starter for Alexa (Alexa v1 SDK)

Spring-boot-starter-alexa is an open source project, licenced under Apache V2. This is a spring boot starter, that helps to host a Custom Skill as a Web Service, using SpringBoot, which 
* Auto configures Speechlet, abstracts all the boilerplate code that is needed Alexa Skill Kit.
* Provides default implementation for generic intents, that would occur during the life cycle of the custom intents (start session, wakeup words, ending sessions and Alexa Build in Intents such as welcome/hello). This all can be managed by configuring proper responses in the application.propeties
#### *Alexa SDK Note*
**This starter is compatable (and is built using) Amazon Alexa SK V1. Spring Boot Starter for V2 (ASK SDK) is underway soon to be relased. Watch out this space !!**

This blog is a guide that walks you through the process of building an application that uses Spring Boot Starter Alexa, to build custom skill as a web service.

## Getting Started

These instructions will get you a copy of the project up and running on your local machine for development and testing purposes. See deployment for notes on how to deploy the project on a live system.

### What you will build
You will build simple Hello World custom skill, as a webservice (endpoint /helloworld)

### Prerequisites  (what you will need)

* About 15 minutes to build skill, using starter project
* Abput 15 minutes to test.
** About 10 minutes to configure your skill on Alexa (you will need account to access Alexa Developer Console
** About 5 minutes to test using Alexa Voice/Echo simulator
* A favorite text editor or IDE
* JDK 1.8 or later
* Gradle 4+ or Maven 3.2+
* You can also import the sample code (under samples/alexa-helloworld-springboot-starter-pcf, which is created as a maven project) straight into your IDE as maven project:
** Eclipse/Spring Tool Suite (STS)
** IntelliJ IDEA

```
Give examples
```

### How to use boot starter and create skill? 

A step by step series of examples that tell you how to get a development env running

#### Add maven (or gradel) dependency
Crate a spring boot starter project using spring boot initializer available in IDE (rest) or from start.spring.io.

#### Add maven dependency
* Open pom.xl (or gradle) and spring-boot-starter-alexa dependency.
* Current release version is 1.0.

[Sample Maven Code](https://github.com/sasiperi/alexa-spring-boot/blob/86300097178b1a57f77aa05d19451fe098211a70/samples/alexa-helloworld-springboot-starter-pcf/pom.xml#L28-L32)

~~~xml
    <dependency>
			<groupId>io.github.sasiperi</groupId>
			<artifactId>alexa-spring-boot-starter</artifactId>
			<version>1.0.3</version>		
		</dependency>
~~~

#### Configure application properties. 
* You can in your YAML or .properties file use tab for hints of all available properties and the default values provided.
* You can override these in your application's app.props (or YAML)
* All the available properties start with **alexa**
* Check the additional properties meta data for details of what each property means, what are the allowed values.
below are the availableample properties

~~~.prperties

###The application id that alexa(dev) provides amzn1.ask.skill.xxxxxxx###
alexa.application-id=amzn1.ask.skill.481fb850-g95a-5345-9h29-14fbbc889944

###### card title that you want to go on the account alexa.amazon and in the appstore ####
alexa.card-title=alexa-hello

#####Comma sepratated list of speechlet URI mappings, which will be invoked for intent(s) ############
alexa.speechlet-uri-mappings=/alexaHello

############## Various responses, for generic actions and intents ###################
alexa.response.good-bye= Good Bye Sample Spring Boot Hello 
alexa.response.hello-intent=Hello Sample Spring Boot Hello
alexa.response.help-intent=Help Sample Spring Boot Hello
alexa.response.welcome=Welcome Sample Spring Boot Hello
~~~

End with an example of getting some data out of the system or using it for a little demo

#### Creating Custom Skill Speechlet
Starter will auto cinfigure ASK SDK for you. You need to override onIntent() method of the default Specchlet implementation.
Example snippet

[Snippet from HelloWorld Sample Project](https://github.com/sasiperi/alexa-spring-boot/blob/2c0a3eea40fe27da5d3b12c9dde126c5a968bb1d/samples/alexa-helloworld-springboot-starter-pcf/src/main/java/io/github/sasiperi/alexa/spring/boot/examples/service/HelloWorldSpeechlet.java#L47-L75)

**Note:

  * AlexaProperties is a comvenient class (bean) injected, to access any props starting with "alexa."
  * AlexaProperties bean is injected by starter and can be autowired to detect all/any props starting with "alexa." in your skill project
      - either to override default properties from starter 
      - or you want your own properties.
      - **For example* 
          - If you created a brand new prop called alexa.myProp then @alexaProperties.getMyprop() will give you this value 
	      - OR if you have alexa.hello-intent  @alexaProperties.getHelloIntent() will give inject the overriden value into your skill app.

##### Authentication, Autherization and Account Linking
  * To enable authentication, autherization, you may need to override other methods in the default impl.
  * An example project to demonstrate Auth using OAUTH2 (using auth-code grant) and does the account linking, is under samples/alexa-oms (sample order tracking skill for authorized customer).

~~~java
@Service
public class HelloWorldSpeechlet extends SkillSpeechletDefaultImpl
{
    private static final Logger log = LoggerFactory.getLogger(HelloWorldSpeechlet.class);


    @Autowired
    private AlexaProperties alexaProps;


    @Override
    public SpeechletResponse onIntent(final IntentRequest request, final Session session) throws SpeechletException
    {
        log.info("onIntent requestId={}, sessionId={}", request.getRequestId(), session.getSessionId());


        Intent intent = request.getIntent();
        String intentName = (intent != null) ? intent.getName() : null;


        if ("HelloWorldIntent".equals(intentName))
        {
            return getResponse(alexaProps.getResponse().getHelloIntent());
        } else if ("AMAZON.HelpIntent".equals(intentName))
        {
            return getResponse(alexaProps.getResponse().getWelcome());
        } else
        {
            throw new SpeechletException("Invalid Intent");
        }
    }


}

~~~


## Running the tests

* Get an account to Amazon Alexa developer console.
* Add and configure your skill.
  - the endpoint requires a public hhtp/https url, which can inherit certs from the main domain e.g. hosted on PWS, can inherit from PWS
  - OR for local testing, Amazon susggests NGROK that can expose a HTTP/HTTPS urls, that can be used to configure endpoints, which would rout the request to an application running on your localhost:port.

* Click on the skill and create your speech assets.
* Sample hello world sppech assets for this sample application can be found here, that can be copy pasted.
: [Hello World Speech Assets](https://github.com/sasiperi/alexa-spring-boot/tree/master/samples/alexa-helloworld-springboot-starter-pcf/src/main/resources/speechAssets)

* Go to Alexa Skill Simulator, activate your skill, using the wake up word.
* You can now test default intents and HelloWorld custom intent as shown in the below screen shot.
**You can see Alexa responses are created based on the configuration, as shown in the below Screenshot.

![Alexa Simulator Test](https://sasiperi.github.io/blog//static/img/alexa-helloworld-springboot-starter.png)


### Break down into end to end tests


## Deployment


## Built and Released with

* [Maven](https://maven.apache.org/) - Dependency Management
* [SonaType](https://oss.sonatype.org/)
* [Nexus Repo](https://rometools.github.io/rome/) - Artifacts Repo
* [Maven Central](https://repo.maven.apache.org/maven2/io/github/sasiperi/alexa-spring-boot-starter/)
* ngrok [https://ngrok.com/]  really rocks, and lets you test skills locally, on local host.

## Contributing

Please read [CONTRIBUTING.md](https://gist.github.com/sasiperi/spring-boot-starter-alexa/CONTRIBUTING.md) for details on our code of conduct, and the process for submitting pull requests to us.

## Versioning

I use [SemVer](http://semver.org/) for versioning. For the versions available, see the [tags on this repository](https://github.com/your/project/tags). 

## Authors

* **Sasi Peri** - *Initial work* - [sasiperi](https://github.com/sasiperi)
* **Home Page and Blog** [*Homepage & Blog*](sasiperi.github.io)


## License

This project is licensed under the Apache V2 License - see the [LICENSE](https://github.com/sasiperi/alexa-spring-boot/blob/master/LICENSE) file for details

## Acknowledgments
### Inspiration
1. [David Conway](https://www.linkedin.com/in/david-conway-31681513/?lipi=urn%3Ali%3Apage%3Ad_flagship3_search_srp_top%3BYB6kGZP%2FQdSP%2Fzr%2B2vFEzw%3D%3D&licu=urn%3Ali%3Acontrol%3Ad_flagship3_search_srp_top-search_srp_result&lici=FXLx3kG0REi70kl3UZAElw%3D%3D), Managing Director, Morgan Stanely
2. [Jim Shingler](https://github.com/jshingler), Director, Enterprise Arch, Cardinal Health





