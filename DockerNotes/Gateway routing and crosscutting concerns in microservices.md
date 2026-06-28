It is called a cross-cutting concern because if you look at a system's architecture as a matrix, these features literally cut across the vertical slices of your business logic.  
**The Visual Metaphor**  

- **Vertical Concerns (Business Domains):** Imagine an e-commerce system where your code is organized into distinct vertical columns based on business features—like a `PaymentService`, an `OrderService`, and an `InventoryService`. Each column handles its own independent business logic.  
    
- **Horizontal Concerns (System-wide Features):** Security, logging, and error handling don't fit into just one of those columns. Instead, they form horizontal rows that slice through **every single vertical column** at the exact same level.


**Common Examples of Cross-Cutting Concerns**  

- **Security & Authentication:** Verifying user identities (JWT validation, OAuth2) and enforcing role-based access control across all API endpoints.  
    
- **Logging & Distributed Tracing:** Tracking a single user request as it travels through multiple network hops across various services (using tools like OpenTelemetry, Zipkin, or Jaeger).  
    
- **Service Discovery:** Allowing microservices to find and communicate with each other dynamically without hardcoding IP addresses.  
    
- **Resilience & Fault Tolerance:** Implementing circuit breakers, retries, and rate-limiting to prevent a failure in one service from cascading through the system.  
    
- **Configuration Management:** Centralizing environment variables and secret management so services don't have to manage local configuration files.


**How They Are Managed**  
Instead of rewriting this logic inside every microservice, engineering teams typically handle cross-cutting concerns using specific architectural patterns:  

- **API Gateway:** Acting as a single entry point for all client requests, the gateway handles edge concerns like authentication, rate limiting, SSL termination, and global routing.  
    
- **Service Mesh:** Utilizing a sidecar proxy (like Envoy or Istio) deployed alongside each service to handle network-level concerns like service discovery, encryption (mTLS), and distributed tracing automatically.  
    
- **Shared Libraries/SDKs:** Packaging boilerplate code (like standard logging formats or database connection pools) into custom libraries that services import, though this introduces language coupling.

# Spring cloud gateway

This will sit in between our client applications and microservices. 
Go to Spring.io and select Spring cloud gateway and there is official documentation for that  [https://spring.io/projects/spring-cloud-gateway](https://spring.io/projects/spring-cloud-gateway)

### How to build an api gateway

Go to [https://start.spring.io](https://start.spring.io)  and create a new project add dependencies  Gateway ,  EurekaDiscoveryClient , configClient, springboot actuator

use 

```

<dependency>  
    <groupId>org.springframework.cloud</groupId>  
    <artifactId>spring-cloud-starter-gateway-server-webflux</artifactId>  
</dependency>
```

instead of gatewayserver web-mvc



and add google jib related configurations to build in the pom.xml 

```

  
<plugin>  
    <groupId>com.google.cloud.tools</groupId>  
    <artifactId>jib-maven-plugin</artifactId>  
    <version>3.5.1</version>  
    <configuration>       
    <to>          
    <image>${project.artifactId}:${project.version}</image>  
    </to>    
    </configuration>
</plugin>

```


and in the applications.yml file we need to include 

```
management:
	endpoint:  
		  gateway:  
			    access: unrestricted
```

and we can mention the port details and eureka related properties

```
server:  
  port: 8072
```

```
eureka:  
  instance:  
    prefer-ip-address: true  
  
  client:  
    fetch-registry: true  
    register-with-eureka: true  
  
    service-url:  
      defaultZone: http://localhost:8070/eureka/
```

and we need to mention the gateway related properties which lets the gateway server to know about the other services

```
spring:  
  cloud:  
    gateway:  
      server:  
        webflux:  
          discovery:  
            locator:  
              enabled: true
```

now start the applications in the order 

configserver. -> eurekaserver ->  (needed applications) -> gateway 


### How to make it support lowercase service Id

in the applications.yml file of gateway server add 

```
lower-case-service-id: true
```

under the gateway server enabled 

```
spring:  
  cloud:  
    gateway:  
      server:  
        webflux:  
          discovery:  
            locator:  
              enabled: true  
              lower-case-service-id: true
```

so we can call 

```
http://localhost:8072/accounts/accountService/findUserByEmail?email=psaneesh99@gmail.com
```

instead of 

```
http://localhost:8072/ACCOUNTS/accountService/findUserByEmail?email=psaneesh99@gmail.com
```


The complete application.properties for gateway server


```
server:  
  port: 8072  
  
spring:  
  cloud:  
    gateway:  
      server:  
        webflux:  
          discovery:  
            locator:  
              enabled: true  
  
  config:  
    import: "optional:configserver:http://localhost:8071/"  
  application:  
    name: gateway  
  main:  
    banner-mode: 'off'  
management:  
  endpoints:  
    web:  
      exposure:  
        include: "*"  
    #this is used to see the info which we added on the bottom, by adding this we can see  
    # the info by accessing the url actuator/info  info:  
    env:  
      enabled: true  
  
  endpoint:  
    gateway:  
      access: unrestricted  
  
eureka:  
  instance:  
    prefer-ip-address: true  
  
  client:  
    fetch-registry: true  
    register-with-eureka: true  
  
    service-url:  
      defaultZone: http://localhost:8070/eureka/  
  
info:  
  app:  
    name: "gateway"  
    description: "gateway server description"  
    version: "1.0.0"  
  
endpoint:  
  shutdown:  
    enabled: true
```
### Why did Spring Cloud Gateway choose WebFlux?

A gateway sits in front of every microservice:

```
Client
   ↓
Gateway
   ↓
Accounts
Loans
Cards
Notifications
```

The gateway spends most of its time:

- Receiving requests
- Forwarding requests
- Waiting for downstream services

It doesn't do much business logic.

This is exactly the type of workload where reactive I/O shines.

So the original Spring Cloud Gateway team built it on:

```
Spring WebFlux + Netty
```

not MVC.


### Implementing custom routing using spring clound gateway 

instead of calling  http://localhost:8072/ACCOUNTS/accountService/findUserByEmail?email=psaneesh99@gmail.com

We can add a prefix to the url like https://localhost:8072/bookslots/accounts/accountService....   we can replace the ACCOUNTS with a custom url bookslots/accounts 

inorder to create this we need to create a new bean in the main application of the edgeserver

```
@Bean  
public RouteLocator routelocatorConfig(RouteLocatorBuilder builder) {  
    return builder.routes().route(  
          r -> r.path("/bookslots/accountService/**")  
                .filters(f -> f.rewritePath("/bookslots/accountService/(?<segment>.*)", "/${segment}"))  
                .uri("lb://ACCOUNTS")  
    ).build();  
}
```

we should put this inside the main applciation of the edge server after the main method

what it will do is it will replace the ACCOUNTS with the path /booking/accountService

so we can call the url like this

```
http://localhost:8072/bookslots/accountService/accountService/findUserByEmail?email=psaneesh99@gmail.com
```

### How to define multiple url like this

just we need to add a new .route before the .build

```
@Bean  
public RouteLocator routelocatorConfig(RouteLocatorBuilder builder) {  
    return builder.routes().route(  
          r -> r.path("/bookslots/accountService/**")  
                .filters(f -> f.rewritePath("/bookslots/accountService/(?<segment>.*)", "/${segment}"))  
                .uri("lb://ACCOUNTS")  
    ).route(  
          r -> r.path("/bookslots/bookingService/**")  
                .filters(f -> f.rewritePath("/bookslots/bookingService/(?<segment>.*)", "/${segment}"))  
                .uri("lb://BOOKING")  
    ).build();  
}
```

 explanation 

```
return builder.routes()
```

This is to indicate that we are going to define custom routes, and we are using builder pattern to pass each routes in to this and create a routeLocator object 
and inside each route   each path is known as route predicate   
```
"/bookslots/accountService/**" 
```
this indicates that any url which starts with the pattern   /bookslots/accountService/   

example 
```
/bookslots/accountService/findUser
/bookslots/accountService/login
/bookslots/accountService/test/abc
```


```
.filters(...)
```

and . filters allows us to modify requests before forwarding it 

Examples:

- Add headers
- Remove headers
- Validate requests
- Rewrite paths

in our case we are rewring the path

```
.filters(f -> f.rewritePath("/bookslots/accountService/(?<segment>.*)", "/${segment}"))
```

here there are two parts 

```
"/bookslots/accountService/(?<segment>.*)"
```

and 

```
"/${segment}"
```

This is a kind of saving mechanism  we will save whatever coming after /bookslots/accountService. in to variable called segment  and convert it to  / segment.      so the segment value will be   /accountService/findUserByEmail?email=psaneesh99@gmail.com 

and the last final part 

```
.uri("lb://ACCOUNTS")
```

it is a resource indicator , indicating to which resource we need to forward this request to , so the gateway server asks for the ACCOUNTS microservice to the eureka server and forwards the  request  /accountService/findUserByEmail?email=psaneesh99@gmail.com    to that 

so the original url will look like this 

```
http://localhost:8081/accountService/findUserByEmail?email=psaneesh99@gmail.com
```

then we added gateway and made it to 


```
http://localhost:8072/ACCOUNTS/accountService/findUserByEmail?email=psaneesh99@gmail.com
```

and then we use route forwaring and made it to 

```
http://localhost:8072/bookslots/accountService/accountService/findUserByEmail?email=psaneesh99@gmail.com
```


## Sending the header inside the response

we can  add a new filter which is responsible for sending the header inside the response 
In the official springcloud gateway documentation go to gateway filter factory -> add response header gateway filter factory we can see the application.yaml based configuration there

we can also add the same in the java configuration ( this is what recommended)

```
addResponseHeader("timestamp", LocalDateTime.now().toString()
```

inside the filters we can add this 

```
.filters(f -> f.rewritePath("/bookslots/accountService/(?<segment>.*)", "/${segment}").addResponseHeader("timestamp", LocalDateTime.now().toString()))
```

and we can see this data in the header section of the response  [[request header.png]]


and same thing we can append to the request header and this value will be passed to every request. For that we just need to change this responseheader to requestheader

```
.addRequestHeader("requestheaderkey", "this is the request header value")
```

we can append this with the filter values 

```
.filters(f -> f.rewritePath("/bookslots/accountService/(?<segment>.*)", "/${segment}").addResponseHeader("timestamp", LocalDateTime.now().toString()).addRequestHeader("requestheaderkey", "this is the request header value"))
```
so the whole value becomes 

```
@Bean  
public RouteLocator routelocatorConfig(RouteLocatorBuilder builder) {  
    return builder.routes().route(  
          r -> r.path("/bookslots/accountService/**")  
                .filters(f -> f.rewritePath("/bookslots/accountService/(?<segment>.*)", "/${segment}").addResponseHeader("timestamp", LocalDateTime.now().toString()).addRequestHeader("requestheaderkey", "this is the request header value"))  
                .uri("lb://ACCOUNTS")  
    ).route(  
          r -> r.path("/bookslots/bookingService/**")  
                .filters(f -> f.rewritePath("/bookslots/bookingService/(?<segment>.*)", "/${segment}"))  
                .uri("lb://BOOKING")  
    ).build();  
}
```

and we can add this value to the controller class 

```
public ResponseEntity<Response<UserDto>> findUserByEmail(@Email @RequestParam String email,@RequestHeader String requestheaderkey ) throws AccountException {
```
