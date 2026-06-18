It is used to start all our containers together with the use of a yaml file 

for this first we need to verify whether we installed docker compose 

for this we need to run the command 

```
docker compose version
```

```
aneesh@Aneeshs-MacBook-Air accountService % docker compose version
Docker Compose version v5.1.3
```

and for using docker compose we need to create a docker compose file  

1. create a file named  compose.yml  file on the application directory 
2. and create a tag named services:   inside this tag we can specify all our images which we need to run 

```
services:  
  accounts:  
    image: aneeshps010/accounts:s4  
    container_name: accounts-ms  
    ports:  
      - "8081:8081"  
    deploy:  
      resources:  
        limits:  
          memory: 700m  
    networks:  
      - abc
```

this is the example of one service.  here we can specify all the things which we use when we run a docker image 

how to create multiple images at once

```
services:  
  accounts:  
    image: aneeshps010/accounts:s4  
    container_name: accounts-ms  
    ports:  
      - "8081:8081"  
    deploy:  
      resources:  
        limits:  
          memory: 700m  
    networks:  
      - abc  
  booking:  
    image: aneeshps010/bookingservice:s4  
    container_name: booking-ms  
    ports:  
      - "8083:8083"  
    deploy:  
      resources:  
        limits:  
          memory: 700m  
    networks:  
      - abc  
  space:  
    image: aneeshps010/spaceservice:s4  
    container_name: space-ms  
    ports:  
      - "8082:8082"  
    deploy:  
      resources:  
        limits:  
          memory: 700m  
    networks:  
      - abc  
networks:  
  abc:  
    driver: "bridge"
```

here we need to give networks:  networkname. because each container will be running its own isolated space and we need a way to connect all of them together that's why we are using this. 

we need to give 

```
    networks:  
      - abc 
```

and then we need to create network for that 

```
networks:  
  abc:  
    driver: "bridge"
```

### What is a bridge network?

A bridge network is Docker's default networking mode for containers on a single host.

use http://space:8080. from the accounts docker container it will automatically calls whatever we need inside the space service because all of them are running under the same network now.

and use command 

```
docker compose up 
```

to make all the containers

```
docker compose stop
```

will stop all the running containers 

and 

```
docker compose start 
```

will restart them again 

```
docker compose down 
```

will remove all the running containers build using compose