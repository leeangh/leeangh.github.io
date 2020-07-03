---
layout: post
title:  "Crash Course - Spring Cloud Netflix Eureka - Part 3/3"
subtitle: "Eureka Consumer"
tags: [Spring Cloud, Netflix Eureka, Micro Service]
---

## Course Overview

Indeed, there are already many articles, blog posts of introducing the Spring Cloud Netflix Eureka. But I would like to share my own sample with more detailed descriptions on latest version of SpringCloud - Finchley.SR1(until 2018/8/15).

* **[Service Server (Service Registry)](/2018-08-15-Spring-Cloud-Eureka-Start-1-3/)**
    * Registry High Availability
* **[Service Client (Discovery Client/Service Provider)](/2018-08-16-Spring-Cloud-Eureka-Start-2-3/)**
* **[Service Consumer](/2018-08-17-Spring-Cloud-Eureka-Start-3-3/)**

In last post [Service Server (Service Registry)](/2018-08-15-Spring-Cloud-Eureka-Start-1-3/), we have setup two eureka servers as a HA cluster with one service provider registered. Now I will create a consumer application to verify calling the service via the eureka client.

## Setup Service Consumer

### 1. Init Project

```bash
spring init --build=maven --java-version=1.8 --dependencies=cloud-eureka,web web-app.zip
```

### 2. Create and Update the application.yml
```yaml
spring:
  application:
    name: web-app

server:
  port: 8080
  
eureka:
  client:
    registerWithEureka: false
    fetchRegistry: true
    serviceUrl:
      defaultZone: http://eureka-peer1:8761/eureka/,http://eureka-peer2:8762/eureka/

  instance:
    hostname: eureka-web
    instanceId: web-app:${server.port}
```

### 3. Call service from Provider

```java
@SpringBootApplication
@Configuration
@EnableAutoConfiguration
public class DemoApplication {

	@Autowired
	RestTemplate restTemplate;

	private @RestController class Controller {

		@RequestMapping("/sayHelo/{name}")
		public String greeting(@PathVariable String name) {
			return restTemplate.getForObject("http://eureka-service/greeting/{name}", String.class, name);
		}

		@Bean
		@LoadBalanced
		public RestTemplate restTemplate() {
			return new RestTemplate();
		}
	}

	public static void main(String[] args) {
		SpringApplication.run(DemoApplication.class, args);
	}
}
```

Looking at the statement: 
>restTemplate.getForObject("http://eureka-service/greeting/{name}", String.class, name)

The URI of the server - **http://eureka-service/greeting/{name}**, we use the service instance name instead of the hostname or ip address.

### 4. Start the Consumer Application
```bash
mvn spring-boot:run
```

Open the web browser, test the web URL - http://eureka-web:8080/sayHelo/Tracy.

![Access Service](https://raw.githubusercontent.com/leeangh/leeangh.github.io/master/res/images/crash-course-eureka/access-service-from-web-client.png)

### 5. High Availability - How?

I have mentioned the HA cluster at the first part, now I will show you a simple way to verify what HA means for the Eureka.

* Now we stop the server peer1
  
  You can see the peer1 is <font color='red'>DOWN(1)</font> in the peer2's status page:

  ![peer2-with-closed-peer1](https://raw.githubusercontent.com/leeangh/leeangh.github.io/master/res/images/crash-course-eureka/peer2-with-closed-peer1.png)

* Open the web browser, test againg on - http://eureka-web:8080/sayHelo/Tracy
  
  ![Access Service](https://raw.githubusercontent.com/leeangh/leeangh.github.io/master/res/images/crash-course-eureka/access-service-from-web-client.png)

This is why we call it 'High Availability', even only one single server is up then the whole system is still available. 

We can see that I have stoped the server peer1, but the consumer is still able to call the service because both servers have the registration of the service providers.

OK, this is the end of this Crash Course. There might some mistakes through these three posts becuase I'm also fresh to the Spring Cloud. But I hope it can help you to get a overview, and welcome your comments.

Checkout the source code of the demo [projects](https://github.com/leeangh/spring-cloud-eureka-sample).
