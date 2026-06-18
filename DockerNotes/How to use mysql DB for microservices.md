we can run an instance of mysql inside our local using docker and can have separate db for each microservices. 

inside the cmd type this to start a container of mysql db 

```
aneesh@Aneeshs-MacBook-Air section7 % docker run -p 3306:3306 --name accountservicedb -e MYSQL_ROOT_PASSWORD=root -e MYSQL_DATABASE=accountsdb -d mysql

```

```
aneesh@Aneeshs-MacBook-Air section7 % docker run -p 3307:3306 --name accountservicedb_docker -e MYSQL_ROOT_PASSWORD=root -e MYSQL_DATABASE=accountsdb -d mysql
2966720e69764ff71f3403631af6091b20bcc3b832031ff3c064d2bfa7f8ac6d
```

Here i used port 3307 because i already had an instance of mysql running on my local with the port 3307. so docker can't use that , but the docker host port can be 3306 because that is running on it's own isolated space. 

This will create a container in the name accountsdb , inorder to connect to this database we need a mysql client like toad. a simple client is SQLECTRON with which we can connect to any db. 

download and install it from [https://sqlectron.github.io](https://sqlectron.github.io)   

after installation on the server tab click on add and add details of the connection and create a new connection and after the creation  [[db connection.png]] can see the db created inside that 

In this way we can create db for each microservices  just we need to change the name and the local exposed portnumber

and we need to the database name will be  accountsdb  and the docker container name will be accountservicedb_docker.

we need to add these dependencies to the pom.xml in order for this to work

```
<dependency>  
    <groupId>com.mysql</groupId>  
    <artifactId>mysql-connector-j</artifactId>  
    <scope>runtime</scope>  
</dependency>
```

and the connection details in the application.yml file 

```
datasource:  
  driver-class-name: com.mysql.cj.jdbc.Driver  
  username: root  
  url: jdbc:mysql://localhost:3307/accountsdb  
  password: root
```

### How to work with docker compose using mysql db

if we are using docker compose the localhost mysql db will not work , for that we need to make some changes in the docker compose file 

we need to provide the details of each db service we create inside the docker compose file 

```
accountsdb:  
  image: mysql  
  container_name: accountservicedb_docker  
  ports:  
    - "3307:3306"  
  healthcheck:  
    test: ["CMD","mysqladmin","ping","-h","localhost"]  
    timeout: 10s  
    retries: 10  
    interval: 10s  
    start_period: 10s  
  environment:  
    MYSQL_ROOT_PASSWORD: root  
    MYSQL_DATABASE: accountsdb  
  extends:  
    file: common-config.yml  
    service: network-deploy-services
```

```
network-deploy-services:  
  networks:  
    - abc
```

in the common-config.yml   i have created a service in the name network-deploy-services:

in each service we need to give its corresponding db details 


```
depends_on:  
  configserver:  
    condition: service_healthy  
  accountsdb:  
    condition: service_healthy
```

```
environment:  
  SPRING_CONFIG_IMPORT: "configserver:http://configserver:8071/"  
  SPRING_PROFILES_ACTIVE: default  
  SPRING_APPLICATION_NAME: accounts  
  SPRING_DATASOURCE_URL: "jdbc:mysql://accountsdb:3306/accountsdb"  
  SPRING_DATASOURCE_USERNAME: "root"
```

here we can give the same portnumber 3306 for all the services because the db is running on the isolated network everytime so we can give the port as 3306 for every container .   the 3307 which we give for the db.   ports:  - "3307:3306"    this is for external communication an external application communicate to the container but the accounts is a service which is running on the internal network so we can use same port for all the dbs