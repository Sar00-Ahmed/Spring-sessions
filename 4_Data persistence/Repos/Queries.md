
# Query Methods
```java
[action]By[Property][Condition][Connector][Property][Condition]...[OrderBy][Field][Asc/Desc]
```

|Part|Description|Examples|
|---|---|---|
|**[action]**|What kind of query you want|`find`, `read`, `get`, `count`, `exists`, `delete`, `remove`|
|**By**|Always written as “By” — separates the action from conditions|`findBy`, `countBy`, `existsBy`|
|**[Property]**|The name of the entity field you’re filtering on (must match the field name in your entity class)|`Name`, `Email`, `Status`, `CreatedDate`|
|**[Condition] (optional)**|Adds an operator or pattern for comparison|`LessThan`, `GreaterThanEqual`, `Containing`, `Between`, `IsNull`, `True`, etc.|
|**[Connector] (optional)**|Used to combine multiple conditions|`And`, `Or`|

### Examples

```java
findByEmail(String email)
```

→ Finds all records where `email = ?`.

```java
findByStatusAndDepartment(String status, String department)
```

→ `WHERE status = ? AND department = ?`

```java
findBySalaryGreaterThan(Double amount)
```

→ `WHERE salary > ?`

```java
findByNameContaining(String part)
```

→ `WHERE name LIKE %part%`

```java
findByStatusOrDepartment(String status, String department)
```

→ `WHERE status = ? OR department = ?`

```java
findTop5ByStatusOrderByCreatedDateDesc()
```

→ Finds the first 5 records with a given status, ordered by `createdDate` descending.
> [!todo] Excercise
> - Find the top 5 active users whose name contains “ahmed”, ordered by registration date descending.
> 	`findTop5ByNameContainingOrderedBy(String name, LocalDate date)`
>- Count how many employees in department “HR” have a salary greater than 10000.  
>	`CountByDepartmentNameAndSalaryGreaterThan(String name, Int Salary)`
>- Check if any orders exist with status “SHIPPED” and deliveryDate before today.  
>	``
>- Delete all products that are **inactive** and have `stockQuantity` less than 5.  
>- Find all customers from Egypt whose name **starts with** “a” and sort them by `lastName` ascending.
>	`FindByCountryAndnameStartingWithOrderByLastNameAsc(String....)`

# `@Query` annotation
### When to use
- The method naming convention becomes too long or complex.
``` Java
List<User> findByFirstNameAndLastNameAndAgeGreaterThanAndDepartmentNameAndCreatedDateBetweenAndIsActiveTrue(
        String firstName, 
        String lastName, 
        int age, 
        String departmentName, 
        Date startDate, 
        Date endDate
    );
```
- You need to perform a `JOIN` or other complex operation that isn't easily expressed with derived query methods.
> [!todo] Research
> Can query methods handle joins? If so then when and why do we use `@Query`

- You want to use **native SQL** instead of JPQL.
- You need to `UPDATE` or `DELETE` data (which isn't the default for repository methods).
#### JPQL (Java Persistence Query Language) - DEFAULT
*   **Object-Oriented:** You query against entity classes and their attributes, not database tables and columns.
*   **It's database-agnostic**.
*  Uses **entity class names and attribute names**.

```java
public interface UserRepository extends JpaRepository<User, Long> {

    // Query on the 'User' entity and its 'email' field
    @Query("SELECT u FROM User u WHERE u.email = ?1")
    User findByEmailAddress(String emailAddress);

    // More complex with a JOIN
    @Query("SELECT u FROM User u JOIN u.orders o WHERE o.total > :amount")
    List<User> findUsersWithOrdersOver(@Param("amount") BigDecimal amount);
}
```

#### Native SQL
*   You write raw SQL that is **specific to your database**.
*   Can be **necessary for complex, database-specific features**.
*   You must set `nativeQuery = true` and **query the actual table/column names**.

```java
public interface UserRepository extends JpaRepository<User, Long> {

    @Query(value = "SELECT * FROM users u WHERE u.email = ?1", nativeQuery = true)
    User findByEmailAddressNative(String emailAddress);

    // Native query with pagination (requires a countQuery!)
    @Query(value = "SELECT * FROM users u WHERE u.active = true",
           countQuery = "SELECT count(*) FROM users u WHERE u.active = true",
           nativeQuery = true)
    Page<User> findAllActiveUsers(Pageable pageable);
}
```

### Parameter Binding: How to Pass Values

```java
@Query("SELECT u FROM User u WHERE u.name = :name AND u.age > :minAge")
List<User> findByNameAndAgeMin(@Param("name") String name, 
                               @Param("minAge") int minAge);
```

### Count
``` Java
@Query(countQuery = "SELECT COUNT(*) FROM ...")
```

# Modifying Queries: For UPDATE and DELETE

By default, `@Query` methods are for `SELECT` statements. 
To perform `UPDATE`, `DELETE`, or even `INSERT` (with native queries), you need an extra annotation.

> [!error]
> It sees a `@Query("UPDATE ...")` But since there is **no `@Modifying`**, it still treats this as a **SELECT query**
    
Spring tries to execute it using `Query.getResultList()`

``` console
java.lang.IllegalStateException:  Update/delete queries cannot be executed using getResultList()

javax.persistence.TransactionRequiredException:  Executing an update/delete query
```


It must be used in combination with `@Query` and pair it with a `void`, `int`, or `Integer` return type. 

The `int` value returned is the number of affected rows.

```java
@Modifying
@Query("UPDATE User u SET u.status = :status WHERE u.lastLogin < :date")
int deactivateUsersNotLoggedInSince(@Param("date") LocalDateTime date, 
                                    @Param("status") String status);

@Modifying
@Query("DELETE FROM User u WHERE u.active = false")
void deleteInactiveUsers();

// Native INSERT example
@Modifying
@Query(value = "INSERT INTO users_audit (user_id, action) VALUES (:userId, :action)", 
       nativeQuery = true)
void logUserAction(@Param("userId") Long userId, @Param("action") String action);
```

> [!tip]
> You often need to pair this with `@Transactional` (typically on the service layer, but can be on the repository method) to ensure the operation runs within a transaction. 
> You might also need to clear the persistence context afterwards.

```java
@Modifying
@Query("UPDATE User u SET u.points = u.points + 10")
@Transactional
// @Transactional(readOnly = false) is the default
int addPointsToAllUsers();
// clearAutomatically = true can help avoid outdated data in the context
```


# Return types
## DTO Projection

```java
// DTO Class
public class UserDto {
    private String name;
    private String email;
    // Constructor
    public UserDto(String name, String email) {
        this.name = name;
        this.email = email;
    }
    // ... getters
}

// Repository Method
public interface UserRepository extends JpaRepository<User, Long> {
    @Query("SELECT NEW com.example.dto.UserDto(u.name, u.email) FROM User u WHERE u.department = ?1")
    List<UserDto> findUserDtoByDepartment(String department);
}
```

## Return Other Types
You can return a single attribute, a `List` of attributes, or even a `Map` (with native queries).

```java
// Returning a single attribute
@Query("SELECT u.email FROM User u")
List<String> findAllEmails();

// Returning a custom object array (Object[])
@Query("SELECT u.id, u.name FROM User u")
List<Object[]> findUserIdAndName();
```

> [!todo] Research
> Querying for some data may return null, this makes the app prone to null pointer exceptions. What are ways to handle this?
 

> [!summary]
> 1.  **Prefer JPQL over Native:** Use native queries only when necessary for performance or database-specific features. JPQL is more maintainable and portable.
>2.  **Prefer Named Parameters:** They make your code more readable and less fragile to parameter order changes.
>3.  **Always Use `@Modifying` for `UPDATE`/`DELETE`:** Don't forget this, or you'll get an `InvalidDataAccessApiUsageException`.
>4.  **Use `@Transactional` with `@Modifying`:** Ensure your write operations are wrapped in a transaction.
>5.  **Use DTO Projections for Read-Only Data:** If you only need a subset of data, projecting to a DTO is more efficient than fetching the entire entity graph.
>6.  **Pagination Works:** You can use `Pageable` as a parameter with `@Query` methods. For native queries, you must provide a `countQuery`.

# References
> [!cite]
> ```embed
title: "Using Optional in Java the right way"
image: "https://media2.dev.to/dynamic/image/width=1000,height=500,fit=cover,gravity=auto,format=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fgkx6niy2v7qgi0wayt12.png"
description: "It doesn't matter how much time you have been programming, NullPointerException is one of the most..."
url: "https://dev.to/sohailshah/using-optionals-in-java-the-right-way-4aho"
favicon: ""
aspectRatio: "50"
>```

#core