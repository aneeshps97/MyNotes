to create a bean we can use @Service annotation

and when we create a bean with @Service annotation we can create multiple implementations of the same interface and we are injecting the interface to the controller or any other required class

### using @Qualifier and @Primary annotation

create a service interface

```
public interface Service {  
    public String getData();  
}
```

and two implementations for it  and use @Qualifier to specify names for the implementations

```
@Qualifier("serviceImpl1")  
public class ServiceImpl1 implements Service {  
    @Override  
    public String getData() {  
        return "from serviceimpl1";  
    }  
}

```


```
@Qualifier("serviceImpl2")  
public class ServiceImpl2 implements Service {  
    @Override  
    public String getData() {  
        return "from serviceimpl2";  
    }  
}
```

and in the place of injection we can use 

```
private Service service;  
@Autowired  
SampleController(@Qualifier("serviceImpl1")  Service service) {  
    this.service = service;  
}
```

we can specify it using @Qualifier annotation and it will inject the one needed

and if we specify any one of the implementation class with @Primary the spring will inject that one if there is no implementation of that service is specified by the @Qualifier annotation


in the serviceImpl
```
@Qualifier("serviceImpl2")  
@Primary  
public class ServiceImpl2 implements Service {  
    @Override  
    public String getData() {  
        return "from serviceimpl2";  
    }  
}
```

in the controller

```
private Service service;  
@Autowired  
SampleController(Service service) {  
    this.service = service;  
}
```

this will take the ServiceImpl2 because there is no @Qualifier is specified

### How to switch them based on a configuration 

we can use.  ==@ConditionalOnProperty==  annotation and we can change the implementation classes based on this condition provided in the application.yml class

There are 3 arguments to this annotation name, havingValue, matchIfMissing. 
name = name of the configuration we give in the application.yml file
having value = what should be the name 
matchIfMissing = if no values present for this name in the application.yml file which one should we take 

in the same example we can take 

in each of the implementation 


```
@ConditionalOnProperty(name = "serviceClass", havingValue = "serviceImpl1", matchIfMissing = true)  
public class ServiceImpl1 implements Service {  
    @Override  
    public String getData() {  
        return "from serviceimpl1";  
    }  
}

```

```
@ConditionalOnProperty(name = "serviceClass", havingValue = "serviceImpl2")  
public class ServiceImpl2 implements Service {  
    @Override  
    public String getData() {  
        return "from serviceimpl2";  
    }  
}
```

in application.yaml file we can set 

```
serviceClass: serviceImpl1
```

and in the controller we can inject the service interface normally 

```
private Service service;  
@Autowired  
SampleController(Service service) {  
    this.service = service;  
}
```

and it will take the one configured in the application.yml file 

### By using profile

we can make the implementations specific to the profiles

by using @Profile annotation 

@Profile("prod")

we can provide the information about for which profile which implementation should get injected on top of each implementation class and we can take them 

create application-prod.yaml in the project and in the needed serviceImpl we can put annotation @Profile("prod") and inject the service interface normally 
```
@Profile("prod")  
public class ServiceImpl1 implements Service {  
    @Override  
    public String getData() {  
        return "from serviceimpl1";  
    }  
}
```

```
@Autowired  
SampleController(Service service) {  
    this.service = service;  
}
```
and activate the profile in the application.yaml file 

```
spring:  
  profile:  
    active: prod
```