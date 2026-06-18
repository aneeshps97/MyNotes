
### How to externalise configuration using ide

####   By providing command line arguments using ide

select main application and on the dropdown choose modify run configuration and pass the program arguments as 
[[Edit run configurations.png]]
[[add run configurations.png]]

```
--spring.profiles.active = "prod" --build.version = 1.1
```

#### Modifying the jvm system variables

In the same modify page click on modify options and click on add vm options and paste same options there 

[[add vm options.png]]
[[vm options added.png]]

```
-Dspring.profiles.active="prod" -Dbuild.version=1.2
```

   *Here the jvm system variables have the highest precedence* 



#### Editing the Environment variables

[[Editing environment variables.png]]
In the same page we can edit the environment variables also , but the precedence will be low 

```
SPRING_PROFILES_ACTIVE=dev;BUILD_VERSION=1.8
```

when both the command line arguments and environment variables are there the system will take the command line arguments, 
to get the result remove the command line arguments and run.

*This will resolve the immutability issue to some extend and many organizations are still using it
