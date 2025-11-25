# What did repos add?
### From SQL to JPQL

| SQL Concept                            | JPQL Equivalent        | Mental Bridge                              |
| -------------------------------------- | ---------------------- | ------------------------------------------ |
| `SELECT * FROM users`                  | `SELECT u FROM User u` | Think "from Entity" not "from table"       |
| `WHERE age > 18`                       | `WHERE u.age > 18`     | Use object properties, not column names    |
| `JOIN orders o ON user.id = o.user_id` | `JOIN u.orders o`      | Use object relationships, not foreign keys |

### From JDBC to Spring Data

|JDBC Concept|Spring Data Equivalent|Mental Shift|
|---|---|---|
|`Connection`, `PreparedStatement`|Repository Interface|Declarative vs. Imperative|
|Manual parameter setting|Method parameters|Automatic parameter binding|
|`ResultSet` processing|Return types (`List<T>`, `Optional<T>`)|Automatic object mapping|
|Exception handling|Spring DataAccessException|Unified exception hierarchy|

### From DAO Pattern to Repository Pattern

``` java

// Traditional DAO 
public interface UserDao {
    User findById(Long id);
    List<User> findAll();
    void save(User user);
    void update(User user);
    void delete(Long id);
}

// Spring Data Repository - This is what we're learning
public interface UserRepository extends JpaRepository<User, Long> {
    // Methods auto-implemented!
}
```

# Repository interfaces
### Inheritance Hierarchy:
``` mermaid
classDiagram
    direction BT
    class Repository {
        <<marker interface>>
    }
    
    class CrudRepository~T, ID~ {
        <<interface>>
        +save(S entity) S
        +saveAll(Iterable~S~ entities) Iterable~S~
        +findById(ID id) Optional~T~
        +existsById(ID id) boolean
        +findAll() Iterable~T~
        +findAllById(Iterable~ID~ ids) Iterable~T~
        +count() long
        +delete(T entity) void
        +deleteById(ID id) void
        +deleteAll(Iterable~? extends T~ entities) void
        +deleteAll() void
    }
    
    class PagingAndSortingRepository~T, ID~ {
        <<interface>>
        +findAll(Sort sort) Iterable~T~
        +findAll(Pageable pageable) Page~T~
    }
    
    class JpaRepository~T, ID~ {
        <<interface>>
        +findAll() List~T~
        +findAll(Sort sort) List~T~
        +saveAll(Iterable~S~ entities) List~S~
        +flush() void
        +saveAndFlush(S entity) S
        +saveAllAndFlush(Iterable~S~ entities) List~S~
        +deleteAllInBatch(Iterable~T~ entities) void
        +deleteAllByIdInBatch(Iterable~ID~ ids) void
        +deleteAllInBatch() void
        +getReferenceById(ID id) T$
    }
    
    Repository <|-- CrudRepository
    CrudRepository <|-- PagingAndSortingRepository
    PagingAndSortingRepository <|-- JpaRepository
```

1. **You get all methods automatically** - no implementation needed
2. **All methods are transactional** (for write operations)
3. **Exception translation** is handled automatically (SQL exceptions → Spring exceptions)


## CrudRepository 
provides basic Create, Read, Update, Delete operations for a single entity.

```java
public interface UserRepository extends CrudRepository<User, Long> {
    // Inherits all CRUD methods automatically
}

// Usage:
User user = new User("John", "john@email.com");
userRepository.save(user);                    // CREATE

Optional<User> found = userRepository.findById(1L);  // READ
userRepository.existsById(1L);                // CHECK EXISTS

user.setEmail("new@email.com");
userRepository.save(user);                    // UPDATE

userRepository.deleteById(1L);                // DELETE
```
## JpaRepository 

JPA-specific functionality + everything from CrudRepository

```java
public interface UserRepository extends JpaRepository<User, Long> {
    // Has ALL CrudRepository methods + JPA extras
}

// Usage examples of JPA-specific features:

// Pagination
Page<User> usersPage = userRepository.findAll(
    PageRequest.of(0, 10, Sort.by("name").ascending())
);

// Batch operations - more efficient
List<User> users = Arrays.asList(user1, user2, user3);
userRepository.saveAllAndFlush(users);  // Save and immediately flush

// Bulk delete - single query instead of multiple
userRepository.deleteAllInBatch(users); // Much faster for large datasets
```


| Operation | CrudRepository | JpaRepository |
|-----------|----------------|---------------|
| **Save single** | `save(entity)` | `save(entity)` |
| **Save multiple** | `saveAll(entities)` | `saveAll(entities)` |
| **Find by ID** | `findById(id)` | `findById(id)` |
| **Find all** | `findAll()` → `Iterable<T>` | `findAll()` → `List<T>` |
| **Find all sorted** | ❌ Not available | `findAll(Sort sort)` |
| **Find all paginated** | ❌ Not available | `findAll(Pageable pageable)` |
| **Check exists** | `existsById(id)` | `existsById(id)` |
| **Count** | `count()` | `count()` |
| **Delete by entity** | `delete(entity)` | `delete(entity)` |
| **Delete by ID** | `deleteById(id)` | `deleteById(id)` |
| **Flush changes** | ❌ Not available | `flush()` |
| **Save + flush** | ❌ Not available | `saveAndFlush(entity)` |
| **Batch delete** | ❌ Not available | `deleteAllInBatch(entities)` |

#conceptual 