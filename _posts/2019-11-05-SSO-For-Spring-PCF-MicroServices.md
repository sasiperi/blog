---
title: SSO Integration - Microservices in PCF
tags: 
- java
- Spring Boot
- Spring Cloud
- Pivotal
categories: developer
desc: Integrating with SSO/IDP for PCF hooused microservices
excerpt: In this blog, we will see how to seamlessly integrate applications deployed in PCF with your SSO/IDP. There are 2 parts to this.We will in this article cover both a) Platform level configuration changes b) Application level changes
layout: post

---

* TOC
{:toc}

In this blog, we will see how to seamlessly integrate applications deployed in PCF with your SSO/IDP. There are 2 parts to this.
1. PCF platform level configuration (SSO Plan creation, setting up federation partnership)
2. Application level changes - to consume the claims (returned by your IDP) presented in OAUTH Token by PCF. 

