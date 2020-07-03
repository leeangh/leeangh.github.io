---
layout: post
title: "[Demo] Spring Cloud Ribbon without Eureka"
subtitle: 'Ribbon without Eureka'
tags:
  - Spring Cloud
  - Ribbon
  - Micro Service
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

**But~~~** As described in Spring Cloud official site:
> Eureka is a convenient way to abstract the discovery of remote servers so that you do not have to hard code their URLs in clients. However, if you prefer not to use Eureka, Ribbon and Feign also work. Suppose you have declared a @RibbonClient for "stores", and Eureka is not in use (and not even on the classpath).

## How to Use Ribbon Without Eureka

First, we need explicitly disable the Eureka in Ribbon:
```yaml
ribbon:
    eureka:
      enabled: false
```

And we shall explicitly specify the location of the service servers, so the complete configuration(basic) shall be like:
```yaml
greeting-serive:
  ribbon:
    eureka:
      enabled: false
    listOfServers: eureka-service-p1:10073,eureka-service-p1:10072
```

If you want to know what is the greeting-service and the eureka-service-p1:1007x, please check my previous posts.

## Using Ribbon API directly

My example is based on the code segament from official documentation:
```java
@RestController
public class MyController {

	@Autowired
	private LoadBalancerClient loadBalancer;

	private RestTemplate restTemplate = new RestTemplate();

	@RequestMapping(value="/greeting/{name}", method = RequestMethod.GET)
	public String greeting(@PathVariable String name) {
		ServiceInstance instance = loadBalancer.choose("greeting-serive");
		URI storesUri = URI.create(String.format("http://%s:%s", instance.getHost(), instance.getPort()));

		return restTemplate.getForObject(storesUri+"/greeting/"+name, String.class);
	}
}
```

Now you can start this application and open url `http://localhost:10101/greeting/haha`, then the following messages will be shown if you keep refreshing the page:
```
Hello haha! I'm service from port : 10073
Hello haha! I'm service from port : 10072
Hello haha! I'm service from port : 10073
Hello haha! I'm service from port : 10072
....
...
```

Check out the [source](https://github.com/lee/spring-cloud-eureka-sample/tree/master/ribbon-app)