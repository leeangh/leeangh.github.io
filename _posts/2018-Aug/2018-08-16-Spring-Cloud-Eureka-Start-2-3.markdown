---
layout: post
title:  "Crash Course - Spring Cloud Netflix Eureka - Part 2/3"
subtitle: "Eureka Client/Provider"
tags: [Spring Cloud, Netflix Eureka, Micro Service]
---

## Course Overview

Indeed, there are already many articles, blog posts of introducing the Spring Cloud Netflix Eureka. But I would like to share my own sample with more detailed descriptions on latest version of SpringCloud - Finchley.SR1(until 2018/8/15).

* **[Service Server (Service Registry)](/2018-08-15-Spring-Cloud-Eureka-Start-1-3/)**
    * Registry High Availability
* **[Service Client (Discovery Client/Service Provider)](/2018-08-16-Spring-Cloud-Eureka-Start-2-3/)**
* **[Service Consumer](/2018-08-17-Spring-Cloud-Eureka-Start-3-3/)**

In last post [Service Server (Service Registry)](/2018-08-15-Spring-Cloud-Eureka-Start-1-3/), we have setup two eureka servers to a HA cluster. Now we will create an Eureka Service and register it into the cluster.

## Setup Service Provider

### 1. Init Project

```bash
spring init --build=maven --java-version=1.8 --dependencies=cloud-eureka,cloud-eureka-server eureka-service.zip
```

### 2. Create and Update the application.yml
```yaml
spring:
  application:
    name: eureka-service

eureka:
  client:
    registerWithEureka: true
    fetchRegistry: true
    serviceUrl:
      defaultZone: http://eureka-peer1:8761/eureka/,http://eureka-peer2:8762/eureka/

  instance:
    hostname: eureka-service-p1
    instanceId: ${eureka.instance.hostname}:${server.port}
```

**Note!**
> In above yaml file, we configure both server's URLs for the defaultZone. Actually we can also put only one server's URL here, once we register our service it will be automatically registered into all other replicas servers. This is one of the benefits of what we want from the HA cluster.

### 3. Update Entrypoint Class

In the following class, we have enabled the EurekaClient, create a API of _$host/greeting/{name}_. In next course, we will call this API from an Eureka Consumer.

```java
package com.example.eurekaservice;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.EnableAutoConfiguration;
import org.springframework.cloud.netflix.eureka.EnableEurekaClient;
import org.springframework.context.annotation.Configuration;
import org.springframework.web.bind.annotation.PathVariable;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;

@Configuration
@EnableAutoConfiguration
@EnableEurekaClient
@RestController
public class DemoApplication {

	@RestController
	class ServiceInstanceRestController {

		@RequestMapping("/greeting/{name}")
		public String greeting(@PathVariable String name) {
			return "Hello " + name + "!";
		}
	}

	public static void main(String[] args) {
		SpringApplication.run(DemoApplication.class, args);
	}
}
```

### 4. Run the server

```bash
mvn -Dserver.port=10071 spring-boot:run
```

After application is started, open the web browser and open the url - **eureka-peer1:8761** and **eureka-peer2:8762**. You will find a new instance has been registered in both server.

>**eureka-peer1:8761**
![standalone-server1]({{site.baseurl}}/res/images/crash-course-eureka/peer1-server-service.png?raw=true)

>**eureka-peer2:8762**
![standalone-server2]({{site.baseurl}}/res/images/crash-course-eureka/peer2-server-service.png?raw=true)

> Next part **[Service Consumer](/2018-08-17-Spring-Cloud-Eureka-Start-3-3/)**