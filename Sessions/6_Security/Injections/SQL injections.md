Yes, absolutely! Spring provides multiple layers of protection against SQL injection, with **Spring Data JPA** being the primary and most effective defense mechanism.

## 1. Spring Data JPA - The Primary Defense

Spring Data JPA uses Hibernate under the hood, which provides **Object-Relational Mapping (ORM)** that naturally prevents SQL injection through parameter binding.

### Repository Interface (Automatic Queries)
```java
public interface UserRepository extends JpaRepository<User, Long> {
    // These are automatically implemented by Spring - NO SQL injection risk
    User findByUsername(String username);
    List<User> findByEmailAndActiveTrue(String email);
    List<User> findByAgeGreaterThan(int age);
}
```

### Custom Queries with @Query (Safe)
```java
public interface UserRepository extends JpaRepository<User, Long> {
    
    // Positional parameters - SAFE
    @Query("SELECT u FROM User u WHERE u.username = ?1 AND u.email = ?2")
    User findByUsernameAndEmail(String username, String email);
    
    // Named parameters - SAFE  
    @Query("SELECT u FROM User u WHERE u.username = :username AND u.active = :active")
    User findByUsernameAndActiveStatus(@Param("username") String username, 
                                     @Param("active") boolean active);
    
    // Native query with parameters - SAFE when using parameter binding
    @Query(value = "SELECT * FROM users u WHERE u.username = :username", nativeQuery = true)
    User findByUsernameNative(@Param("username") String username);
}
```

## 2. JPA Criteria API (Type-Safe Queries)

For dynamic queries, use the Criteria API which is completely type-safe:

```java
@Repository
public class UserCustomRepository {
    
    @PersistenceContext
    private EntityManager entityManager;
    
    public List<User> findUsersByDynamicCriteria(String username, String email, Boolean active) {
        CriteriaBuilder cb = entityManager.getCriteriaBuilder();
        CriteriaQuery<User> query = cb.createQuery(User.class);
        Root<User> root = query.from(User.class);
        
        List<Predicate> predicates = new ArrayList<>();
        
        if (username != null) {
            predicates.add(cb.equal(root.get("username"), username)); // SAFE
        }
        if (email != null) {
            predicates.add(cb.like(root.get("email"), "%" + email + "%")); // SAFE
        }
        if (active != null) {
            predicates.add(cb.equal(root.get("active"), active)); // SAFE
        }
        
        query.where(predicates.toArray(new Predicate[0]));
        return entityManager.createQuery(query).getResultList();
    }
}
```

## 3. Spring JDBC Template (Safe Alternative to Raw JDBC)

If you need to use SQL directly, **always use JdbcTemplate** instead of raw JDBC:

### ❌ DANGEROUS - String Concatenation
```java
// NEVER DO THIS - Vulnerable to SQL injection
public User findUserUnsafe(String username) {
    String sql = "SELECT * FROM users WHERE username = '" + username + "'";
    return jdbcTemplate.queryForObject(sql, new UserRowMapper());
}
```

### ✅ SAFE - Using JdbcTemplate with Parameters
```java
@Repository
public class UserJdbcRepository {
    
    @Autowired
    private JdbcTemplate jdbcTemplate;
    
    // Positional parameters - SAFE
    public User findUserSafe(String username) {
        String sql = "SELECT * FROM users WHERE username = ?";
        return jdbcTemplate.queryForObject(sql, new UserRowMapper(), username);
    }
    
    // Named parameters - SAFE
    public User findUserWithNamedParams(String username, String email) {
        String sql = "SELECT * FROM users WHERE username = :username AND email = :email";
        
        MapSqlParameterSource params = new MapSqlParameterSource()
            .addValue("username", username)
            .addValue("email", email);
            
        return namedParameterJdbcTemplate.queryForObject(sql, params, new UserRowMapper());
    }
    
    // IN clause - SAFE
    public List<User> findUsersInList(List<String> usernames) {
        String sql = "SELECT * FROM users WHERE username IN (?)";
        return jdbcTemplate.query(sql, new UserRowMapper(), usernames.toArray());
    }
}
```

## 4. Spring Data REST - Automatic Protection

If you're using Spring Data REST, it's automatically protected:

```java
// Automatically secured against SQL injection
public interface ProductRepository extends JpaRepository<Product, Long> {
    List<Product> findByNameContaining(String name);  // SAFE
}

// REST endpoints are automatically created and secured
// GET /api/products/search/findByNameContaining?name=test
```

## 5. Validation Layer with Spring Validation

Combine database protection with input validation:

```java
public class UserDto {
    
    @NotBlank
    @Size(min = 3, max = 50)
    @Pattern(regexp = "^[a-zA-Z0-9_]+$") // Only allow alphanumeric and underscore
    private String username;
    
    @Email
    private String email;
    
    // getters and setters
}

@RestController
public class UserController {
    
    @PostMapping("/users")
    public ResponseEntity<User> createUser(@Valid @RequestBody UserDto userDto) {
        // Input is validated before reaching database
        User user = userService.createUser(userDto);
        return ResponseEntity.ok(user);
    }
}
```

## 6. Advanced: Custom Security Configuration

For additional protection, you can configure security at the application level:

```yaml
# application.yml
spring:
  sql:
    init:
      mode: always
  jpa:
    show-sql: true
    properties:
      hibernate:
        # Additional safety measures
        connection:
          # Limit exposure in error messages
          show_sql: false
          use_sql_comments: false
```

## Common Vulnerable Patterns to Avoid

### ❌ NEVER DO THESE:

```java
// 1. String concatenation in JPA
@Query("SELECT u FROM User u WHERE u.username = '" + "#{username}" + "'")
User findVulnerable(String username);

// 2. Raw SQL concatenation
String sql = "SELECT * FROM users WHERE username = '" + userInput + "'";

// 3. JPA with unchecked expressions
String jpql = "SELECT u FROM User u WHERE u.username = " + userInput;
Query query = entityManager.createQuery(jpql);

// 4. Native queries with concatenation
@Query(value = "SELECT * FROM users WHERE username = '" + "#{username}" + "'", 
       nativeQuery = true)
User findVulnerableNative(String username);
```

## Summary

Spring provides comprehensive SQL injection protection through:

1. **Spring Data JPA** - Automatic parameter binding in derived queries and `@Query` annotations
2. **JdbcTemplate** - Safe parameterized queries for raw SQL
3. **Criteria API** - Type-safe query building
4. **Input Validation** - Bean Validation API to sanitize input before database access
5. **Automatic Escaping** - All Spring Data modules automatically escape parameters

The key principle is: **Always use parameter binding** instead of string concatenation, and leverage Spring's built-in data access abstractions rather than writing raw SQL/JDBC code.

Spring makes it easy to write safe code - you have to actively work around the safety mechanisms to introduce SQL injection vulnerabilities.