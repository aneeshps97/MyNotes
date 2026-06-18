we can perform the data validation of inputs even before they enter in to the controller class . This can be done at the dto objects which we pass to the controller or the path variable or request param which we pass to the controller. 

#### How to do an input data validation

we need to add the dependency in pom.xml

```
<dependency>  
    <groupId>org.springframework.boot</groupId>  
    <artifactId>spring-boot-starter-validation</artifactId>  
</dependency>
```


Then in the dto class we can add validations to each of the variables


```
@Data  
@AllArgsConstructor  
@NoArgsConstructor  
public class SignInRequestDTO {  
    @NotBlank  
    String username;  
    @NotBlank String password;  
}
```

like this common validations include 

## Common Validation Annotations

| Annotation        | Purpose                              |
| ----------------- | ------------------------------------ |
| `@NotNull`        | Cannot be null                       |
| `@NotBlank`       | Cannot be null, empty, or whitespace |
| `@NotEmpty`       | Cannot be null or empty              |
| `@Email`          | Valid email format                   |
| `@Size`           | String/Collection size limit         |
| `@Min`            | Minimum numeric value                |
| `@Max`            | Maximum numeric value                |
| `@Positive`       | Must be > 0                          |
| `@PositiveOrZero` | Must be >= 0                         |
| `@Pattern`        | Regex validation                     |
| `@Past`           | Date must be in the past             |
| `@Future`         | Date must be in the future           |

##### Example for a password validation

```
@NotBlank @Pattern(  
        regexp = "^(?=.*[a-z])(?=.*[A-Z])(?=.*\\d)(?=.*[@#$%^&+=!]).{8,20}$",  
        message = "Password must contain uppercase, lowercase, digit and special character"  
)  
private String password;
```

example of a dto class using validation 

```
@Data  
@NoArgsConstructor  
@AllArgsConstructor  
@Schema(description = "Data Transfer Object for User profiles")  
public class UserDTO {  
    private Long userId;  
  
    @NotBlank  
    private String userName;  
  
    @NotBlank  
    private String firstName;  
  
    @NotBlank  
    private String lastName;  
  
    @NotBlank  
    private String phoneNumber;  
  
    @NotBlank @Pattern(  
            regexp = "^(?=.*[a-z])(?=.*[A-Z])(?=.*\\d)(?=.*[@#$%^&+=!]).{8,20}$",  
            message = "Password must contain uppercase, lowercase, digit and special character"  
    )  
    private String password;  
  
    private String userStatus;  
}
```
The controller class which we use must be annotated with 

```
@Validated
```

```
@Validated  
public class UserController {
```

and when we pass this dto we can use @Valid annotation to check whether the data passed is valid or not 

```
@PostMapping("/users")  
public ResponseEntity<ApiResponseDTO<UserDTO>> signUp(@Valid UserDTO userDTO){  
    log.info("Request received signing up the user::{}", userDTO);  
    UserDTO userDto = userService.signUp(userDTO);  
    //boolean success, int status, ResponseCode responseCode,T data  
    return ResponseEntity.ok(ApiResponseDTO.generateResponse(userDto, ResponseCode.USER_SIGN_UP_SUCCESS, true));  
}
```

and in order for this exception to process we need to annotate the method with 

```
@ExceptionHandler(MethodArgumentNotValidException.class)
```



```
@ExceptionHandler(MethodArgumentNotValidException.class)
protected ResponseEntity<Object> handleArgumentsNotValidException(MethodArgumentNotValidException e){  
  Map<String,String> validationErrors = new HashMap<>();  
  List<FieldError> fieldErrors = e.getBindingResult().getFieldErrors();  
  for (FieldError fieldError : fieldErrors) {  
    validationErrors.put(fieldError.getField(),fieldError.getDefaultMessage());  
  }  
  return new ResponseEntity<>(validationErrors,HttpStatus.BAD_REQUEST);  
}
```

Spring provides default handlers for many framework exceptions:

```
MethodArgumentNotValidException
HttpMessageNotReadableException
MissingServletRequestParameterException
MethodArgumentTypeMismatchException...
```

we don't actually need to extend the ResponseEntityExceptionHandler class but if we want to change the default behaviour of the inbuit classes we need to extend it and override the method needed.