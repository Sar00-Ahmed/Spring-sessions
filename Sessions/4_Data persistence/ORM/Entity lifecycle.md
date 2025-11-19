### The Core States of an Entity
An entity, which is a simple Java object (POJO) annotated with `@Entity`, can be in one of four main states:

1.  **Transient**
2.  **Managed (or Persistent)**
3.  **Detached**
4.  **Removed**

![[Pasted image 20251118090558.png]]
### Transient
An object is **Transient** if it has just been instantiated using the `new` operator. 

It has no association with the Hibernate `Session` (or JPA `EntityManager`) and **no representation in the database**.

```java
// This is a NEW object. It's in the Transient state.
User user = new User();
user.setName("John Doe");
user.setEmail("john@example.com");
// At this point, 'user' is Transient.
// It exists only in memory.
```

### 2. Managed (Persistent)
It is associated with a persistence context (Hibernate `Session`/JPA `EntityManager`). Hibernate is now **tracking changes** to this object.

Any changes made to the object in this state will be **automatically detected and synchronized with the database** when the session is flushed (typically at transaction commit). This is the magic of "**dirty checking.**"

``` java
// Transition from Transient -> Managed
User savedUser = userRepository.save(user);
// 'savedUser' is now a Managed Entity.

// Or, by fetching from the DB (this object is also Managed)
User foundUser = userRepository.findById(1L).get();
// 'foundUser' is Managed.

// Because it's Managed, changes are auto-saved.
foundUser.setName("Jane Doe");
// You DON'T need to call save() again at this point!
// When the transaction commits, Hibernate will detect the change
// and execute an UPDATE query.
```
### 3. Detached
A **Detached** entity is one that was previously Managed but is no longer associated with an active persistence context (e.g., the `Session` was closed, the `EntityManager` was closed, or the transaction ended).

**Database Impact:** None. Changes made to a detached object are **NOT automatically tracked or saved**.

**How it gets here:**
- The `Session` or `EntityManager` is closed.
- The transaction ends (in a non-web context, this often happens after a `@Transactional` method finishes).
- You manually `evict` or `detach` the entity from the session.

```java
@Transactional
public void updateUser() {
    // Inside a transaction, the entity is Managed.
    User user = userRepository.findById(1L).get(); // Managed
    user.setName("New Name");
    // Change will be persisted when method exits (transaction commits).
}
// The transaction and session are now closed.
// The 'user' object is now Detached.

// ... later, in a different method or request ...
public void updateDetachedUser() {
    // This user object is Detached. If we change it...
    user.setEmail("new@email.com"); // ...nothing happens to the DB.

    // To save changes, we must use merge().
    // This copies the state of the detached object onto a new Managed instance.
    User managedUser = userRepository.save(user); // This is a merge operation
    // 'managedUser' is now Managed again.
}
```
### 4. Removed
A **Removed** entity is one that has been scheduled for deletion from the database. It is still associated with the persistence context but will be removed upon the next flush.

A `DELETE` SQL statement will be executed when the session is flushed.
*   **How it gets here:** You call `remove` on a Managed entity (`repository.delete(entity)` or `entityManager.remove()`).
*   **What happens next?** After the session is flushed and the transaction commits, the entity is completely gone from the database and becomes **Transient** (a "ghost" object in memory).

```java
@Transactional
public void deleteUser() {
    User user = userRepository.findById(1L).get(); // Managed State
    userRepository.delete(user); // Transition to Removed State
    // The entity is now in the Removed state.
    // The actual DELETE statement will be executed when the transaction commits.
}
// After commit, the object is Transient (and the DB row is gone).
```
### Key Takeaways for Spring Developers

1.  **Automatic Dirty Checking:** Only works on **Managed** entities. This is why you often see `@Transactional` on service methods—it keeps the entity managed for the duration of the business operation.
2.  **`save()` vs `merge()`:** In Spring Data JPA, the `save()` method is clever. It acts as a `persist` for new (Transient) entities and as a `merge` for existing (Detached) entities.
3.  **LazyInitializationException:** This common error occurs when you try to access a lazy-loaded collection (e.g., `user.getOrders()`) on a **Detached** entity. The session is closed, so Hibernate can't fetch the data from the database.
4.  **Stateless Nature of Web:** In a typical web request, the session is often open for the duration of the request (Open Session in View pattern) or just for the service method. When the response is sent back to the client, all entities become **Detached**.

> [!todo] Research
> What are key differences between Transient and Detached states

# Tricky scenarios
## Scenario 1: No `@Transactional` - Edit After Save

```java
@Service
public class UserService {
    
    public void saveAndEditWithoutTransaction() {
        // Transient -> Managed (temporarily)
        User user = new User("John");
        User savedUser = userRepository.save(user); // INSERT happens immediately?
        System.out.println("ID: " + savedUser.getId()); // ID is populated
        
        // Now edit the entity
        savedUser.setName("Jane");
        // What happens here?
    }
}
```

1. **INSERT executes immediately** when `save()` is called
2. **Entity becomes DETACHED immediately** after `save()` returns because the internal transaction ends
3. **The change to "Jane" is NOT saved** - the entity is now Detached, so no automatic dirty checking
4. **No UPDATE is sent to database** - the name remains "John" in the database

```java
// To fix this, you need @Transactional on the method:
@Transactional
public void saveAndEditWithTransaction() {
    User user = new User("John");
    User savedUser = userRepository.save(user); // INSERT not necessarily immediate
    
    savedUser.setName("Jane"); // This change WILL be saved
    // UPDATE happens when method exits (transaction commits)
}
```

## Scenario 2: Call `getAll` After Save (WITHIN Transaction)

```java
@Service
@Transactional
public class UserService {
    
    public void testWithinTransaction() {
        User user = new User("John");
        userRepository.save(user); // Becomes Managed
        
        // Get all users
        List<User> allUsers = userRepository.findAll();
        
        // Will the new user be in the list?
        System.out.println("Users count: " + allUsers.size());
        System.out.println("Contains new user: " + allUsers.contains(user));
    }
}
```

1. **YES, the new user WILL be in the list** 
2. **The persistence context** acts as a first-level cache and contains all managed entities
3. **No SQL flush required** - Hibernate checks the persistence context first
4. **The new user exists in memory** as a Managed entity, so `findAll()` will include it

**Note:** The actual `INSERT` might not have executed in the database yet - it might be waiting for transaction commit, but the persistence context knows about the new entity.
## Scenario 3: Call `getAll` After Save (NO Transaction)

```java
@Service
public class UserService {
    
    public void testWithoutTransaction() {
        User user = new User("John");
        userRepository.save(user); // Short internal transaction
        
        // Get all users
        List<User> allUsers = userRepository.findAll(); // New transaction
        
        // Will the new user be in the list?
        System.out.println("Users count: " + allUsers.size());
    }
}
```

1. **YES, the new user WILL be in the list** (but for a different reason)
2. **Why?** Because `save()` already committed the INSERT to the database
3. **Two separate transactions:**
   - First transaction: `save()` → INSERT + COMMIT
   - Second transaction: `findAll()` → SELECT
4. **The entity is now in the database**, so subsequent queries will see it
5. **However, the entity from `save()` is DETACHED** and different from the entities returned by `findAll()`.

## Scenario 4: Call JDBC after Save (Transactional)

``` java
@Transactional
public void mixedAccess() {
    // Save entity using JPA
    User user = new User("John");
    userRepository.save(user); 
        
    // Try to query using JDBC Template
    Integer count = jdbcTemplate.queryForObject(
            "SELECT COUNT(*) FROM users WHERE name = ?", 
            Integer.class, "John");
            
    System.out.println("JDBC count: " + count); 
        
    List<User> users = userRepository.findByName("John");
    System.out.println("JPA count: " + users.size()); 
    
}
```

- **JPA/Hibernate**: Works with the persistence context (1st level cache)
- **JDBC Template**: Bypasses the persistence context and talks directly to the database
- **The INSERT is queued** until Hibernate flushes (usually at transaction commit)
- Solution: `userRepository.flush(); // or entityManager.flush()`
## Key Takeaways

1. **`@Transactional` boundaries define persistence context lifespan**
2. **Within a transaction**: All managed entities are visible to all queries within that same persistence context
3. **Without `@Transactional`**: Each repository method typically uses its own short transaction, so entities become Detached between calls
4. **Database visibility**: Once committed, data is visible to all subsequent transactions (assuming standard isolation levels)

The behavior can also be affected by **flush modes** and **transaction isolation levels**, but these are the default behaviors you'll see most often.

#conceptual 