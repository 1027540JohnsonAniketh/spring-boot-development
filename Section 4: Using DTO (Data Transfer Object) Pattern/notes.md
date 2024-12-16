# DTO (Data Transfer Object) Pattern in Spring Boot

## Table of Contents
1. [What is a DTO?](#what-is-a-dto)
2. [Why Use DTOs?](#why-use-dtos)
3. [DTO Pattern Implementation](#dto-pattern-implementation)
4. [Mapping Strategies](#mapping-strategies)
5. [Validation](#validation)
6. [Best Practices](#best-practices)
7. [Advanced Techniques](#advanced-techniques)

## What is a DTO?

A Data Transfer Object (DTO) is an object used to transfer data between layers of an application, typically between the client and the server. It's a lightweight object that contains only data, without any business logic.

### Key Characteristics
- Separates presentation layer from domain model
- Reduces over-fetching of data
- Provides a clean contract for API responses
- Helps in decoupling layers of the application

## Why Use DTOs?

### Advantages
- **Encapsulation**: Hide internal domain model details
- **Performance**: Transfer only necessary data
- **Flexibility**: Customize data representation
- **Security**: Control what data is exposed

### Example Scenario
Consider a User entity with sensitive information:

```java
// Domain Entity (Internal Representation)
@Entity
public class User {
    @Id
    private Long id;
    private String username;
    private String email;
    private String password;  // Sensitive information
    private Role role;
}

// DTO (External Representation)
public class UserDTO {
    private Long id;
    private String username;
    private String email;
    // No password or sensitive fields
}
```

## DTO Pattern Implementation

### Basic DTO Example
```java
// User DTO
public class UserDTO {
    private Long id;
    private String username;
    private String email;

    // Constructors
    public UserDTO() {}

    public UserDTO(Long id, String username, String email) {
        this.id = id;
        this.username = username;
        this.email = email;
    }

    // Getters and Setters
    // Equals and HashCode methods
}
```

### Repository Layer
```java
@Repository
public interface UserRepository extends JpaRepository<User, Long> {
    Optional<User> findByUsername(String username);
}
```

### Service Layer with DTO Conversion
```java
@Service
public class UserService {
    @Autowired
    private UserRepository userRepository;

    // Convert Entity to DTO
    public UserDTO convertToDto(User user) {
        return new UserDTO(
            user.getId(), 
            user.getUsername(), 
            user.getEmail()
        );
    }

    // Retrieve User DTO
    public UserDTO getUserById(Long id) {
        User user = userRepository.findById(id)
            .orElseThrow(() -> new ResourceNotFoundException("User not found"));
        return convertToDto(user);
    }

    // Create User from DTO
    public UserDTO createUser(UserDTO userDTO) {
        User user = new User();
        user.setUsername(userDTO.getUsername());
        user.setEmail(userDTO.getEmail());
        
        User savedUser = userRepository.save(user);
        return convertToDto(savedUser);
    }
}
```

## Mapping Strategies

### 1. Manual Mapping
```java
// Manual conversion method
public UserDTO mapUserToDto(User user) {
    UserDTO dto = new UserDTO();
    dto.setId(user.getId());
    dto.setUsername(user.getUsername());
    return dto;
}
```

### 2. ModelMapper Library
```java
// Dependency: implementation 'org.modelmapper:modelmapper:2.4.4'
@Configuration
public class ModelMapperConfig {
    @Bean
    public ModelMapper modelMapper() {
        return new ModelMapper();
    }
}

@Service
public class UserService {
    @Autowired
    private ModelMapper modelMapper;

    public UserDTO getUserDto(User user) {
        return modelMapper.map(user, UserDTO.class);
    }
}
```

### 3. MapStruct (Compile-time Mapping)
```java
// MapStruct Mapper
@Mapper(componentModel = "spring")
public interface UserMapper {
    UserMapper INSTANCE = Mappers.getMapper(UserMapper.class);

    UserDTO userToUserDTO(User user);
    User userDTOToUser(UserDTO userDTO);
}
```

## Validation

### DTO Validation
```java
public class UserDTO {
    @NotNull(message = "Username cannot be null")
    @Size(min = 3, max = 50, message = "Username must be between 3 and 50 characters")
    private String username;

    @Email(message = "Invalid email format")
    private String email;
}
```

### Controller Validation
```java
@RestController
public class UserController {
    @PostMapping("/users")
    public ResponseEntity<UserDTO> createUser(
        @Valid @RequestBody UserDTO userDTO) {
        UserDTO createdUser = userService.createUser(userDTO);
        return ResponseEntity.ok(createdUser);
    }
}
```

## Best Practices

### Mapping Best Practices
1. Keep DTOs simple and focused
2. Use different DTOs for different use cases
3. Avoid complex logic in DTOs
4. Use validation annotations
5. Consider performance when mapping

### Example of Multiple DTOs
```java
// Detailed DTO for admin
public class UserAdminDTO extends UserDTO {
    private List<RoleDTO> roles;
    private LocalDateTime lastLogin;
}

// Minimal DTO for public profile
public class UserPublicProfileDTO {
    private String username;
    private String displayName;
}
```

## Advanced Techniques

### Projection Interfaces
```java
// Spring Data Projection
public interface UserProjection {
    String getUsername();
    String getEmail();
}

// Repository Method
public interface UserRepository extends JpaRepository<User, Long> {
    UserProjection findProjectedById(Long id);
}
```

### Inheritance and Polymorphic DTOs
```java
@JsonTypeInfo(use = JsonTypeInfo.Id.NAME, property = "type")
@JsonSubTypes({
    @JsonSubTypes.Type(value = AdminDTO.class, name = "admin"),
    @JsonSubTypes.Type(value = CustomerDTO.class, name = "customer")
})
public abstract class BaseUserDTO {
    private Long id;
    private String username;
}
```

## Common Challenges and Solutions

### Performance Considerations
- Use lazy loading carefully
- Implement efficient mapping strategies
- Use projections for large datasets
- Consider caching mapped results

### Security Considerations
- Never include sensitive data in DTOs
- Use proper validation
- Implement role-based access control

## Conclusion
The DTO pattern is a powerful technique in Spring Boot for managing data transfer, providing clean separation of concerns, and controlling data exposure.

## Recommended Libraries
- ModelMapper
- MapStruct
- Jackson (for JSON serialization)
- Hibernate Validator

## Further Reading
- Spring Boot Documentation
- Baeldung DTO Tutorials
- Martin Fowler's DTO Pattern Article