# Important Docker Commands

## Docker Image Commands


* **01. `docker images`** – To list all the docker images present in the Docker server.
```
aneesh@Aneeshs-MacBook-Air accountService % docker images
                                                                                                                                   i Info →   U  In Use
IMAGE                         ID             DISK USAGE   CONTENT SIZE   EXTRA
aneesh010/accountservice:01   df11160c5007        810MB          258MB 
```
	  
* **02. `docker image inspect [image-id]`** – To display detailed image information for a given image id.

```
aneesh@Aneeshs-MacBook-Air accountService % docker image inspect df11160 
[
    {
        "Id": "sha256:df11160c50073faa97cf103f3b991dd3a33556c39d6af49240d2e519529fd6b4",
        "RepoTags": [
            "aneesh010/accountservice:01"

```
* **03. `docker image rm [image-id]`** – To remove one or more images for a given image ids.

```

aneesh@Aneeshs-MacBook-Air accountService % docker image rm df11160 
Untagged: aneesh010/accountservice:01
Deleted: sha256:df11160c50073faa97cf103f3b991dd3a33556c39d6af49240d2e519529fd6b4
```

* **04. `docker build . -t [image-name]`** – To generate a docker image based on a Dockerfile.

```
aneesh@Aneeshs-MacBook-Air accountService % docker build . -t aneeshps010/accountservice:01
[+] Building 2.5s (8/8) FINISHED                                                                                                  docker:desktop-linux
 => [internal] load build definition from Dockerfile   

```

* **05. `docker image push [container_registry/username:tag]`** – To push an image from a container registry.'
       we need to login first inorder to push that image

```
aneesh@Aneeshs-MacBook-Air accountService % docker login
Authenticating with existing credentials... [Username: aneeshps010]

i Info → To login with a different account, run 'docker logout' followed by 'docker login'


Login Succeeded


aneesh@Aneeshs-MacBook-Air accountService % docker image push aneeshps010/accountservice:01
The push refers to repository [docker.io/aneeshps010/accountservice]


```

* **06. `docker image pull [container_registry/username:tag]`** – To pull an image from a container registry.


```
aneesh@Aneeshs-MacBook-Air accountService % docker image pull aneeshps010/accountservice:01
01: Pulling from aneeshps010/accountservice
45b457ca1211: Pull complete 
9e4f9d1978e6: Download complete 
Digest: sha256:798eadd141681b103c060852fec23568d2c8afb7da9b88755f8f576e75e1c190
Status: Downloaded newer image for aneeshps010/accountservice:01
docker.io/aneeshps010/accountservice:01
```

* **07. `docker image prune`** – To remove all unused images.
		**What does** `docker prune` **actually do?**  
		The primary purpose of the `prune` command is to reclaim disk space by removing unused objects. Docker doesn't automatically delete old layers or stopped containers because it assumes you might want them back.

```
aneesh@Aneeshs-MacBook-Air accountService % docker image prune
WARNING! This will remove all dangling images.
Are you sure you want to continue? [y/N] y
Total reclaimed space: 0B
```

* **08. `docker rmi [image-id]`** – To remove one or more images based on image ids.


```
aneesh@Aneeshs-MacBook-Air accountService % docker rmi 798eadd
Untagged: aneeshps010/accountservice:01
Deleted: sha256:798eadd141681b103c060852fec23568d2c8afb7da9b88755f8f576e75e1c190

```

* **09. `docker history [image-name]`** – Displays the intermediate layers and commands that were executed when building the image.


```
neesh@Aneeshs-MacBook-Air accountService % docker history 798eadd
IMAGE          CREATED        CREATED BY                                      SIZE      COMMENT
798eadd14168   40 hours ago   ENTRYPOINT ["java" "-jar" "AccountService-0.…   0B        buildkit.dockerfile.v0
<missing>      40 hours ago   COPY target/AccountService-0.0.1-SNAPSHOT.ja…   54MB      buildkit.dockerfile.v0
<missing>      9 days ago     CMD ["jshell"]                                  0B        buildkit.dockerfile.v0

```

## Container Lifecycle & Management

* **10. `docker run -p [hostport]:[containerport] [image_name]`** – To start a docker container based on a given image.

```
aneesh@Aneeshs-MacBook-Air accountService % docker run -d -p 8085:8081 --name accountservicecontainer 798eadd14168
c34ad728081e0785929801e5442b5fc1e70734d3a428f57f7be27bb81df21a29

```

* **11. `docker ps`** – To show all running containers.

```
aneesh@Aneeshs-MacBook-Air accountService % docker ps
CONTAINER ID   IMAGE          COMMAND                  CREATED          STATUS          PORTS                                         NAMES
c34ad728081e   798eadd14168   "java -jar AccountSe…"   22 minutes ago   Up 22 minutes   0.0.0.0:8085->8081/tcp, [::]:8085->8081/tcp   accountservicecontainer

```
* **12. `docker ps -a`** – To show all containers including running and stopped.

```
aneesh@Aneeshs-MacBook-Air accountService % docker stop c34ad728
c34ad728
aneesh@Aneeshs-MacBook-Air accountService % docker ps                  
CONTAINER ID   IMAGE     COMMAND   CREATED   STATUS    PORTS     NAMES
aneesh@Aneeshs-MacBook-Air accountService % docker ps -a
CONTAINER ID   IMAGE          COMMAND                  CREATED          STATUS                        PORTS     NAMES
c34ad728081e   798eadd14168   "java -jar AccountSe…"   23 minutes ago   Exited (143) 16 seconds ago             accountservicecontainer

```

* **13. `docker container start [container-id]`** – To start one or more stopped containers.

```
aneesh@Aneeshs-MacBook-Air accountService % docker container start c34ad72
c34ad72
aneesh@Aneeshs-MacBook-Air accountService % docker ps
CONTAINER ID   IMAGE          COMMAND                  CREATED          STATUS         PORTS                                         NAMES
c34ad728081e   798eadd14168   "java -jar AccountSe…"   24 minutes ago   Up 3 seconds   0.0.0.0:8085->8081/tcp, [::]:8085->8081/tcp   accountservicecontainer

```

* **14. `docker container pause [container-id]`** – To pause all processes within one or more containers.

      when it is paused we can see that in the postman the request is paused!
    [[DockerNotes/images/docker paused.png]]
      
```
aneesh@Aneeshs-MacBook-Air accountService % docker container pause c34ad72
c34ad72

```

* **15. `docker container unpause [container-id]`** – To resume/unpause all processes within one or more containers.
		only after unpause the request will resume processing and produce result

	[[DockerNotes/images/docker paused.png]]

```
aneesh@Aneeshs-MacBook-Air accountService % docker container unpause c34ad72
c34ad72

```

* **16. `docker container stop [container-id]`** – To stop one or more running containers.
		This will provide the docker with a grace period to shut down its processes and save all the data by default 10 sec

```
aneesh@Aneeshs-MacBook-Air accountService % docker container stop c34ad72
c34ad72
```

* **17. `docker container kill [container-id]`** – To kill one or more running containers instantly.

		This will stop the container immidietly without waiting for anything.

```
aneesh@Aneeshs-MacBook-Air accountService % docker kill c34ad72
c34ad72

```

* **18. `docker container restart [container-id]`** – To restart one or more containers.

```
aneesh@Aneeshs-MacBook-Air accountService % docker restart c34ad72
c34ad72

```


* **19. `docker container inspect [container-id]`** – To inspect all the details for a given container id.

```
aneesh@Aneeshs-MacBook-Air accountService % docker container inspect c34ad72
[
    {
        "Id": "c34ad728081e0785929801e5442b5fc1e70734d3a428f57f7be27bb81df21a29",
        "Created": "2026-05-17T14:28:29.070354802Z",
        "Path": "java",
```

* **20. `docker rm [container-id]`** – To remove one or more containers based on container ids.

```
aneesh@Aneeshs-MacBook-Air accountService % docker ps -a
CONTAINER ID   IMAGE          COMMAND                  CREATED       STATUS                     PORTS     NAMES
c34ad728081e   798eadd14168   "java -jar AccountSe…"   8 hours ago   Exited (137) 7 hours ago             accountservicecontainer

aneesh@Aneeshs-MacBook-Air accountService % docker rm c34ad72
c34ad72

aneesh@Aneeshs-MacBook-Air accountService % docker ps -a
CONTAINER ID   IMAGE     COMMAND   CREATED   STATUS    PORTS     NAMES

```

* **21. `docker container prune`** – To remove all stopped containers.

```
aneesh@Aneeshs-MacBook-Air section 6 % docker container prune
WARNING! This will remove all stopped containers.
Are you sure you want to continue? [y/N] y
Total reclaimed space: 0B

```


## Logs, Stats & Interaction
* **22. `docker container logs [container-id]`** – To fetch the logs of a given container id.

```
aneesh@Aneeshs-MacBook-Air section 6 % docker container logs 1dcfcef
2026-05-17T22:02:57.735Z  INFO 1 --- [AccountService] [           main] c.e.A.AccountServiceApplication          : Starting AccountServiceApplication v0.0.1-SNAPSHOT using Java 17.0.19 with PID 1 (/AccountService-0.0.1-SNAPSHOT.jar started by root in /)
2026-05-17T22:02:57.738Z  INFO 1 --- [AccountService] [           main] c.e.A.AccountServiceApplication          : No active profile set, falling back to 1 default profile: "default"
2026-05-17T22:02:58.093Z  INFO 1 --- [AccountService] [           main] .s.d.r.c.RepositoryConfigurationDelegate : Bootstrapping Spring Data JPA repositories in DEFAULT mode.
2026-05-17T22:02:58.111Z  INFO 1 --- [AccountService] [           main] .s.d.r.c.RepositoryConfigurationDelegate : Finished Spring Data repository scan

```

* **23. `docker container logs -f [container-id]`** – To follow log output of a given container id.

```
aneesh@Aneeshs-MacBook-Air section 6 % docker container logs 86403d0e -f
2026-05-17T22:06:48.118Z  INFO 1 --- [AccountService] [           main] c.e.A.AccountServiceApplication          : Starting AccountServiceApplication v0.0.1-SNAPSHOT using Java 17.0.19 with PID 1 (/AccountService-0.0.1-SNAPSHOT.jar started by root in /)
2026-05-17T22:06:48.122Z  INFO 1 --- [AccountService] [           main] c.e.A.AccountServiceApplication          : No active profile set, falling back to 1 default profile: "default"
2026-05-17T22:06:48.423Z  INFO 1 --- [AccountService] [           main] .s.d.r.c.RepositoryConfigurationDelegate : Bootstrapping Spring Data JPA repositories in DEFAULT mode.
20

```

* **24. `docker container stats`** – To show all containers statistics like CPU, memory, I/O usage.

```
CONTAINER ID   NAME                CPU %     MEM USAGE / LIMIT    MEM %     NET I/O           BLOCK I/O        PIDS
86403d0e0511   heuristic_newton    0.11%     438.8MiB / 7.75GiB   5.53%     71.8kB / 42.3kB   1.11MB / 135kB   47
1dcfcef988cd   heuristic_keldysh   0.12%     416.3MiB / 7.75GiB   5.25%     68.3kB / 38.9kB   93.7MB / 250kB   46

```

* **25. `docker exec -it [container-id] sh`** – To open a shell inside a running container and execute commands.

```


```

## System & Registry Commands
* **26. `docker system prune`** – Remove stopped containers, dangling images, and unused networks, volumes, and cache.

```
aneesh@Aneeshs-MacBook-Air section 6 % docker system prune
WARNING! This will remove:
  - all stopped containers
  - all networks not used by at least one container
  - all dangling images
  - unused build cache

Are you sure you want to continue? [y/N] y
Deleted Containers:
1dcfcef988cd17bef01aa723e418b42b6c202a75cfa997ed1cd8d6e44f23cb2e

Deleted build cache objects:
a5gvc1gve3j5aq7bt7r074alr
apwdk221fqz3guhfl1gpya388
uwxkkdup9fqmvzlca1yeq40tx
m4h37xetvpm3fqad033tmkapg

Total reclaimed space: 54.01MB

```

* **27. `docker login -u [username]`** – To login in to docker hub container registry.

```
aneesh@Aneeshs-MacBook-Air section 6 % docker login -u aneeshps010

i Info → A Personal Access Token (PAT) can be used instead.
         To create a PAT, visit https://app.docker.com/settings

```

* **28. `docker logout`** – To logout from docker hub container registry id.

```


```
## Docker Compose
* **29. `docker compose up`** – To create and start containers based on given docker compose File.
     we need to have a compose file in this location for this command to work

```
aneesh@Aneeshs-MacBook-Air section 6 % docker compose up
no configuration file provided: not found

What's next:
    Debug this Compose error with Gordon → docker ai "help me fix this compose error"
aneesh@Aneeshs-MacBook-Air section 6 % cd accountService/
aneesh@Aneeshs-MacBook-Air accountService % docker compose up

```


* **30. `docker compose down`** – To stop and remove containers for services defined in the Compose File.


```
aneesh@Aneeshs-MacBook-Air accountService % docker compose down
	aneesh@Aneeshs-MacBook-Air accountService % 

```