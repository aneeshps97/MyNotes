When we build an application using docker what it does in the background is it packs all the necessary dependencies like java , python or anything like that which is required for the application to run. This package is called an image. 
And a running image is called a container. To this container we can communicate using the port it exposed to the outside world.

## How to build a docker image

There are 3 main approaches to build a docker image
1. docker file
2. build packs
3. Google jib

### Using docker file

Steps 
1. Update the pom.xml
		In the pom.xml after versiontag we need to provide the packaging as jar 

```
<groupId>com.example</groupId>  
<artifactId>AccountService</artifactId>  
<version>0.0.1-SNAPSHOT</version>  
<packaging>jar</packaging>
```

And if we do 

```
aneesh@Aneeshs-MacBook-Air accountService % mvn clean install 
[INFO] Scanning for projects...
[INFO] 
[INFO] ---------------------< com.example:AccountService >---------------------
[INFO] Building AccountService 0.0.1-SNAPSHOT
```

it will generate a jar file in the target folder in the name

*AccountService-0.0.1-SNAPSHOT.jar*

and using java command we can run this 

java -jar target/AccountService-0.0.1-SNAPSHOT.jar

#### we need to create a docker file to make an image out of this 


```
FROM eclipse-temurin:17-jdk  
COPY target/AccountService-0.0.1-SNAPSHOT.jar AccountService-0.0.1-SNAPSHOT.jar  
ENTRYPOINT ["java","-jar","AccountService-0.0.1-SNAPSHOT.jar"]
```

FROM - This indicates based on which image it is getting build, like java, python cpp things like that 

MAINTAINER - to let the people know who ows it 

COPY - from where we need to copy the file to where we need to paste it , here we are copying the file from the target folder and pasting it outside

ENTRYPOINT - this is the command which we use to run to make the executable file jar or war, if the command is like this 
java -jar AccountService-0.0.1-SNAPSHOT.jar we need to give each of the sub components as a string array. 

### How to build the docker image using docker file


```
aneesh@Aneeshs-MacBook-Air accountService % docker build -t accounttest:1.0 .
```

here. accounttest is the name and 1.0 is the version and . means the docker file is in the same folder where we are trying to run this command

if the docker file is in another folder

```
docker build -t accounttest:1.0 -f docker/Dockerfile .
```

it will create an image like this 

```
IMAGE                           ID             DISK USAGE   CONTENT SIZE   EXTRA
accounttest:1.0                 c906187ab338        810MB          258MB        
```

### How to run the docker container 


```
docker run -p 8080:8080 <docker image name> -d 
```

```
aneesh@Aneeshs-MacBook-Air accountService % docker run -p 8080:8080 accounttest:1.0
```

here there are two ports which we give using the -p tag  


```
docker run -p 8080:8080 accounttest:1.0
```

the two ports have different meanings:

```
-p <host-port>:<container-port>
```

So:

```
-p 8080:8080
```

means:

```
Host (your Mac)     -> 8080Container (Docker)  -> 8080
```

Requests to:

```
http://localhost:8080
```

on your Mac are forwarded to port `8080` inside the container.

---

For example, if your Spring Boot app runs on port `8080` inside the container:

```
server.port=8080
```

you could also do:

```
docker run -p 9090:8080 accounttest:1.0
```

which means:

```
Mac port 9090  -> Container port 8080
```

Then you'd access the application at:

```
http://localhost:9090
```

even though the application itself is still listening on `8080` inside the container.

##### also when we build using this docker we need to make sure that we are providing the mysql db url like this 

```
username: root  
url: jdbc:mysql://host.docker.internal:3306/accountservicedb  
password: Psaneesh010@
```

here the db can't connect to the local host because of that we need to give like this <host.docker.internal>

in the container perspective the localhost means the container itself and host.docker.internal means the host machine in which the docker container is running that is our machine . 

# How to create a docker image using google jib

go to the url 

[https://github.com/googleContainerTools/jib](https://github.com/googleContainerTools/jib) and we can find the instructions on how to create an image using google jib
(google jib only works on java)

essentially what we need to do is we need to copy and paste this plugin to our pom.xml 

```
<project>
  ...
  <build>
    <plugins>
      ...
      <plugin>
        <groupId>com.google.cloud.tools</groupId>
        <artifactId>jib-maven-plugin</artifactId>
        <version>3.5.1</version>
        <configuration>
          <to>
            <image>myimage</image>
          </to>
        </configuration>
      </plugin>
      ...
    </plugins>
  </build>
  ...
</project>
```

here we can give the image name as 
```
<to>  
    <image>aneeshps010/${project.artifactId}:s4</image>  
</to>
```

and the commad which we use to create an image using google jib is 

```
mvn compile jib:dockerBuild 
```

we need to build that using 

```
mvn compile jib:dockerBuild -Djib.from.platforms=linux/arm64
```

otherwise the docker compse will not work properly

and then we can run this using the same way as shown above. 


```
docker run -p 9090:8080 accounttest:1.0
```

