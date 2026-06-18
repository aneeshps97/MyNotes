			note: if we are using springboot 3.2 we should change the springboot cloudClient version to 
			  <spring-cloud.version>2023.0.1</spring-cloud.version>

			  note when working with config server always start the config server before starting the other
			  services if we do start the other services first the services will pick up the local data from 
			  the services own dev or prod profile based on the configuration. 

				if not call the actuator/refresh and it will work fine 
				

Spring config server is used to manage configurations just like we did using [[How to read configurations]]. and [[How to use profiling in springboot]]  the main drawbacks of that approach was 
1. We need to give instructions change the configurations manually everytime we run the application [[Externalizing configuration]]
2. If our git code is exposed our application security is compromised.
3. When instances grow it is difficult to give this configurations to each and every instances using command.
4. There we are giving them as plain text it does not support any encryption or decryption.
5. If we want to make any changes to the configuration we need to restart the application.
The solution for these issues are to use a  *Springboot config server* .

What is a springboot config server. 
It is an another application which we use to inject the configurations to the applications which we need. It works on a server client model
it is the server application which has all the configurations required and our applications are the client applications which listens to the configuration changes. 

### How to make the springboot config server

Go to  [ https://spring.io](https://spring.io) 

Under the projects select Spring Cloud [[springCloud.png]]  and under that select Spring Cloud config [[springCloudConfig.png]]. There we can read about the springboot config server.

### How to create the springboot config server application

Go to [https://start.spring.io](https://start.spring.io) and create a project and as dependencies we should give config server and actuator as dependency [[spring cloud dependencies.png]]

in the main application class add @EnableConfigServer

```
@SpringBootApplication  
@EnableAutoConfiguration
@EnableConfigServer  
public class SpringcloudApplication {  
  
    public static void main(String[] args) {  
       SpringApplication.run(SpringcloudApplication.class, args);  
    }  
  
}
```

inside the resources folder create a folder named config [[configFolder.png]] 

and put the application.yaml files we need inside it , but rename them in the corresponding application's name
like if we want to configure it for accounts microserver we should change this to  accounts.yaml , accounts-qa.yaml   we should remove 
from applications.yaml (in accountsmicroservice)

```
config:  
  import:  
    - "application-dev.yaml"  
    - "application-qa.yaml"  
spring:  
  profiles:  
    active:  
      - "dev"
```

and  from applications-dev.yaml (in accountsmicroservice)
```
spring:  
  config:  
    activate:  
      onprofile: "dev"
```

That is in the application files we should only provide the properties which are getting overriden and nothing else

like this 
```
build:  
  version: "1.1"  
  
env:  
  environment: "dev"  
  
contact-info:  
  message: contact info for booking microservice dev  
  contact-details:  
    email: psaneesh010@gmail.com  
    phone: 7907736835  
  onCall-support:  
    +91 9656932559  
    +91 9544732807

```

in the end the file structure will look like this 
[[config folder structure.png]]


in the application.yaml file of spring cloud give 

```
server:  
  port: 8071  
  
spring:  
  application:  
    name: ConfigServer  
  profiles:  
    active: native  
  cloud:  
    config:  
      server:  
        native:  
          search-locations: "classpath:/config"
```

using the search location only we will get the properties loaded 

and if we go to the website *http://localhost/accounts/prod*   we can see 


```
{
  "name": "accounts",
  "profiles": [
    "prod"
  ],
  "label": null,
  "version": null,
  "state": null,
  "propertySources": [
    {
      "name": "classpath:/config/accounts-prod.yaml",
      "source": {
        "build.version": "1.2",
        "env.environment": "prod",
        "contact-info.message": "contact info for account microservice production",
        "contact-info.contact-details.email": "psaneesh010@gmail.com",
        "contact-info.contact-details.phone": 7907736835,
        "contact-info.onCall-support": "+91 9656932559 +91 9544732807"
      }
    },
    {
      "name": "classpath:/config/accounts.yaml",
      "source": {
        "build.version": "1.0",
        "env.environment": "none",
        "contact-info.message": "contact info for account microservice",
        "contact-info.contact-details.email": "psaneesh010@gmail.com",
        "contact-info.contact-details.phone": 7907736835,
        "contact-info.onCall-support": "+91 9656932559 +91 9544732807"
      }
    }
  ]
}

```


### How to connect the config server with other microservices

we should remove 

```
spring:  
  config:  
    import:  
      - "application-dev.yaml"  
      - "application-prod.yaml"

```

this from the main application.yaml of microservice as well as from the accounts.yaml from the config server config

in the application.yaml file of account microservice

```
spring:  
  config:  
    import: "optional:configserver:http://localhost:8071/"
```

and in the account service application we need to give the config client related dependencies for that go to spring.io and create a project and add *config client * as a dependency
[[config client.png]]

press explore and find the pom.xml. [[config client explore.png]]

copy and paste the <spring-cloud.version>2025.0.2</spring-cloud.version>   in the properties

```
<properties>  
    <java.version>17</java.version>  
    <maven.compiler.release>17</maven.compiler.release>  
    <lombok.version>1.18.30</lombok.version>  
    <spring-cloud.version>2025.0.2</spring-cloud.version>  
</properties>
```

and springcloud dependency

```
<dependency>

      <groupId>org.springframework.cloud</groupId>

      <artifactId>spring-cloud-starter-config</artifactId>

    </dependency>
```

and dependency management 

```
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
  
```

The name in the application.yaml and name of the yaml fie in the config server should be the same 

final application.yaml of accountService

```
server:  
  address: 0.0.0.0  
  servlet:  
    context-path: /accountService  
  port: '8081'  
spring:  
  config:  
    import: "configserver:http://localhost:8071/"  
  datasource:  
    driver-class-name: com.mysql.cj.jdbc.Driver  
    username: root  
    url: jdbc:mysql://localhost:3306/accountservicedb  
    password: Psaneesh010@  
  application:  
    name: accounts  
  main:  
    banner-mode: 'off'  
  jpa:  
    database-platform: org.hibernate.dialect.MySQLDialect  
    hibernate:  
      ddl-auto: update  
    show-sql: 'true'  
  profiles:  
    active:  
      - "dev"  
  
  
build:  
  version: "2.0 mainapplication"  
  
env:  
  environment: "none"  
  
contact-info:  
  message: contact info for account microservice  
  contact-details:  
    email: psaneesh010@gmail.com  
    phone: 7907736835  
  onCall-support:  
    +91 9656932559  
    +91 9544732807

```


final accounts.yaml of config server

```
build:  
  version: "1.0"  
  
env:  
  environment: "none"  
  
contact-info:  
  message: contact info for account microservice  
  contact-details:  
    email: psaneesh010@gmail.com  
    phone: 7907736835  
  onCall-support:  
    +91 9656932559  
    +91 9544732807
```

final accounts-dev.yaml in config server 

```
build:  
  version: "1.1 from spring cloud"  
  
env:  
  environment: "dev"  
  
contact-info:  
  message: contact info for account microservice dev  
  contact-details:  
    email: psaneesh010@gmail.com  
    phone: 7907736835  
  onCall-support:  
    +91 9656932559  
    +91 9544732807
```

now if we print the build it will print 1.1 from spring cloud


***we just removed the imports from local and import changed to config server thats all

### How to load configuration files from the local file system 

for that the only change we need to do is 
in the applications.yaml file of configServer we should give 


```
native:  
  search-locations: "file:///Users//aneesh//myProjects//learing from udemy//configuration using spring cloud//configurations"
```

we just need to change the search-locations

### How to configure it using github

This is the most recommended approach because 

1. we can see the history of changes
2.  It is easy to maintain
3. more secure

How to do it
 1.  move the configuration files in to a github repo. [[configuration inside git.png]]
 2.  copy the https url of the github repo [[http url of git.png]]
  in the profiles.active part of applications.yaml we should give 
  
```
  spring:  
  application:  
    name: ConfigServer  
  profiles:  
    active: git  
    #native
```

and in the cloud config server instead of native we need to use git configurations

```
cloud:  
  config:  
    server:  
      git:  
        uri: https://github.com/aneeshps97/configurations.git  
        default-label: main  
        timeout: 5  
        clone-on-start: true  
        force-pull: true
```


### How to encrypt and decrypt the values

while using spring cloud if we save all the configuration values inside the git without any encrption and decrption the data becomes vulnerable to hacking . so we need to encrypt and store the value

inorder to do that we need to provide 

```
encrypt:  
  key: "thisistheencryptionkeywhichweareusing"
```

this in the application.yaml file of the springboot config server
[[config server encryption key.png]]

this will be the key for encryption 

and inorder to find the encrypted value we need to hit the post man with the url 

```
http://localhost:8071/encrypt
```

with method post and data in raw text 

this will gives us the encrypted data and we need to insert that value in to the config files inside the git 
[[encrypted value inside the config files.png]]

and then we can restart the spring cloud application and hit the url to get the configurations in the accoutsmicroservice and we will get a decrypted value in the result
[[decrypted value in postman.png]]


### How to change the value without restarting any application.

Right now we need to restart the spring cloud server inorder for the changes in the configuration files to reflect in our main applications .[[cloud config value changed.png]]    [[cloud config value changed and not reflecting it .png]]

we can fix this using actuator . Inorder to do this we need to add a configuration to the applications.yaml of accounts microservice

```
management:   
  endpoints:  
    web:  
      exposure:  
        include: "*"
```

we need to add actuator dependency inside the pom.xml as well

```
<dependency>  
    <groupId>org.springframework.boot</groupId>  
    <artifactId>spring-boot-starter-actuator</artifactId>  
</dependency>
```

Also if we are using the dto class which helps save the info we need to change it from record to class , record will not allow to reset the values once set it we need to restart the application inorder to reset the data

```
@ConfigurationProperties(prefix = "contact-info")  
public record ConfigurationRecordDto(String message, Map<String, String> contactDetails, List<String> onCallSupport) {  
}
```

instead of this we need to use 

```
@ConfigurationProperties(prefix = "contact-info")  
@Getter  
@Setter  
public class ConfigurationRecordDto {  
    String message;  
    Map<String, String> contactDetails;  
    List<String> onCallSupport;  
}
```

and everytime we make a change to the configuration call the url 

```
http://localhost:8081/accountService/actuator/refresh
```

This will reset the values.

If there are multiple microservices and several instances of it manually calling the url of each of the micro services is not practical so we need a better mechanism to call all the url at once for that we can use spring cloud bus

### spring cloud bus

###### using this we only need to hit the actuator/refresh on one service and it will reflect on all the services it is done internally by using rabbit mq or kafka


For this first we need to run rabbit mq  [https://www.rabbitmq.com](https://www.rabbitmq.com)  go to the website either download it and run or run it using docker 
[[rabbit mq getting started.png]]   [[docker pull command.png]]

and we need to add the dependency in all the microservices including the spring config server

```
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-bus-amqp</artifactId>
</dependency>
```

after the dependency of spring cloud starter config we can give this 

and start the rabbit mq in our local  using docker 
make sure to install this same version apparently this is the only version working

```
docker run -d --name rabbitmq -p 5672:5672 -p 15672:15672 rabbitmq:3-management
```

and the application.yaml file of every microservices including the spring cloud we need to give 

```
spring:  
  rabbitmq:  
    host: "localhost"  
    port: 5672  
    username: "guest"  
    password: "guest"
```

and in the application.yaml file of microservices excluding spring cloud we should give 

```
amqp:  
  rabbit:  
    auto-declare: true
```

and in the spring cloud application use this version of spring cloud if the springboot version is   <version>3.2.5</version>

```
<spring-cloud.version>2023.0.1</spring-cloud.version>
```

and try restarting the ide for this to work 


#### Application.yaml of spring cloud


```
server:  
  port: 8071  
  
spring:  
  rabbitmq:  
    host: "localhost"  
    port: 5672  
    username: "guest"  
    password: "guest"  
  application:  
    name: ConfigServer  
  profiles:  
    active: git  
  cloud:  
    bus:  
      enabled: true  
    config:  
      server:  
        git:  
          uri: https://github.com/aneeshps97/configurations.git  
          default-label: main  
          timeout: 5  
          clone-on-start: true  
          force-pull: true  
  
encrypt:  
  key: "thisistheencryptionkeywhichweareusing"

```

#### pom.xml of spring cloud


```
<?xml version="1.0" encoding="UTF-8"?>  
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"  
    xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 https://maven.apache.org/xsd/maven-4.0.0.xsd">  
    <modelVersion>4.0.0</modelVersion>  
    <parent>       <groupId>org.springframework.boot</groupId>  
       <artifactId>spring-boot-starter-parent</artifactId>  
       <version>3.2.5</version>  
       <relativePath/> <!-- lookup parent from repository -->  
    </parent>  
    <groupId>com.example</groupId>  
    <artifactId>springcloud</artifactId>  
    <version>0.0.1-SNAPSHOT</version>  
    <name/>    <description/>    <url/>    <licenses>       <license/>    </licenses>    <developers>       <developer/>    </developers>    <scm>       <connection/>       <developerConnection/>       <tag/>       <url/>    </scm>    <properties>       <java.version>17</java.version>  
       <spring-cloud.version>2023.0.1</spring-cloud.version>  
    </properties>    <dependencies>       <dependency>          <groupId>org.springframework.boot</groupId>  
          <artifactId>spring-boot-starter-actuator</artifactId>  
       </dependency>       <dependency>          <groupId>org.springframework.cloud</groupId>  
          <artifactId>spring-cloud-config-server</artifactId>  
       </dependency>  
       <dependency>          <groupId>org.springframework.cloud</groupId>  
          <artifactId>spring-cloud-starter-bus-amqp</artifactId>  
       </dependency>  
       <dependency>          <groupId>org.springframework.boot</groupId>  
          <artifactId>spring-boot-starter-test</artifactId>  
          <scope>test</scope>  
       </dependency>    </dependencies>    <dependencyManagement>       <dependencies>          <dependency>             <groupId>org.springframework.cloud</groupId>  
             <artifactId>spring-cloud-dependencies</artifactId>  
             <version>${spring-cloud.version}</version>  
             <type>pom</type>  
             <scope>import</scope>  
          </dependency>       </dependencies>    </dependencyManagement>  
    <build>       <plugins>          <plugin>             <groupId>org.springframework.boot</groupId>  
             <artifactId>spring-boot-maven-plugin</artifactId>  
          </plugin>       </plugins>    </build>  
</project>
```


### Application.yaml of accounts microservice

```
server:  
  address: 0.0.0.0  
  servlet:  
    context-path: /accountService  
  port: '8081'  
spring:  
  cloud:  
    bus:  
      enabled: true  
  amqp:  
    rabbit:  
      auto-declare: true  
  rabbitmq:  
    host: "localhost"  
    port: 5672  
    username: "guest"  
    password: "guest"  
  config:  
    import: "optional:configserver:http://localhost:8071/"  
  datasource:  
    driver-class-name: com.mysql.cj.jdbc.Driver  
    username: root  
    url: jdbc:mysql://localhost:3306/accountservicedb  
    password: Psaneesh010@  
  application:  
    name: accounts  
  main:  
    banner-mode: 'off'  
  jpa:  
    database-platform: org.hibernate.dialect.MySQLDialect  
    hibernate:  
      ddl-auto: update  
    show-sql: 'true'  
  profiles:  
    active:  
      - "dev"  
management:  
  endpoints:  
    web:  
      exposure:  
        include: "*"  
  
  
build:  
  version: "2.0 mainapplication"  
  
env:  
  environment: "none"  
  
contact-info:  
  message: contact info for account microservice  
  contact-details:  
    email: psaneesh010@gmail.com  
    phone: 7907736835  
  onCall-support:  
    +91 9656932559  
    +91 9544732807
```

### pom.xml of account microservice

```
<?xml version="1.0" encoding="UTF-8"?>  
<project xmlns="http://maven.apache.org/POM/4.0.0"  
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"  
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0  
         https://maven.apache.org/xsd/maven-4.0.0.xsd">  
  
    <modelVersion>4.0.0</modelVersion>  
  
    <parent>       <groupId>org.springframework.boot</groupId>  
       <artifactId>spring-boot-starter-parent</artifactId>  
       <version>3.2.5</version>  
    </parent>  
    <groupId>com.example</groupId>  
    <artifactId>AccountService</artifactId>  
    <version>0.0.1-SNAPSHOT</version>  
    <packaging>jar</packaging>  
  
    <properties>       <java.version>17</java.version>  
       <maven.compiler.release>17</maven.compiler.release>  
       <lombok.version>1.18.30</lombok.version>  
       <spring-cloud.version>2023.0.1</spring-cloud.version>  
    </properties>  
    <dependencies>  
       <dependency>          <groupId>org.springframework.boot</groupId>  
          <artifactId>spring-boot-starter-actuator</artifactId>  
       </dependency>  
       <dependency>          <groupId>org.springframework.boot</groupId>  
          <artifactId>spring-boot-starter-web</artifactId>  
       </dependency>  
       <dependency>          <groupId>org.springframework.boot</groupId>  
          <artifactId>spring-boot-starter-validation</artifactId>  
       </dependency>  
       <dependency>          <groupId>org.springframework.boot</groupId>  
          <artifactId>spring-boot-starter-data-jpa</artifactId>  
       </dependency>  
       <dependency>          <groupId>com.mysql</groupId>  
          <artifactId>mysql-connector-j</artifactId>  
          <scope>runtime</scope>  
       </dependency>  
       <dependency>          <groupId>org.projectlombok</groupId>  
          <artifactId>lombok</artifactId>  
          <version>1.18.30</version>  
          <optional>true</optional>  
       </dependency>  
       <dependency>          <groupId>org.springdoc</groupId>  
          <artifactId>springdoc-openapi-starter-webmvc-ui</artifactId>  
          <version>2.3.0</version>  
       </dependency>  
       <dependency>          <groupId>org.springframework.boot</groupId>  
          <artifactId>spring-boot-starter-test</artifactId>  
          <scope>test</scope>  
       </dependency>  
       <dependency>          <groupId>org.jetbrains</groupId>  
          <artifactId>annotations</artifactId>  
          <version>13.0</version>  
          <scope>compile</scope>  
       </dependency>       <dependency>          <groupId>org.springframework.cloud</groupId>  
          <artifactId>spring-cloud-starter-config</artifactId>  
       </dependency>  
       <dependency>          <groupId>org.springframework.cloud</groupId>  
          <artifactId>spring-cloud-starter-bus-amqp</artifactId>  
       </dependency>  
    </dependencies>  
    <dependencyManagement>       <dependencies>          <dependency>             <groupId>org.springframework.cloud</groupId>  
             <artifactId>spring-cloud-dependencies</artifactId>  
             <version>${spring-cloud.version}</version>  
             <type>pom</type>  
             <scope>import</scope>  
          </dependency>       </dependencies>    </dependencyManagement>  
  
    <build>       <plugins>          <plugin>             <groupId>org.apache.maven.plugins</groupId>  
             <artifactId>maven-compiler-plugin</artifactId>  
             <version>3.11.0</version>  
             <configuration>                <annotationProcessorPaths>                   <path>                      <groupId>org.projectlombok</groupId>  
                      <artifactId>lombok</artifactId>  
                      <version>${lombok.version}</version>  
                   </path>                </annotationProcessorPaths>             </configuration>          </plugin>  
          <plugin>             <groupId>org.springframework.boot</groupId>  
             <artifactId>spring-boot-maven-plugin</artifactId>  
             <configuration>                <excludes>                   <exclude>                      <groupId>org.projectlombok</groupId>  
                      <artifactId>lombok</artifactId>  
                   </exclude>                </excludes>             </configuration>          </plugin>       </plugins>    </build>  
</project>
```


### steps to configure the spring cloud bus


Spring Cloud Bus, available at https://spring.io/projects/spring-cloud-bus, facilitates seamless communication between all connected application instances by establishing a convenient event broadcasting channel. It offers an implementation for AMQP brokers, such as RabbitMQ, and Kafka, enabling efficient communication across the application ecosystem.

Below are the steps to follow,



Add actuator dependency in the Config Server & Client services: Add Spring Boot Actuator dependency inside pom.xml of the individual microservices like accounts, loans and cards to expose the /busrefresh endpoint

Enable / busrefresh API: The Spring Boot Actuator library provides a configuration endpoint called "/actuator/busrefresh" that can trigger a refresh event. By default, this endpoint is not exposed, so you need to explicitly enable it in the application.yml File using the below config, management: endpoints: web:

exposure:



include: busrefresh

Add Spring Cloud Bus dependency in the Config Server & Client services: Add Spring Cloud Bus dependency (spring-cloud-starter-bus-amap) inside pom.xml of the individual microservices like accounts, loans, cards and Config server



Set up a RabbitMQ: Using Docker, setup RabbitMQ service. If the service is not started with default values, then configure the rabbitmq connection details in the application.yml file of all the individual microservices and Config


# How to setup the spring cloud config monitor

Till now we need to manually trigger any of the actuator of any microservice , but we can automate this using the spring cloud config monitor .  This spring cloud config monitor helps us to achieve that . we can add a webhook inside the github repo and whenever a change happens it will trigger a url name/monitor and this will do this configuration change reflection automatically. 

Steps 

1. first we need to add a dependency in the config server microservice 
	  
```
<dependency>  
    <groupId>org.springframework.cloud</groupId>  
    <artifactId>spring-cloud-config-monitor</artifactId>  
</dependency>

```

and we need this in application.yaml file of config server 

```
management:  
  endpoints:  
    web:  
      exposure:  
        include: "*"
```

with this enabled we can configure a webhook inside the github repo and this will trigger this actuator config monitor and refresh the config server

go to the github repo and create a webhook there 

under the settings tab we can see webhooks [[web hooks.png]]  this should be our server path to trigger the refresh url but right now we are using local machine so we need to use an intermediate website to trigger this. 
[[configure webhook.png]]
Here the payload url is where we need to trigger the webhook to , ie if any change happened we can trigger a request to the given url. 

because we are working in local host we can create a link using a website  use website hookdeck.com [https://hookdeck.com/](https://hookdeck.com/)  create a new link.  It will give you something like https://hkdk.events/2k1t1cilmq1luj. as a link use this inside the github 

and go to the cli page and it will show you how to create a hook [[hookdeck cli.png]]



and on the cli use 

hookdeck listen 8071 test-source --path /monitor

and whenever we make a change in the github it will reflect in the configuration changes. 

