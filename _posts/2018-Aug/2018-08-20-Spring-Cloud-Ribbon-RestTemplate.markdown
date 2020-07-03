---
layout: post
title: "[Demo] Spring Cloud Ribbon + RestTemplate"
subtitle: 'Ribbon + RestTemplate'
tags:
  - Spring Cloud
  - Ribbon
  - RestTemplate
published: true
---
## What is Ribbon

Ribbon is a client-side load balancer that gives you a lot of control over the behavior of HTTP and TCP clients.
Feign already uses Ribbon, so, if you use `@FeignClient`, this section also applies.

A central concept in Ribbon is that of the named client.
Each load balancer is part of an ensemble of components that work together to contact a remote server on demand, and the ensemble has a name that you give it as an application developer (for example, by using the `@FeignClient` annotation).
On demand, Spring Cloud creates a new ensemble as an `ApplicationContext` for each named client by using
`RibbonClientConfiguration`.
This contains (amongst other things) an `ILoadBalancer`, a `RestClient`, and a `ServerListFilter`.

> Reference : [Client Side Load Balancer: Ribbon](https://cloud.spring.io/spring-cloud-netflix/multi/multi_spring-cloud-ribbon.html)

## RestTemplate

RestTemplate class is Spring's central class for synchronous client-side HTTP access.

Now let's have a look at the the main code of the **[Crash Course - Spring Cloud Netflix Eureka - Part 3/3 ~~ Eureka Consumer](/2018-08-17-Spring-Cloud-Eureka-Start-3-3/)** again:

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
Notice for the following two items which are used the in above code block.

* ```RestTemplate restTemplate;```
* ```@LoadBalanced```

### RestTemplate

If we make a little modification, remove the annotation `@LoadBalanced`.
```java
@Bean
@LoadBalanced
public RestTemplate restTemplate() {
        return new RestTemplate();
}
```
Try to access the url http://eureka-web:8081/sayHelo/Tracy
```
Whitelabel Error Page
This application has no explicit mapping for /error, so you are seeing this as a fallback.

Tue Aug 21 14:26:02 CST 2018
There was an unexpected error (type=Internal Server Error, status=500).
I/O error on GET request for "http://eureka-service/greeting/Tracy": eureka-service; nested exception is java.net.UnknownHostException: eureka-service
```

Does it means the `RestTemplate` must be used together with the `@LoadBalanced`?
> Of course, **NO!**

`RestTemplate` is a pure client side library for http accessing, to use it without eureka you just use the real URL - ***http://eureka-service-p1:8071/greeting/{name}***

### @LoadBalanced

The annotation `@LoadBalanced` is used to mark a RestTemplate bean to be configured to use a RibbonLoadBalancerClient. It implicitly include the Ribbon to locate & indicate with the eureka services.

## Multiple Services for Load Balance

We start three instances of the same service at three different ports, look at the [source code](https://github.com/leeangh/spring-cloud-eureka-sample/tree/master/eureka-service-multiple).

![2-servers-3-services]({{site.baseurl}}/res/images/crash-course-eureka/2_server_3_services.png)

New service API will print a simple message, indicates which service instace is accessed:
```java
public class DemoApplication {

        @RestController
        class ServiceInstanceRestController {

                @Value("${server.port}")
                String port;

                @RequestMapping("/greeting/{name}")
                public String greeting(@PathVariable String name) {
                        return "Hello " + name + "! I'm service from port : " + port;
                }
        }

        public static void main(String[] args) {
                SpringApplication.run(DemoApplication.class, args);
        }
}
```

## Start the consumer app

Now, what we expect if we access our consumer URL - ***http://eureka-web:8081/sayHelo/Tracy***???
Our access will be balanced to three service instances:
```json
[
   {
      "0": "Hello Tracy! I'm service from port : 10071"
   },
   {
      "0": "Hello Tracy! I'm service from port : 10073"
   },
   {
      "0": "Hello Tracy! I'm service from port : 10072"
   },
   {
      "0": "Hello Tracy! I'm service from port : 10071"
   }
   //.....
]
```

Well, this is not the end. If you have read the official document of : [Reference - Client Side Load Balancer: Ribbon](https://cloud.spring.io/spring-cloud-netflix/multi/multi_spring-cloud-ribbon.html), you may know:
* Ribbon can be used with eureka -- As I used in this post.
* Ribbon can be used without eureka.

Another thing that I would like to highlight about the [different of @RibbonClient and @LoadBalanced](https://stackoverflow.com/questions/39587317/difference-between-ribbonclient-and-loadbalanced).

In next post, I would try to introduce you how to use Ribbon without eureka.