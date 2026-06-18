
website to convert application.properties to application.yaml
[https://mageddo.com/tools/yaml-converter](https://mageddo.com/tools/yaml-converter)

*how to switch this at the run time*
[[Externalizing configuration]]


we can create multiple set of configurations each for a specific environment and by switching this we can set the application to take that set of value

here we need to create multiple applications.yaml file for each profiles

applications-prod.yaml
applications-dev.yaml

[[applications.yaml creation.png]]

And in the applications.yaml file we need to give 

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

and in the applications-dev.yaml we need to give 

```
spring:  
  config:  
    activate:  
      onprofile: "dev"
```


#### Sample application.yaml

```
config:  
  import:  
    - "application_dev.yaml"  
    - "application_qa.yaml"  
  
server:  
  address: 0.0.0.0  
  servlet:  
    context-path: /bookingService  
  port: '8082'  
spring:  
  datasource:  
    driver-class-name: com.mysql.cj.jdbc.Driver  
    username: root  
    url: jdbc:mysql://localhost:3306/accountservicedb  
    password: Psaneesh010@  
  application:  
    name: BookingService  
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

#### sample application_dev.yaml

```
build:  
  version: "1.1"  
  
env:  
  environment: "dev"  
  
contact-info:  
  message: contact info for account microservice  
  contact-details:  
    email: psaneesh010@gmail.com  
    phone: 7907736835  
  onCall-support:  
    +91 9656932559  
    +91 9544732807
```



