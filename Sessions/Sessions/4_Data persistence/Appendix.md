# `@column` properties

|**Property**|**Type**|**Default**|**Purpose / When to Use**|
|---|---|---|---|
|`name`|`String`|Field name|Custom column name (e.g., `@Column(name = "user_name")`)|
|`nullable`|`boolean`|`true`|Whether the column allows `NULL`. Set to `false` for `NOT NULL`.|
|`unique`|`boolean`|`false`|Adds a unique constraint on the column.|
|`length`|`int`|`255`|Maximum column length (for `String`/`char[]`). Applies to `VARCHAR`.|
|`precision`|`int`|`0` (provider default)|Total number of digits for numeric/decimal types.|
|`scale`|`int`|`0`|Number of digits to the right of the decimal point (used with `precision`).|
|`insertable`|`boolean`|`true`|If `false`, this column is **excluded** from SQL `INSERT` statements.|
|`updatable`|`boolean`|`true`|If `false`, this column is **excluded** from SQL `UPDATE` statements.|
|`columnDefinition`|`String`|(none)|Directly specify SQL column definition (DDL). Useful for DB-specific customization.|
|`table`|`String`|Primary table|If the entity maps to multiple tables (via `@SecondaryTable`), specify which table the column belongs to.|

# Date columns
`@Temporal` is one of those annotations that confuses many beginners, because **its importance changed after Java 8**.

### The Problem

- In **pre-Java 8**, the only date/time classes were `java.util.Date` and `java.util.Calendar`.
    
- But those classes don’t differentiate between **date**, **time**, and **timestamp** in a type-safe way — they just hold a moment in time (milliseconds since epoch).
    
- JPA therefore needs to know **how to map them to a database column**:
    
    - Store only the **date** part? (`DATE`)
        
    - Only the **time** part? (`TIME`)
        
    - Or the full timestamp? (`TIMESTAMP`)
        

### The Solution: `@Temporal`

`@Temporal` tells JPA how to persist a `Date` or `Calendar` field.

```java
@Temporal(TemporalType.DATE)
private Date birthDate;  // only yyyy-MM-dd, no time

@Temporal(TemporalType.TIME)
private Date wakeUpTime; // only hh:mm:ss

@Temporal(TemporalType.TIMESTAMP)
private Date createdAt;  // full date + time
```
## With Java 8+

Java 8 introduced the **java.time** API (`LocalDate`, `LocalDateTime`, `Instant`, etc.) which are **type-safe**:

|Java Type|Stored As|Need `@Temporal`?|
|---|---|---|
|`LocalDate`|SQL `DATE`|❌ No|
|`LocalTime`|SQL `TIME`|❌ No|
|`LocalDateTime`|SQL `TIMESTAMP`|❌ No|
|`Instant`|SQL `TIMESTAMP` (UTC)|❌ No|

So **you do not need `@Temporal` anymore** if you use Java 8 date/time types — Hibernate/JPA knows how to map them automatically.


> [!tip]
> - Only use `@Temporal` if you’re working with **legacy code** that uses `java.util.Date` or `Calendar`.
>- For new projects, **prefer Java 8+ date/time types** and skip `@Temporal` entirely.

- - -
# Bidirectional Inconsistency Issue
## What is Inconsistent State?

**Inconsistent state** means that the two sides of the bidirectional relationship in your **Java memory** don't agree with each other, even though the **database** might be correct.
### Example 1: Basic Inconsistency
```java
User user = new User("John");
Post post = new Post("Hello World");

// ONLY setting one side
post.setUser(user); // Post knows about User
// Forgot: user.getPosts().add(post);

System.out.println(post.getUser().getName()); // "John" ✅
System.out.println(user.getPosts().size());   // 0 ❌ (Should be 1!)
```

### Example 2: After Database Save
```java
User user = userRepository.save(new User("John"));
Post post = new Post("Hello World");

// Only set one side
post.setUser(user);
postRepository.save(post);

// Now check both sides
User freshUser = userRepository.findById(user.getId()).get();
System.out.println(freshUser.getPosts().size()); // 1 ✅ (DB is correct)

System.out.println(user.getPosts().size()); // 0 ❌ (Our in-memory object is wrong!)
```

## Caused Issues

### Caching Problems
```java
// Scenario: Using @Cacheable or second-level cache
User user = userRepository.findById(1L); // Cached
Post post = new Post("New Post");
post.setUser(user);
postRepository.save(post);

// Later...
User cachedUser = userRepository.findById(1L); // Gets cached version
System.out.println(cachedUser.getPosts().size()); // 0 ❌ Cache has stale data
```

### Unexpected Behavior in Business Logic
```java
// In a service method
public void approveUserPosts(Long userId) {
    User user = userRepository.findById(userId);
    
    // This might return incomplete results!
    for (Post post : user.getPosts()) {
        post.setStatus(Status.APPROVED);
    }
    // Some posts might be missing from the collection!
}
```

### Transactional Boundary Issues
```java
@Transactional
public void createPostForUser(Long userId) {
    User user = userRepository.findById(userId);
    Post post = new Post("New Post");
    
    post.setUser(user); // Only set one side
    postRepository.save(post);
    
    // Within same transaction, the inconsistency causes bugs
    if (user.getPosts().contains(post)) { // This returns FALSE!
        // This code never executes
        sendNotification();
    }
}
```

### JSON Serialization Surprises
```java
@RestController
public class PostController {
    
    @PostMapping("/posts")
    public Post createPost(@RequestBody Post post) {
        // Only post.user is set, user.posts is empty
        Post saved = postRepository.save(post);
        
        return saved; // Serializes as: {user: {posts: []}} - looks wrong!
    }
}
```

## The Database vs Memory Distinction

```java
User user = new User("John");
Post post1 = new Post("Post 1");
Post post2 = new Post("Post 2");

//-----------------------------------------------------
// SCENARIO A: Inconsistent in memory, consistent in DB
//-----------------------------------------------------
post1.setUser(user); // Only one side
postRepository.save(post1);
// MEMORY: user.posts = [], post1.user = John
// DATABASE: post1 has user_id = John's ID ✅

//-----------------------------------------------------
// SCENARIO B: Inconsistent everywhere  
//-----------------------------------------------------
user.getPosts().add(post2); // Only one side  
postRepository.save(post2);
// MEMORY: user.posts = [post2], post2.user = null  
// DATABASE: post2 has user_id = NULL ❌
```
## The Solution: Helper Methods
Always use helper methods to maintain consistency:

```java
@Entity
public class User {
    @OneToMany(mappedBy = "user")
    private List<Post> posts = new ArrayList<>();
    
    // Helper method to maintain consistency
    public void addPost(Post post) {
        posts.add(post);
        post.setUser(this);
    }
    
    public void removePost(Post post) {
        posts.remove(post);
        post.setUser(null);
    }
}

// Usage - always consistent!
User user = new User("John");
Post post = new Post("Hello");
user.addPost(post); // Sets both sides automatically

// Now both sides are consistent:
System.out.println(post.getUser() == user); // true
System.out.println(user.getPosts().contains(post)); // true
```

## Summary

**Inconsistent state** means your Java objects in memory don't reflect the true relationships, which causes:

1. **Wrong business logic results**
2. **Caching issues** 
3. **Transactional problems**
4. **Unexpected serialization behavior**
5. **Bugs that are hard to reproduce**

The database will eventually be consistent when you save, but your in-memory objects can be wrong, leading to bugs during the same transaction or when using cached data.

- - -
# Unidirectional `@OneToMany` performance
```java
@OneToMany
private List<Post> posts = new ArrayList<>();
```
**Result:** Creates a join table `user_posts` with `user_id` and `post_id` columns. 

When you add `@JoinColumn` to a `@OneToMany`, you're telling JPA: "Don't create a join table. Instead, use a foreign key column in the 'many' side table that points back to me."

**Example: User with a unidirectional collection of Posts**

```java
@Entity
public class User {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    private String name;

    // Unidirectional OneToMany WITH proper foreign key in Post table
    @OneToMany
    // This creates a 'user_id' column in the 'post' table
    @JoinColumn(name = "user_id") 
    private List<Post> posts = new ArrayList<>();
}

@Entity
public class Post {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    private String title;
    // No reference back to User - this is unidirectional from User side
}
```

**Resulting Schema:**
- `user` table: `id`, `name`
- `post` table: `id`, `title`, `user_id` (foreign key)

| Approach | Use When |
|----------|----------|
| **Unidirectional `@ManyToOne`** | ✅ **DEFAULT CHOICE** - You only need to navigate from "many" to "one" |
| **Unidirectional `@OneToMany` with `@JoinColumn`** | ✅ You only need to navigate from "one" to "many" AND can't modify the "many" entity |
| **Bidirectional with `mappedBy`** | ✅ **BEST FOR BIDIRECTIONAL** - You need navigation from both sides |
| **Unidirectional `@OneToMany` (default)** | ❌ **AVOID** - Creates inefficient join table |

### Performance Considerations

The unidirectional `@OneToMany` with `@JoinColumn` has one potential performance issue to be aware of:

**When adding new Posts to a User's collection:**
```java
User user = userRepository.findById(1L).orElseThrow();
Post newPost = new Post("My New Post");
user.getPosts().add(newPost); // This works...
userRepository.save(user);
```

Hibernate will actually execute:
1. `INSERT INTO post (title) VALUES ('My New Post')` - creates post with 
	`user_id = NULL`
2. `UPDATE post SET user_id = 1 WHERE id = ?` - updates the post to set the foreign key

This results in **two SQL statements** instead of one. With the bidirectional approach, you can do it in a single INSERT.

# Advanced `@JsonIdentityInfo` Scenarios

### Different ID Properties
```java
@Entity
@JsonIdentityInfo(
    generator = ObjectIdGenerators.PropertyGenerator.class,
    property = "authorId"  // Custom ID property
)
public class Author {
    @Id
    private Long authorId;
    private String name;
    
    @OneToMany(mappedBy = "author")
    private List<Book> books;
}

@Entity
@JsonIdentityInfo(
    generator = ObjectIdGenerators.PropertyGenerator.class,
    property = "isbn"  // Different ID property
)
public class Book {
    @Id
    private Long id;
    private String isbn;
    private String title;
    
    @ManyToOne
    private Author author;
}
```

### Using Scope
```java
@Entity
@JsonIdentityInfo(
    generator = ObjectIdGenerators.PropertyGenerator.class,
    property = "id",
    scope = Author.class  // Defines scope for object identity
)
public class Author {
    // ...
}

@Entity
@JsonIdentityInfo(
    generator = ObjectIdGenerators.PropertyGenerator.class,
    property = "id", 
    scope = Book.class
)
public class Book {
    // ...
}
```

### Global Configuration
```java
// You can also configure it globally
@JsonIdentityInfo(generator = ObjectIdGenerators.PropertyGenerator.class, property = "id")
@MappedSuperclass
public abstract class BaseEntity {
    @Id
    private Long id;
    
    // getters/setters
}

@Entity
public class Author extends BaseEntity {
    // Inherits @JsonIdentityInfo
    private String name;
    
    @OneToMany(mappedBy = "author")
    private List<Book> books;
}

@Entity
public class Book extends BaseEntity {
    // Inherits @JsonIdentityInfo
    private String title;
    
    @ManyToOne
    private Author author;
}
```


# Common Database issues
## Polymorphic foreign keys and inheritence
#todo 