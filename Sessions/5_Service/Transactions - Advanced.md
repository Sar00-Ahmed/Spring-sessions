## Basic Usage
> [!definition]
>`@Transactional` defines the boundaries of a single unit of work. Starting a transaction when the annotated method is called, committing it on success, and rolling it back on failure.

```java
@Service
@Transactional
public class UserService {
    
    public User createUser(String name) {
        User user = new User(name);
        return userRepository.save(user);
    }
}
```

**Spring creates a proxy to wrap the method execution within a transaction.**
## Key Concepts You MUST Know
### Propagation (How transactions behave when calling other transactional methods)

```java
@Transactional(propagation = Propagation.REQUIRED) // DEFAULT
public void requiredExample() {
    // If no transaction exists: creates new one
    // If transaction exists: joins existing one
}
```
 Example: A service method that updates multiple related database records and requires atomicity for all operations.

``` Java
@Transactional(propagation = Propagation.REQUIRES_NEW)
public void requiresNewExample() {
    // Always creates a new transaction
    // Suspends current transaction if one exists
}
```
Often because you want changes committed immediately or rollback independently. 

Example: Logging or audit actions inside a transactional service where failures in logging should not impact the main transaction.

```
@Transactional(propagation = Propagation.NESTED)
public void nestedExample() {
    // Creates a savepoint within existing transaction
    // Partial rollback possible
}

@Transactional(propagation = Propagation.SUPPORTS)
public void supportsExample() {
    // Uses existing transaction if available
    // No transaction created if none exists
}

@Transactional(propagation = Propagation.NEVER)
public void neverExample() {
    // Must NOT be called within a transaction
    // Throws exception if transaction exists
}
```

### Isolation (How transactions interact with concurrent access)

```java
@Transactional(isolation = Isolation.READ_COMMITTED) // Common default
public void readCommitted() {
    // Can only read committed data
    // Prevents dirty reads
}

@Transactional(isolation = Isolation.REPEATABLE_READ)
public void repeatableRead() {
    // Consistent reads within same transaction
    // Prevents non-repeatable reads
}

@Transactional(isolation = Isolation.SERIALIZABLE)
public void serializable() {
    // Highest isolation, sequential execution
    // Prevents phantom reads
}
```

## 3. Rollback Rules

```java
@Transactional(rollbackFor = Exception.class) // Rollback on any Exception
public void transferMoney() throws InsufficientFundsException {
    // Business logic that might throw checked exception
}

@Transactional(noRollbackFor = IllegalArgumentException.class)
public void noRollbackOnIllegalArgument() {
    // Transaction won't rollback for IllegalArgumentException
}

// DEFAULT: Rollbacks only on RuntimeExceptions and Errors
// NOT on checked Exceptions
```

## 4. Important Configuration Options

```java
@Transactional(
    readOnly = true,          // Optimization hint for reads
    timeout = 30,             // Seconds before transaction times out
    rollbackFor = Exception.class,
    propagation = Propagation.REQUIRED
)
public void complexOperation() {
    // Method implementation
}
```

## 5. Common Pitfalls and Solutions

### Pitfall 1: Self-Invocation
```java
@Service
@Transactional
public class UserService {
    
    public void createUsers() {
        createUser("John");  // ❌ @Transactional won't work!
        this.createUser("Jane"); // ❌ Still won't work!
    }
    
    @Transactional
    public void createUser(String name) {
        userRepository.save(new User(name));
    }
}

// Solution: Use external call or self-injection
@Service
@Transactional
public class UserService {
    @Autowired
    private UserService self; // Self-injection
    
    public void createUsers() {
        self.createUser("John"); // ✅ Works through proxy
    }
}
```

### Pitfall 2: Wrong Annotation Placement
```java
// ❌ On interface (less flexible)
public interface UserService {
    @Transactional
    void createUser();
}

// ✅ On concrete class (recommended)
@Service
@Transactional
public class UserServiceImpl implements UserService {
    public void createUser() { ... }
}

// ✅ On public methods (private methods don't work)
@Service
public class UserService {
    @Transactional
    public void publicMethod() { ... } // ✅ Works
    
    @Transactional
    private void privateMethod() { ... } // ❌ Ignored
}
```

### Pitfall 3: Mixing Data Access Technologies
```java
@Transactional
public void mixedAccess() {
    // JPA operation
    userRepository.save(user);
    
    // JDBC operation - might not see unflushed JPA changes
    jdbcTemplate.update("UPDATE accounts SET balance = ?", amount);
    
    // Solution: Force flush
    userRepository.flush();
}
```

## 7. Best Practices

### Use at Service Layer
```java
@Service
@Transactional
public class OrderService {
    // Business logic with transactional boundaries
    // Coordinates multiple repositories
}

@Repository
public class OrderRepository {
    // Usually no @Transactional here
    // Let service layer manage transactions
}
```

### Read-Only for Queries
```java
@Transactional(readOnly = true)
public List<User> findAllUsers() {
    return userRepository.findAll(); // Optimized for reads
}

@Transactional // Read-write by default
public User createUser(User user) {
    return userRepository.save(user);
}
```

### Keep Transactions Short
```java
// ❌ Don't do this
@Transactional
public void longRunningProcess() {
    processStep1();
    Thread.sleep(5000); // ❌ Long operation holding transaction
    processStep2();
}

// ✅ Better approach
public void longRunningProcess() {
    processStep1();
    // No transaction for waiting
    Thread.sleep(5000);
    transactionalStep2();
}

@Transactional
public void transactionalStep2() {
    processStep2();
}
```

## Quick Reference Cheatsheet

```java
// Most common usage patterns:

// 1. Default transaction
@Transactional
public void defaultTx() {}

// 2. Read-only query
@Transactional(readOnly = true)
public void readOperation() {}

// 3. Custom rollback
@Transactional(rollbackFor = BusinessException.class)
public void businessOperation() {}

// 4. New transaction
@Transactional(propagation = Propagation.REQUIRES_NEW)
public void independentOperation() {}

// 5. Time-bound operation
@Transactional(timeout = 10)
public void timeSensitiveOperation() {}
```



#todo 