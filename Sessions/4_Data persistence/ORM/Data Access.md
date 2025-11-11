## Historical & Conceptual Overview

### 1. Early Days: JDBC (Java Database Connectivity) — 1997

JDBC is the low-level API introduced in Java 1.1 for interacting with relational databases.
    
It allows developers to:
- Open a database connection
- Execute SQL queries
 - Process results
    
> [!caution] Problem
 > Too verbose and error-prone (manual connection handling, SQL, result set processing)
 > Logic for persistence scattered across code
 > Developers wanted an abstraction over JDBC to reduce boilerplate.

``` Java
Connection conn = DriverManager.getConnection(...); 
PreparedStatement stmt = conn.prepareStatement("SELECT * FROM users WHERE id=?"); stmt.setInt(1, 42); 
ResultSet rs = stmt.executeQuery();

```

### 2. Rise of ORM (Object-Relational Mapping) — Late 1990s to Early 2000s

A programming technique that bridges the gap between object-oriented programming languages and relational databases by mapping application objects to database tables, allowing developers to interact with data using familiar objects instead of raw SQL queries.
    
ORM handles:
- Mapping Java fields to table columns
- Managing object lifecycle (insert, update, delete)     
- Automatic SQL generation        

##### Benefits:
- Reduces boilerplate code
- Encourages domain-driven design
- Hides database details

### 3. Hibernate (2001)
Hibernate is the most popular ORM implementation in Java. Initially developed independently of Java EE

Hibernate introduced:
- Entity mappings via XML (later via annotations)
- SessionFactory for managing sessions
- HQL (Hibernate Query Language)    

``` Java
SessionFactory factory = new Configuration().configure().buildSessionFactory();
Session session = factory.openSession();

User user = session.get(User.class, 42); // No SQL, just an object
System.out.println(user.getUsername());

session.close();
factory.close();

```
### 4. JPA (Java Persistence API) — 2006 (part of Java EE 5)

JPA is the official Java standard for ORM, introduced to unify ORM practices. It defines a specification (interfaces, annotations), not an implementation.
##### JPA provides

- @Entity, @Id, @OneToMany, etc. annotations
- EntityManager API
- JPQL (Java Persistence Query Language)
- Life-cycle callbacks

 Note: Hibernate is still used under the hood in most Spring apps, but now through JPA.

``` Java
public interface UserRepository extends JpaRepository<User, Integer> {}

// Somewhere in your service class:
User user = userRepository.findById(42).orElseThrow();
System.out.println(user.getUsername());

```
### 5. Spring and Data Persistence

Spring started as a lightweight alternative to Java EE. It aimed to simplify enterprise development.

Over time, Spring introduced:
#### a. Spring JDBC (early 2000s)
- A wrapper over raw JDBC to reduce boilerplate
#### b. Spring ORM
- Integration with Hibernate and JPA
- Managed transactions and sessions for you
#### c. Spring Data JPA (2010s)
- High-level abstraction over JPA
- Auto-generates queries from method names
- Provides Repository interfaces like:
    
``` Java
public interface UserRepository extends JpaRepository<User, Long> {     List<User> findByLastName(String lastName); }
```
    
With Spring Data JPA, you can build repositories without writing SQL or JPQL manually.


``` mermaid
flowchart TD
    JDBC["JDBC\n(Low-level SQL API)"]
    ORM["ORM Concept\n(Object-Relational Mapping)"]
    SPRINGJDBC["Spring JDBC\n(Helper classes)"]
    HIBERNATE["Hibernate\n(ORM library)"]
    SPRINGORM["Spring ORM\n(Hibernate/JPA mgmt)"]
    JPA["JPA\n(Java Persistence API)"]
    SPRINGDATA["Spring Data JPA\n(Auto query gen, JpaRepository, etc.)"]

    JDBC --> ORM
    JDBC --> SPRINGJDBC
    ORM --> HIBERNATE
    SPRINGJDBC --> SPRINGORM
    HIBERNATE --> JPA
    SPRINGORM --> SPRINGDATA
    JPA --> SPRINGDATA
```
> [!todo]
> Implementation
> - Configure `JdbcTemplate`  
>- Run simple queries, inserts, updates  
>- Map `ResultSet` to objects (`RowMapper`)

#conceptual