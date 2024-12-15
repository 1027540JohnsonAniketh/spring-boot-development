# Spring JPA (Java Persistence API) Notes

## Overview
Spring JPA is a powerful abstraction for database operations in Spring applications, providing a simplified approach to data persistence and retrieval.

## Key Concepts

### 1. JPA Repository Interfaces
Spring Data JPA provides repository interfaces that make database operations straightforward:

```java
public interface UserRepository extends JpaRepository<User, Long> {
    // Built-in CRUD operations
    // Custom query methods can be added here
}
```

### 2. Repository Types
- `JpaRepository`: Full-featured repository with CRUD and pagination support
- `CrudRepository`: Basic CRUD operations
- `PagingAndSortingRepository`: Adds pagination and sorting capabilities

### 3. Query Methods
Spring JPA allows creating database queries through method naming conventions:

```java
// Automatically generates a SELECT query
List<User> findByLastNameAndAge(String lastName, int age);

// With @Query annotation for complex queries
@Query("SELECT u FROM User u WHERE u.status = :status AND u.age > :age")
List<User> findCustomUsers(@Param("status") String status, @Param("age") int age);
```

### 4. Entity Mapping Annotations
- `@Entity`: Marks a class as a persistent entity
- `@Table`: Specifies table name
- `@Column`: Defines column properties
- `@Id`: Primary key
- `@GeneratedValue`: Automatic ID generation

```java
@Entity
@Table(name = "users")
public class User {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    @Column(nullable = false, length = 50)
    private String username;
}
```

### 5. Relationship Mappings
- `@OneToMany`: One-to-many relationship
- `@ManyToOne`: Many-to-one relationship
- `@OneToOne`: One-to-one relationship
- `@ManyToMany`: Many-to-many relationship

```java
@Entity
public class Department {
    @Id
    private Long id;

    @OneToMany(mappedBy = "department")
    private List<Employee> employees;
}
```

### 6. Pagination and Sorting
```java
// Pagination example
Pageable firstPageWithTwoElements = PageRequest.of(0, 2);
Page<User> userPage = userRepository.findAll(firstPageWithTwoElements);

// Sorting example
Sort sortByName = Sort.by("lastName").descending();
List<User> sortedUsers = userRepository.findAll(sortByName);
```

### 7. Transaction Management
- `@Transactional`: Ensures method runs in a single transaction
- Supports rollback on exceptions
- Can specify isolation and propagation levels

```java
@Transactional(rollbackFor = CustomException.class)
public void performComplexOperation() {
    // Database operations
}
```

### 8. Performance Considerations
- Use `@Query` for complex queries
- Leverage lazy and eager loading
- Use projection for partial data retrieval
- Consider using specification for dynamic queries

### 9. Common Configuration
```properties
# application.properties
spring.jpa.hibernate.ddl-auto=update
spring.datasource.url=jdbc:mysql://localhost:3306/mydb
spring.datasource.username=root
spring.datasource.password=password
```

## Best Practices
- Keep repository interfaces clean and focused
- Use meaningful method names
- Avoid N+1 query problems
- Use appropriate fetch types
- Implement proper error handling

## Potential Pitfalls
- Overusing eager loading
- Not managing transaction boundaries
- Ignoring query performance
- Complex relationship mappings

## Recommended Tools
- Spring Data JPA
- Hibernate
- Liquibase/Flyway for database migrations
- Database profiling tools

## Learning Resources
- Spring Official Documentation
- Baeldung Spring JPA Tutorials
- Hibernate Reference Documentation

## Sample Project Structure
```
src/
├── main/
│   ├── java/
│   │   └── com/example/
│   │       ├── entity/
│   │       ├── repository/
│   │       └── service/
│   └── resources/
│       └── application.properties
```