---
layout: post
title:  "Crash Course - Spring Cloud Netflix Eureka - Part 1/3"
subtitle: "Overview, Eureka Server"
tags: [Spring Cloud, Netflix Eureka, Micro Service]
---

> This project provides Netflix OSS integrations for Spring Boot apps through autoconfiguration
and binding to the Spring Environment and other Spring programming model idioms. With a few
simple annotations you can quickly enable and configure the common patterns inside your
application and build large distributed systems with battle-tested Netflix components. The
patterns provided include Service Discovery (Eureka), Circuit Breaker (Hystrix),
Intelligent Routing (Zuul) and Client Side Load Balancing (Ribbon).


> ### Features
> * Service Discovery: Eureka instances can be registered and clients can discover the instances using Spring-managed beans
> * Service Discovery: an embedded Eureka server can be created with declarative Java configuration
> * Circuit Breaker: Hystrix clients can be built with a simple annotation-driven method decorator
> * Circuit Breaker: embedded Hystrix dashboard with declarative Java configuration
> * Client Side Load Balancer: Ribbon
> * External Configuration: a bridge from the Spring Environment to Archaius (enables native configuration of Netflix components using Spring Boot conventions)
> * Router and Filter: automatic registration of Zuul filters, and a simple convention over configuration approach to reverse proxy creation

> [Check official description](https://cloud.spring.io/spring-cloud-netflix/)

## Course Overview

Indeed, there are already many articles, blog posts of introducing the Spring Cloud Netflix Eureka. But I would like to share my own sample with more detailed descriptions on latest version of SpringCloud - Finchley.SR1(until 2018/8/15).

* **[Service Server (Service Registry)](/2018-08-15-Spring-Cloud-Eureka-Start-1-3/)**
    * Registry High Availability
* **[Service Client (Discovery Client/Service Provider)](/2018-08-16-Spring-Cloud-Eureka-Start-2-3/)**
* **[Service Consumer](/2018-08-17-Spring-Cloud-Eureka-Start-3-3/)**



### More Prerequisites

#### 1. Modify /etc/hosts
Add the following line:
> 127.0.0.1       eureka-peer1 eureka-peer2 eureka-service-p1 eureka-web

#### 2. My Development Environment
```
OpenJDK 64-Bit Server VM (build 25.172-b11, mixed mode)
Apache Maven 3.5.2
Linux-4.14.60-1-Manjaro-amd64
Spring CLI v2.0.4.RELEASE
IntelliJ IDEA Community Edition
```

## Setup Service Server

### 1. Init Project

```bash
spring init --build=maven --java-version=1.8 --dependencies=cloud-eureka-server eureka-server.zip
```

### 2. Create and Update the application.yml

```yaml
spring:
  application:
    name: eureka-server
server:
  port: 8761
eureka:
  client:
    registerWithEureka: false
    fetchRegistry: false
    serviceUrl:
      defaultZone: http://eureka-peer1:8761/eureka/
  instance:
    preferIpAddress: false
    hostname: eureka-peer1
    instanceId: ${eureka.instance.hostname}:${server.port}
```

### 3. Update Entrypoint Class

```java
package com.example.eurekaserver;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.cloud.netflix.eureka.server.EnableEurekaServer;

@EnableEurekaServer
@SpringBootApplication
public class DemoApplication {

	public static void main(String[] args) {
		SpringApplication.run(DemoApplication.class, args);
	}
}
```

### 4. Run the server

```bash
mvn spring-boot:run
```

After application started, open the web browser and open the url - *eureka-peer1:8761*.

![standalone-server]({{site.baseurl}}/res/images/crash-course-eureka/standalone-server.png?raw=true)

### 5. High Availability

After above steps we have successfully setup a Eureka Server, but now it is a standalone server. The risk is that once the this server is down, the client can't access any services. The a HA Cluster is required in this case, let's have a look at the Eureka Architecture:
![Eureka Architecture]({{site.baseurl}}/res/images/crash-course-eureka/eureka_architecture.png?raw=true)

As described in Eureka Wiki site:

> The architecture above depicts how Eureka is deployed at Netflix and this is how you would typically run it. There is one eureka cluster per region which knows only about instances in its region. There is at the least one eureka server per zone to handle zone failures.
> 
> Services register with Eureka and then send heartbeats to renew their leases every 30 seconds. If the client cannot renew the lease for a few times, it is taken out of the server registry in about 90 seconds. The registration information and the renewals are replicated to all the eureka nodes in the cluster. The clients from any zone can look up the registry information (happens every 30 seconds) to locate their services (which could be in any zone) and make remote calls.

### 6. Setup Server Replicas

We don't need to create a fresh project, just made some modifications on the application.yml.

```yaml
spring:
  profiles:
    active: s1
---
spring:
  profiles: s1
  application:
    name: eureka-server
server:
  port: 8761
eureka:
  client:
    registerWithEureka: true
    fetchRegistry: true
    serviceUrl:
      defaultZone: http://eureka-peer2:8762/eureka/
  instance:
    preferIpAddress: false
    hostname: eureka-peer1
    instanceId: ${eureka.instance.hostname}:${server.port}
---
spring:
  profiles: s2
  application:
    name: eureka-server
server:
  port: 8762
eureka:
  client:
    registerWithEureka: true
    fetchRegistry: true
    serviceUrl:
      defaultZone: http://eureka-peer1:8761/eureka/
  instance:
    preferIpAddress: false
    hostname: eureka-peer2
    instanceId: ${eureka.instance.hostname}:${server.port}
```

In above yaml file, we define two application profiles for two eureka replicas(servers). Please look carefully  of the port, defaultZone and hostname for each server. Benefit for this setup will be described in next posts.

{% raw %}
<div class="mermaid">
graph LR
    p1(fa:fa-registered Eureka Server - Peer1)
    p2(fa:fa-registered Eureka Server - Peer2)

    p1-->|Replicate|p2
    p2-->|Replicate|p1
</div>
{% endraw %}

#### 6.1 Start two Eureka Servers
```bash
mvn -Dspring.profiles.active=s1 spring-boot:run
```
```bash
mvn -Dspring.profiles.active=s2 spring-boot:run
```

#### 6.1 Eureka Server Cluster Demo
![peer1_only_server]({{site.baseurl}}/res/images/crash-course-eureka/peer1_only_server.png?raw=true)
![peer2_only_server]({{site.baseurl}}/res/images/crash-course-eureka/peer2_only_server.png?raw=true)

From above two pictures, you can see that each server now is a relicas of the another server.

> Next part **[Service Client (Discovery Client/Service Provider)](/2018-08-16-Spring-Cloud-Eureka-Start-2-3/)**