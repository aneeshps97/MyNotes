This is what converts dto s to entity and vice versa. we need springboot version higher than 4.0 to work with this

```
//uses = {VehicleMapper.class} add this in the mapper if it needed  
@Mapper(componentModel = "spring")  
public interface  UserMapper {  
    UserDTO toDto(User user);  
    User toEntity(UserDTO userDto);  
    @BeanMapping(  
            nullValuePropertyMappingStrategy =  
                    NullValuePropertyMappingStrategy.IGNORE  
    )  
    void updateUserFromDto(UserUpdateDTO userUpdateDTO,@MappingTarget UserDTO userDTO);  
  
}
```

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

```
package com.example.rhino.entity;  
  
import jakarta.persistence.*;  
import lombok.Data;  
import jakarta.persistence.Id;  
  
@Data  
@Entity  
@Table(name = "user")  
public class User {  
    @Id  
    @GeneratedValue(strategy = GenerationType.IDENTITY)  
    private Long userId;  
    @Column(unique = true)  
    private String userName;  
    @Column  
    private String firstName;  
    @Column  
    private String lastName;  
    @Column  
    private String phoneNumber;  
    @Column  
    private String password;  
    @Column  
    private String userStatus;  
}

```



add the mapper dependency and you are good to go 

```
<properties>  
    <java.version>17</java.version>  
    <org.mapstruct.version>1.6.0.Beta1</org.mapstruct.version>  
    <lombok.version>1.18.30</lombok.version>  
</properties>
```

```
<dependency>  
    <groupId>org.mapstruct</groupId>  
    <artifactId>mapstruct</artifactId>  
    <version>${org.mapstruct.version}</version>  
</dependency>

<dependency>  
    <groupId>org.projectlombok</groupId>  
    <artifactId>lombok-mapstruct-binding</artifactId>  
    <version>0.2.0</version>  
    <scope>provided</scope>  
</dependency>

```
