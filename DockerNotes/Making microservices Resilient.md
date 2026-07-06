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


