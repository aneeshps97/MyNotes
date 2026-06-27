How services locate each other inside a network - > service discovery 
How do new service instances enter into the network -> service registration 
How to make them properly load balanced -> load balancing 

In the traditional application there will be one instance of an application present and we can hardcode the Ip of the application or use the dns name into another application which will make the communication between them possible. But in a cloud microservice architecture the applications will be scaling up or down continuously , so the ip and dns will change constantly . to the ip and dns record keeping will not be possible because every time it create a new instance it will be different . 

How to resolve the problem for cloud native applications. 

For cloud native applications service discovery is the perfect solution for this. Whenever a new instance is created it should be registered in the registry and when it is terminated it should be appropriately removed automatically. 

The system acknowledges that multiple applications that can be active simultaneously when an application needs to communicate with a backing service it performs a lookup in the registry to determine the ip address to connect to . 

There are two different methods of service discovery 
1. Client side service discovery 
2. Server side service discovery
 Client side service discovery
In this approach applications are responsible for registering themselves with a registry during startup and during shutdown they need to deregister themselves. when an application needs to communicate with another it requests the ip address of that application from the registry. 

The major advantage of client side service discovery is that load balancing can be assigned by many algorithm.  The main disadvantage is that it assigns more responsibilities to the developers and it results in deploying one or more services (service registry)

To resolve all these we can use server side service discovery. 

in the server side discovery the microservice will call to a permanent ip of the loadbalancer and it will handle the traffic , the microservices register to this load balancer , the eureka server is just a database it is not actually handling any traffic , but in the case of loadbalancer it is handling the heavy duty of traffic management.

#### Spring cloud support for client-side service discovery

Spring cloud projects make service discovery and registration setup trivial to undertake with the help of the below components. 

1. spring cloud Netflix's eureka service which act as a service discovery agent. 
2. Spring cloud load balancer : library for client side load balancing 
3. Netflix feign client to lookup for a service b/w micro services

This eureka server is responsible for service discovery and each micro service register themselves to this erureka server up on startup and when shutting down they will deregister. 

##### How to make a eureka server
Go to springCloud -> springCloud netflix and you can find the instructions to build a service discovery agent. 

[[how to make a eureka server.png]]

create a new project and add this dependency 


```
<dependency>  
    <groupId>org.springframework.cloud</groupId>  
    <artifactId>spring-cloud-starter-netflix-eureka-server</artifactId>  
</dependency>
```

and add this annotation to main class 

```
@EnableEurekaServer
```

and add this in the application.yml file in the eureka server

```
spring:  
  application:  
    name: eurekaserver  
  config:  
    import:"optional:configserver:http://localhost/8071/"
```

the config.import tells the eureka server to fetch its configuration from config server and in the config server we need to create a new configuration for eureka server . 

and add this in the eureka server application.yml 

since other microservices have a dependency over the eureka server we need to add the health check related properties there 

```
spring:  
  application:  
    name: eurekaserver  
  
  config:  
    import: "optional:configserver:http://localhost:8071/"  
  
management:  
  endpoints:  
    web:  
      exposure:  
        include: "*"  
  
  health:  
    readiness-state:  
      enabled: true  
  
    liveness-state:  
      enabled: true  
  
    probes:  
      enabled: true
```

This same thing we added in the springCloud server because other microservices have dependency with that too. 

The other configurations we can give it in the config server . In the config server application where we give all the configurations we can create a new file called erurekaserver.yml   and add these properties. 

since it has no prod or dev properties we only need to create this one file. 

and add these in that configuration file 

```
server:  
  port: 8070  
  
eureka:  
  instance:  
    hostname: localhost  
  
  client:  
    fetch-registry: false  
    register-with-eureka: false  
    service-url:  
      defaultZone: http://${eureka.instance.hostname}:${server.port}/eureka/
```

and start the config server and eureka server . 

we need to add the config server dependency to the eureka server 

```
<dependency>  
    <groupId>org.springframework.cloud</groupId>  
    <artifactId>spring-cloud-starter-config</artifactId>  
</dependency>
```
go to 

```
http://localhost:8070
```

and we can see the eureka server dashboard. Once other microservices register in the eureka server we can see them in this dashboard. 

#### How to register the microservices with eureka server

in the pom.xml of microservice we need to add the eureka discovery client. 

go to start.spring.io and create a new project , add the dependency eureka discovery client 

and click on explore and we can find the dependency details which we added earlier. 

```
    <dependency>
      <groupId>org.springframework.cloud</groupId>
      <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
    </dependency>
```

and in the application.yml fiel we need to mention all the eureka related properties. 

under management tab

```
#this is to view the info which we give in the root by accessing the url actuator/info

info:  
  env:  
    enabled: true  

#This indicates who can access the actuator shutdown endpoint we give this to deregister from eureka when 
#shutting down the application

endpoint:  
  shutdown:  
    access: unrestricted
```

in the root 

```
eureka: 

	#by defaut eureka register with the host name like account-service with this enabled it will register with   #machines ip address, because the host name resolution may not work properly 
	
  instance:  
    prefer-ip-address: true  
  
  #This tells the service to register yourself with the eureka
  
  client:  
    fetch-registry: true  
    register-with-eureka: true  
  
    service-url:  
      defaultZone: http://localhost:8070/eureka/  
  
info:  
  app:  
    name: "accounts"  
    description: "Accounts description"  
    version: "1.0.0"  
  
endpoint:  
  shutdown:  
    enabled: true
```


And after adding all this go to the eureka dashboard (http://localhost:8070) and we can see the microservice there [[Eurekaserver.png]]

### How to unregister a microservice from eureka server

Because we enabled endpoint.shutdown.enabled = true.  we can shutdown the application using 

```
localhost:8080/actuator/shutdown
```

the port can be the port of the application we are going to shut down

because it is a post request we need to invoke it from the postman


### Feign client code changes to invoke other microservices

To implement the internal communication between the microservices we need to use feign client. 

1. Find open feign dependancy by creating a new project and add dependancy openfeign and then click on explore and find the dependancy. 
```
    <dependency>
      <groupId>org.springframework.cloud</groupId>
      <artifactId>spring-cloud-starter-openfeign</artifactId>
    </dependency>
```

and in the main application class we need to add the annotation 

```
@EnableFeignClents 
```

so that the microservices can communicate with other microservice. 

and in the microservice we need to add a new package client and add a new interface BookingFeign (the feign class you need)

and we need to annotate that class with  
```
@FeignClient("booking")
```

the name should be whatever name we gave in the eureka client information.  and inside that we need to write the abstract method of the url we need to hit , it should be exactly the same as we gave in the controller class of the other microservice

```  
@FeignClient("booking")  
public interface BookingFeignClient {  
    @GetMapping("bookingService/booking/by-user")  
    public ResponseEntity<Response<List<BookingDTO>>> getBookingByUserForSpace(  
            @RequestParam int userId,  
            @RequestParam int spaceId  
    );  
}
```

The feign client will fetch all the names which are registered under the name Booking and it will cache this information for 30 seconds and use it for another request which come in that period. 

we can just inject this feign client class in to our service class and directly call the methods which we declared there.

```
@Service  
@AllArgsConstructor  
public class ClientServiceImpl implements IClientService{  
    private BookingFeignClient bookingFeignClient;  
    @Override  
    public Response<List<BookingDTO>> getBookingByUserForSpace(int userId, int spaceId) {  
        return bookingFeignClient.getBookingByUserForSpace(userId,spaceId).getBody();  
    }  
}
```


### Updating docker compose file to adapt service discovery changes

create a new section of service in the name eureka server in the docker compose file 

The erureka server starts in the port 8070 by default so we need to mention that i ports and in health check 

and it should depend on configServer to fetch the configurations needed

```
depends_on:  
  configserver:  
    condition: service_healthy
```

we should add the eureka server as a config client too 

we need to create a new environment variable inside the eurekaserver service 

```
environment:  
  SPRING_APPLICATION_NAME: "eurekaserver"
```

we need to create a new configuration in the common config we need to create some configurations for the eureka server so that all the microservices can extend them


```
microservice-eurekaserver-config:  
  extends:  
    service: microservice-configserver-config  
  depends_on:  
    erurekaserver:  
      condition: service_healthy  
  environment:  
    EURKEKA_CLIENT_SERVICE_URL_DEFAULTZONE: "http://eurekaserver/8070/eureka"
```

and all the other services should extend them 

sample of full common config and  docker compose file 

common config

```
services:  
  network-deploy-services:  
    networks:  
      - abc  
  microservice-base-config:  
    extends:  
      service: network-deploy-services  
    environment:  
      SPRING_PROFIES_ACTIVE: default  
      SPRING_RABBITMQ_HOST: rabbit  
    deploy:  
      resources:  
        limits:  
          memory: 700m  
  microservice-configserver-config:  
    extends:  
      service: microservice-base-config  
    depends_on:  
      configserver:  
        condition: service_healthy  
    environment:  
      SPRING_CONFIG_IMPORT: "configserver:http://configserver:8071"  
  
  microservice-eurekaserver-config:  
    extends:  
      service: microservice-configserver-config  
    depends_on:  
      erurekaserver:  
        condition: service_healthy  
    environment:  
      EURKEKA_CLIENT_SERVICE_URL_DEFAULTZONE: "http://eurekaserver/8070/eureka"
```


docker compose file 

```
services:  
  
  configserver:  
    image: springcloud:0.0.1-SNAPSHOT  
    container_name: configserver-ms  
    ports:  
      - "8071:8071"  
    extends:  
      file: common-config.yml  
      service: microservice-base-config  
  
    healthcheck:  
      test: "curl --fail --silent localhost:8071/actuator/health/readiness|grep UP || exit 1"  
      interval: 10s  
      timeout: 5s  
      retries: 10  
      start_period: 10s  
  
  
  eurekaserver:  
    image: springcloud:0.0.1-SNAPSHOT  
    container_name: configserver-ms  
    ports:  
      - "8070:8070"  
    extends:  
      file: common-config.yml  
      service: microservice-base-config  
  
    environment:  
      SPRING_APPLICATION_NAME: "eurekaserver"  
  
    healthcheck:  
      test: "curl --fail --silent localhost:8070/actuator/health/readiness|grep UP || exit 1"  
      interval: 10s  
      timeout: 5s  
      retries: 10  
      start_period: 10s  
    depends_on:  
      configserver:  
        condition: service_healthy  
  
  accounts:  
    image: accountservice:0.0.1-SNAPSHOT  
    container_name: accounts-ms  
    ports:  
      - "8081:8081"  
    extends:  
      file: common-config.yml  
      service: microservice-eurekaserver-config  
  
    environment:  
      SPRING_CONFIG_IMPORT: "configserver:http://configserver:8071/"  
      SPRING_PROFILES_ACTIVE: default  
      SPRING_APPLICATION_NAME: accounts  
  
    depends_on:  
      configserver:  
        condition: service_healthy  
  
  booking:  
    image: bookingservice:0.0.1-SNAPSHOT  
    container_name: booking-ms  
    ports:  
      - "8083:8083"  
    extends:  
      file: common-config.yml  
      service: microservice-eurekaserver-config  
  
    environment:  
      SPRING_CONFIG_IMPORT: "configserver:http://configserver:8071/"  
      SPRING_PROFILES_ACTIVE: default  
      SPRING_APPLICATION_NAME: booking  
  space:  
    image: spaceservice:0.0.1-SNAPSHOT  
    container_name: space-ms  
    ports:  
      - "8082:8082"  
    extends:  
      file: common-config.yml  
      service: microservice-eurekaserver-config  
  
    environment:  
      SPRING_CONFIG_IMPORT: "configserver:http://configserver:8071/"  
      SPRING_PROFILES_ACTIVE: default  
      SPRING_APPLICATION_NAME: space  
networks:  
  abc:  
    driver: "bridge"
```