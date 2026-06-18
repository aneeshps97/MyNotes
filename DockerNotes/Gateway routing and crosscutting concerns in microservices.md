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