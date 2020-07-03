---
layout: post
title: "[Demo] Spring Cloud Feign"
subtitle: 'Declarative REST Client: Feign'
tags:
  - Spring Cloud
  - Feign
  - Micro Service
published: true
---
## What is Feign

[Feign](https://github.com/Netflix/feign) is a declarative web service client. It makes writing web service clients easier. To use Feign create an interface and annotate it. It has pluggable annotation support including Feign annotations and JAX-RS annotations. Feign also supports pluggable encoders and decoders. Spring Cloud adds support for Spring MVC annotations and for using the same `HttpMessageConverters` used by default in Spring Web. Spring Cloud integrates Ribbon and Eureka to provide a load balanced http client when using Feign.

> Reference : [Declarative REST Client: Feign](https://cloud.spring.io/spring-cloud-netflix/multi/multi_spring-cloud-feign.html)

## Create Feign Consumer Application

Create a new application - feign-consumer, pom.xml:
```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
	<modelVersion>4.0.0</modelVersion>

	<groupId>com.example</groupId>
	<artifactId>feign-consumer</artifactId>
	<version>0.0.1-SNAPSHOT</version>
	<packaging>jar</packaging>

	<name>feign-consumer</name>
	<description>Demo project for Spring Boot</description>

	<parent>
		<groupId>org.springframework.boot</groupId>
		<artifactId>spring-boot-starter-parent</artifactId>
		<version>2.0.4.RELEASE</version>
		<relativePath/> <!-- lookup parent from repository -->
	</parent>

	<properties>
		<project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
		<project.reporting.outputEncoding>UTF-8</project.reporting.outputEncoding>
		<java.version>1.8</java.version>
		<spring-cloud.version>Finchley.SR1</spring-cloud.version>
	</properties>

	<dependencies>
		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-web</artifactId>
		</dependency>
		<dependency>
			<groupId>org.springframework.cloud</groupId>
			<artifactId>spring-cloud-starter-openfeign</artifactId>
		</dependency>
		<dependency>
			<groupId>org.springframework.cloud</groupId>
			<artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
		</dependency>
	</dependencies>

	<dependencyManagement>
		<dependencies>
			<dependency>
				<groupId>org.springframework.cloud</groupId>
				<artifactId>spring-cloud-dependencies</artifactId>
				<version>${spring-cloud.version}</version>
				<type>pom</type>
				<scope>import</scope>
			</dependency>
		</dependencies>
	</dependencyManagement>

	<build>
		<plugins>
			<plugin>
				<groupId>org.springframework.boot</groupId>
				<artifactId>spring-boot-maven-plugin</artifactId>
			</plugin>
		</plugins>
	</build>


</project>
```

## Application Configuration
Application configuration in application.yml:
```yaml
spring:
  application:
    name: feign-consumer

server:
  port: 10201

eureka:
  client:
    registerWithEureka: false
    fetchRegistry: true
    service-url:
      defaultZone: http://eureka-peer1:8761/eureka/,http://eureka-peer2:8762/eureka/

feign:
  hystrix:
    enabled: true
```

## Setup

We will have three java files in this demo project.
```bash
FeignConsumerApplication.java
GreetingServiceHystrix.java
GreetingService.java
```

### Include Feign in Application

To include and enable feign client in `FeignConsumerApplication.java`:

```java
@SpringBootApplication
@EnableFeignClients
public class FeignConsumerApplication {

    public static void main(String[] args) {
        SpringApplication.run(FeignConsumerApplication.class, args);
    }

    @RestController
    public class FeignConsumerController {
        @Autowired
        GreetingService greetingService;

        @RequestMapping(value="/greeting", method = RequestMethod.GET)
        public String greeting() {
            return greetingService.greeting();
        }

        @RequestMapping(value="/greeting/{name}", method = RequestMethod.GET)
        public String greeting(@PathVariable String name) {
            return greetingService.greeting(name);
        }
    }
}
```

### Binding Services

Create interface for accessing the Eureka-Service which we have setup in [Crash Course - Spring Cloud Netflix Eureka - Part 2/3](https://leeangh.github.io/2018-08-16-Spring-Cloud-Eureka-Start-2-3/).

`GreetingService.java`

```java
@FeignClient(name = "eureka-service", fallback = GreetingServiceHystrix.class)
public interface GreetingService {

    @RequestMapping(value = "/greeting", headers = "accept=text/plain")
    String greeting();

    @RequestMapping(value = "/greeting/{name}", headers = "accept=text/plain")
    String greeting(@PathVariable String name);
}
```

**!!!Note!!!**

I have faced the issue of `feign.FeignException: status 406 `. Finally I found I have to setting the headers for accessing the service url:
`headers = "accept=text/plain"`


### Feign Hystrix Fallbacks

Looking at the `@FeignClient(name = "eureka-service", fallback = GreetingServiceHystrix.class)`, we enable the fallback handling for @FeignClient.

> Hystrix supports the notion of a fallback: a default code path that is executed when they circuit is open or there is an error. To enable fallbacks for a given @FeignClient set the fallback attribute to the class name that implements the fallback.

`GreetingServiceHystrix.java`

```java
@Component
public class GreetingServiceHystrix implements GreetingService {

    @Override
    public String greeting() {
        return "Eureka-service is not available";
    }

    @Override
    public String greeting(String name) {
        return greeting();
    }
}
```

**Note! To enable Feign Hystrix Support you need to add following content into `application.yml`**

```yaml
feign:
  hystrix:
    enabled: true
```

## Test Feign Consumer
`http://localhost:10201/greeting/lee`

### When Service is available

Results when service is available:
> Hello lee! I'm service from port : 10072

### When Service is offline

We shutdown the service instances, we expect the fallback handling shall works:
> Eureka-service is not available

Checkout the [Source](https://github.com/leeangh/spring-cloud-eureka-sample/tree/master/feign-consumer)