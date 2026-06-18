There are three ways to read configurations 

#### using @Value annotation 

   This is used to read the values from applications.yaml file 
    
    in the file we can give values like 
```
build:  
  version: "1.0"
```

    And in the controller class or any other class where we need this value we can give
```
@Value("${build.version}")  
private String buildVersion;

```

*But when we give this if we have @AllArgsConstructor it will ask us to inject the bean type string too so we need to remove that *@AllArgsConstructor  and use @AutoWired to field inject each of the beans needed or we can use constructor injection manually


### Reading environment variables

We can read java environment variables by 
autowiring this to the class we need
```
@Autowired  
Environment environment;

```

```
@GetMapping("/javaHome")  
public ResponseEntity<String> getJavaHome() {  
    return ResponseEntity.ok(environment.getProperty("java.home"));  
}

```

### Reading configuration using @ConfigurationProperties 

To read configurations using @ConfigurationProperties we need to create a class in the dto with annotation @ConfigurationProperties

```
@ConfigurationProperties(prefix = "contact-info")  
public record ConfigurationRecordDto(String message, Map<String, String> contactDetails, List<String> onCallSupport) {  
}

```

*record is a class which is introduced in the java 17 which only has getters so the data will never get manipulated*

The arguments which we give inside that class is the same as the values which we provide in the applicatio.yaml file 

```
contact-info:  
  message: contact info for booking microservice  
  contact-details:  
    email: psaneesh010@gmail.com  
    phone: 7907736835  
  onCall-support:  
    +91 9656932559  
    +91 9544732807
```

And in the main application class we need to give the annotation 
@EnableConfigurationProperties(ConfigurationRecordDto.class)

```
@EnableJpaAuditing(auditorAwareRef = "auditAwareImpl")  
@EnableConfigurationProperties(ConfigurationRecordDto.class)  
public class BookingServiceApplication {  
  
    public static void main(String[] args) {  
       SpringApplication.run(BookingServiceApplication.class, args);  
    }  
  
}
```

And then we need to autowire this class in to the class which we need and just we can return the value
```
@Autowired  
ConfigurationRecordDto configurationRecordDto;

```

```
@Autowired  
ConfigurationRecordDto configurationRecordDto;  
  
@GetMapping("/getConfig")  
public ResponseEntity<ConfigurationRecordDto> getConfigurations(){  
    return ResponseEntity.ok(configurationRecordDto);  
}
```
