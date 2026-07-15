Resilience means something which is capable of handling tough times and bouncing back . 

How do we avoid cascading failures : If one of our services slow or failing how it affect others . 

How do we handle failures gracefully with fallbacks : In a chain of ms how do we handle a  fallback mechanism . If one of the microservice is not working. 

How do we make our micro services self healing capable , In case of slow performing services how do we configure timeouts retries and give time for a failed service to recover . 

For this we use Resilience4j fault atolerence library designed for functional programing.  [[What is functional programing]]

### It uses following patterns to increase fault tolerance. 

#### Circuit breaker :
It is used to stop making requests to services when invoked services are failing. 
#### Fallback  : 
Alternative path to failing requests
#### Retry : 
Used to makes retries when a service has temporarily failed. 
#### Ratelimit :
LImits the number of calls that a service receives at a time. 
#### Bulk head : 
Limit the number of outgoing concurrent requests to a service to avoid overloading . 

### Circuit breaker

[[microservices circuit breaker.png]]


```
                     ms Bs
                    |

Client  -> api gateway ->  ms A
					^
                     |
                     ms C
```


microservice A is getting input from microservice B and C if any one of them failed A will wait indefinitely and also the api gateway will wait indefinitely .  In order to avoid this we use circuit breakers. 

When the circuit breaker detects a microservice is failing it will respond immediately without waiting and if detects that the service is failing too often it won't even make the call to that service by killing the request and send a fail response. And it will send some partial requests to the service to check whether it recovered or not . 

Think the circuit breaker as the circuit breaker in the house , if it is closed the electricity will flow  though that , if it is open no electricity will flow .  And if any electrical equipment failed this circuit breaker will open and it won't send the current to the equipment. 

[[circuit breaker.

we can implement the circuit breaker in two different places . 
1. At the gateway
2. At the individual microservices,


### At the gateway

we can implement the circuit breaker at the gateway 

For this we need to add the dependency of resilience.4j in the pom.xml of gateway server

```
<dependency>  
    <groupId>org.springframework.cloud</groupId>  
    <artifactId>spring-cloud-starter-circuitbreaker-reactor-resilience4j</artifactId>  
</dependency>
```

and in the applications.properties we need to add 

```
resilience4j.circuitbreaker:  
  configs:  
    default:  
      slidingWindowSize: 10  
      permittedNumberOfCallsInHalfOpenState: 2  
      failureRateThreshold: 50  
      waitDurationOpenState: 10000
```

slidingWindowSize : After how many requests should the circuit breaker move from closed to open state

permittedNumberOfCallsInHalfOpenState : How many requests are permitted in the half open state to determine whether to move from half open to open 

failureRateThreshold : what is the percentage of requests which should fail inorder to move from closed to open   or half open to closed

waitDurationOpenState : after how much time the system should wait in the open state before moving to the half open state

and 

we need to add a circuit breaker to the main application of the gateway server 

```
.circuitBreaker(config -> config.setName("accountsCircuitBreaker")))
```

we should add this to the .filter part of the application which we are trying to fire the request

```
.route(  
       r -> r.path("/bookslots/accountService/**")  
             .filters(f -> f.rewritePath("/bookslots/accountService/(?<segment>.*)", "/${segment}").addResponseHeader("timestamp", LocalDateTime.now().toString()).addRequestHeader("requestheaderkey", "this is the request header value")  
                   .circuitBreaker(config -> config.setName("accountsCircuitBreaker")))  
             .uri("lb://ACCOUNTS")
```

### How to test this 

after adding this start all the services and put a debugger point in the service which we are trying to fire the request to And when we fire this we will get gateway timed out after 10 request we will get service not available because the gate way stopped sending request to the service . and after removing this debugger point we will get proper response.


### How to implement a fallback mechanism

when we implement a circuit breaker we will be getting exception like 500 service not available if the circuit is open , we can avoid this and we can send custom error messages or do some tasks if the circuits are open , inorder to do this we need to implement a fallback mechanism , we need to create a custom controller class inside the gateway server application and just redirect the call to that controller class. 

example controller class 

```
@RestController  
public class FallbackController {  
    @RequestMapping("/contactSupport")  
    public Mono<String> fallback() {  
        return Mono.just("service is down please try again after sometime");  
    }  
}
```

we use Mono because the gateway is based on the reactive stack of springboot

and we need to add this to the gateway server main application.java

```
.setFallbackUri("forward:/contactSupport")
```

after the circuit breaker 

```
.route(  
       r -> r.path("/bookslots/accountService/**")  
             .filters(f -> f.rewritePath("/bookslots/accountService/(?<segment>.*)", "/${segment}").addResponseHeader("timestamp", LocalDateTime.now().toString()).addRequestHeader("requestheaderkey", "this is the request header value")  
                   .circuitBreaker(config -> config.setName("accountsCircuitBreaker").setFallbackUri("forward:/contactSupport")))  
             .uri("lb://ACCOUNTS")
```


and when the service fail it will call to that given url. 

### How to implement fallback mechanism in feignclient

From the spring official documentation  [https://docs.spring.io/spring-framework/reference/index.html](https://docs.spring.io/spring-framework/reference/index.html)  under the projects tab select spring cloud   or go to   [https://docs.spring.io/spring-cloud-openfeign/docs/current/reference/html/](https://docs.spring.io/spring-cloud-openfeign/docs/current/reference/html/)    select Feign spring cloud circuit breaker support. There we can read all the circuit breaker related configurations 

If Spring Cloud CircuitBreaker is on the classpath and `spring.cloud.openfeign.circuitbreaker.enabled=true`, Feign will wrap all methods with a circuit breaker.

```
spring:  
  cloud:  
    openfeign:  
      circuitbreaker:  
        enabled: true
```

we just need to add the dependency in the applications pom.xml and enable this property in the applications.yaml and spring will automatically do the rest

```
    <dependency>
      <groupId>org.springframework.cloud</groupId>
      <artifactId>spring-cloud-starter-circuitbreaker-resilience4j</artifactId>
    </dependency>
```

Also we need to add the circuit breaker properties which we added for the gateway server 

```
resilience4j.circuitbreaker:  
  configs:  
    default:  
      slidingWindowSize: 10  
      permittedNumberOfCallsInHalfOpenState: 2  
      failureRateThreshold: 50  
      waitDurationOpenState: 10000
```


when we add all this and put a debugger point in the method in the feign client method of the the service which we use to access using the feign client . we can see 

```

org.springframework.cloud.client.circuitbreaker.NoFallbackAvailableException: No fallback available.
	at org.springframework.cloud.client.circuitbreaker.CircuitBreaker.lambda$run$0(CircuitBreaker.java:31) ~[spring-cloud-commons-4.1.2.jar:4.1.2]
	at org.springframework.cloud.circuitbreaker.resilience4j.Resilience4JCircuitBreaker.getAndApplyFallback(Resilience4JCircuitBreaker.java:167) ~[spring-cloud-circuitbreaker-resilience4j-3.1.1.jar:3.1.1]
	at org.springframework.cloud.circuitbreaker.resilience4j.Resilience4JCircuitBreaker.run(Resilience4JCircuitBreaker.java:143) ~[spring-cloud-circuitbreaker-resilience4j-3.1.1.jar:3.1.1]
	at org.springframework.cloud.client.circuitbreaker.CircuitBreaker.run(CircuitBreaker.java:30) ~[spring-cloud-commons-4.1.2.jar:4.1.2]
	at org.springframework.cloud.client.circuitbreaker.observation.ObservedCircuitBreaker.run(ObservedCircuitBreaker.java:58) ~[spring-cloud-commons-4.1.2.jar:4.1.2]
	at org.springframework.cloud.openfeign.FeignCircuitBreakerInvocationHandler.invoke(FeignCircuitBreakerInvocationHandler.java:115) ~[spring-cloud-openfeign-core-4.1.1.jar:4.1.1]
	at jdk.proxy2/jdk.proxy2.$Proxy193.getBookingByUserForSpace(Unknown Source) ~[na:na]

```

This in the logs confirming the circuit breaker is working , but it is looking for a fallback 

### How to add a fallback in the feign client 

for that we need to create a fallback class and implement the feignclient interface which we are trying to access 

```
@FeignClient(name = "booking", fallback = Fallback.class)  
public interface BookingFeignClient {  
    @GetMapping("bookingService/booking/by-user")  
    public ResponseEntity<Response<List<BookingDTO>>> getBookingByUserForSpace(  
            @RequestParam int userId,  
            @RequestParam int spaceId,  
            @RequestHeader("transactionId") String transactionId  
    );  
}
```

if this is the feign client interface which we are trying to access the data we need to write a new class which implements this feignclient and we can return the data which we need when this service is down

```
@Component  
public class Fallback implements BookingFeignClient {  
    @Autowired  
    GenerateResponse generateResponse;  
    @Override  
    public ResponseEntity<Response<List<BookingDTO>>> getBookingByUserForSpace(int userId, int spaceId, String transactionId) {  
        List<BookingDTO> bookingDTOList = new ArrayList<>();  
        return generateResponse.formatResponse(ErrorCodes.DATA_RECEIVED_FROM_FALLBACK,bookingDTOList);  
    }  
}
```

also we need to mention this class name on top of the feign client interface 

```
@FeignClient(name = "booking", fallback = Fallback.class)  
```


now we will be redirect to this class when the other service is not responding , inorder to test this just put a debugger point in the other service's controller class in the method which we are calling and start that project in the debug mode. 

### How to handle timeouts 

By default the circuit breaker has a one sec timeout if the service is not responding after 1 sec it will move to the fallback mechanism . 
if we are not using the circuit breaker pattern we can configure this timeout using configuration in the applications.yaml  of gateway server. 

[https://docs.spring.io/spring-cloud-gateway/reference/spring-cloud-gateway-server-webflux/http-timeouts-configuration.html](https://docs.spring.io/spring-cloud-gateway/reference/spring-cloud-gateway-server-webflux/http-timeouts-configuration.html)  in the official documentation of spring cloud we can see the timeout configuration   (to get to this page just search spring cloud timeout configuration in gateway server in google)

and we can see the configuration there 

```
spring:
  cloud:
    gateway:
      httpclient:
        connect-timeout: 1000
        response-timeout: 5s
```

even though the official documentation is showing the above one we should use 

```
spring:  
  cloud:  
    gateway:  
      server:  
        webflux:  
          httpclient:  
            connect-timeout: 1000  
            response-timeout: 5s
```

because the gateway is build on the reactive stack 


if we are not using the circuit breaker and if the services are not responding the system will wait for a long time to get the response . we can use this configuration to avoid that . 

inorder to test this we need to give more time to this and then test whether it is working or not after the configured timeout we will get the gateway timeout exception 

```
spring:  
  cloud:  
    gateway:  
      server:  
        webflux:  
          httpclient:  
            connect-timeout: 100000  
            response-timeout: 25s
```

When we define this this is global for all the microservices . 

And same thing we can override in the main application file of the gateway server They do **not** tell the circuit breaker how long to wait.


```
.metadata(RouteMetadataUtils.CONNECT_TIMEOUT_ATTR, 2000).metadata(RouteMetadataUtils.RESPONSE_TIMEOUT_ATTR, 2000)  
.uri("lb://BOOKING")
```

before the uri we need to give this metadata 


# Retry pattern 

The retry pattern will make the multiple retry attempts when a server has temporarily failed . This pattern is very helpful in the scenarios like network disruption where the client request may successful after a retry attempt. 

The key components and the consideration of implementing the retry pattern 

Retry logic : Determine when and how many times to retry an operation , this can be based on factors such as errorCodes , exceptions or response status. 

Backoff strategy : Define a strategy to avoid overwhelming the system or  exacerbating the underlying issue. 

Circuit breaker integration : Consider combining the retry pattern with the circuit breaker pattern

 Idempotent operations : Ensure that the retried operation is idempotent meaning it produces the same result regardless of  how many times it is invoked. 

# How to implement a retry pattern 

In order to implement this we need to add a retry filter to the gateway server just like we did the circuit breaker before the uri just add 

```
.retry(retryConfig->retryConfig.setRetries(3).setMethods(HttpMethod.GET).setBackoff(Duration.ofMillis(100),Duration.ofMillis(1000),2,
true)
```

the whole filter code becomes 

```
.route(  
       r -> r.path("/bookslots/bookingService/**")  
             .filters(f -> f.rewritePath("/bookslots/bookingService/(?<segment>.*)", "/${segment}").retry(retryConfig->retryConfig.setRetries(3).setMethods(HttpMethod.GET).setBackoff(Duration.ofMillis(100),Duration.ofMillis(1000),2,true)))  
             .uri("lb://BOOKING")  
).build();
```

```
.setRetries(3)
```

This specifies **how many times the gateway retries after the original request fails**.

```
Client
   │
   ├── Attempt 1 (original request)
   │        ❌ Failed
   │
   ├── Retry 1
   │        ❌ Failed
   │
   ├── Retry 2
   │        ❌ Failed
   │
   ├── Retry 3
   │        ❌ Failed
   │
   └── Return failure to client
```

```
1 original + 3 retries = 4 requests
```

```
.setMethods(HttpMethod.GET)
```

This tells the gateway **which HTTP methods are allowed to retry**.

If the request is:

```
GET /buildVersion
```

it can retry.

#### setBackoff(...)

```
.setBackoff(
    Duration.ofMillis(100),
    Duration.ofMillis(1000),
    2,
    true
)
```

Backoff controls **how long the gateway waits before each retry**.

Initial Delay : after how many milliseconds  retry should happen

Maximum Delay : after how many milliseconds the retry should fail , like 100 times retry would make the maxdelay more than 1000 so the request will stop retrying after that 

Multiplier:  each retry delay is multiplied by this number.  after first retry second retry will wait for 200 milliseconds then it will become 400 then 800 and so on

based on previous value : based on the previous value whether it should multiply the time to wait or not

how to test this : put a debugger in the service you are going to hit and after request fail check the console in the debugger we can see the retry 

[[retry in gateway.png]]

### Implementing retry pattern in individual microservices

For this first we need to create a fallback method in the controller to execute when the maximum number of retries are exhausted. 

For example if the method is 

``` 
@GetMapping("/buildVersion")  
public ResponseEntity<String> getBuildVersion(){  
    logger.info("getBuildVersion is called");  
    throw new NullPointerException();  
}  
```


we need to create a fallback method 

```
public ResponseEntity<String> getBuildVersionFallback(Throwable t){  
    logger.info("getBuildVersionFallback is called");  
    return ResponseEntity.ok("fallback build version");  
}
```

and on top of the original method we need to annotate 

```
@Retry(name="getBuildInfo",fallbackMethod="getBuildVersionFallback")  
@GetMapping("/buildVersion")  
public ResponseEntity<String> getBuildVersion(){
```

and in the application.properties we need to add 

```
resilience4j:  
  retry:  
    configs:  
      default:  
        max-attempts: 3  
        wait-duration: 2s  
        enableExponentialBackoff: true  
        exponentialBackoffMultiplier: 2
```

so it will wait for atmost 6 seconds

```
Attempt 1
   ↓ fails
Wait 2 seconds
Attempt 2
   ↓ fails
Wait 4 seconds
Attempt 3
   ↓ fails
Fallback (or exception)
```

make sure that the gateway timeout is more than that
in the gateway server we configure the timeout like this so here i configured 10s

```
spring:  
  cloud:  
    gateway:  
      server:  
        webflux:  
          httpclient:  
            connect-timeout: 1000  
            response-timeout: 10s
```


what if we want to ignore the nullpointer exception and for some other exception only we want to execute this retry but go for a direct fallback 

we need to add 

```
ignoreExceptions:  
          - java.lang.NullPointerException

```

```
resilience4j:  
  retry:  
    configs:  
      default:  
        ignoreExceptions:  
          - java.lang.NullPointerException
```

in the application.properties of the client application.
