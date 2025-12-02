
> [!definition]
> **Unidirectional**: exist when only one entity maintains a reference to another entity. This means you can navigate from one side to the other, but not both ways.
**Bidirectional**: Both entities have fields pointing to each other.

The critical distinction to understand is that **databases only support unidirectional foreign key relationships**. When you create a bidirectional relationship in JPA, you're actually creating an object-oriented illusion on top of a unidirectional database structure. J

PA uses the `mappedBy` attribute to tell Hibernate which side of the relationship the foreign key column should reside in, **in case of bidirectional only**.​

The most practical way to implement `@OneToMany` relationships is bidirectional. The `@ManyToOne` side is always the owning side.

# Unidirectional
## One-to-One
Creates a Foreign Key (FK) in the current table.    
**`@JoinColumn` is Optional.** If used: You define the name of the FK column.

If omitted: JPA generates a default name (e.g., `targetEntity_id`).

``` Java
@Entity
public class User {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    
    //This is the owning side
    @OneToOne
    @JoinColumn(name = "profile_id") //optional
    private Profile profile;
}

@Entity
public class Profile {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    // No reference back to User
}

```
## One-to-Many 
This behavior depends on how you configure it:

- **Without `@JoinColumn` (The Default):** JPA creates a separate **Join Table** (e.g., `Post_Comments`). This is often inefficient.
- **With `@JoinColumn` (The Best Practice):** JPA creates a Foreign Key in the _target_ table (the "Many" side).

``` Java
@Entity
public class Tutorial {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    
    @OneToMany
    @JoinColumn(name = "tutorial_id") //without this a join table is created
    private List<Comment> comments;
}

@Entity
public class Comment {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    // No reference back to Tutorial
}
```

## Many-to-One 
Creates a Foreign Key in the current table.
**`@JoinColumn`** is optional, used to customize the FK column name.
## Many-to-Many
**Behavior:** Always creates a separate **Join Table**.

``` Java
@Entity
public class Student {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    
    @ManyToMany
    @JoinTable(
        name = "student_course",
        joinColumns = 
	        @JoinColumn(name = "student_id"),
        inverseJoinColumns = 
	        @JoinColumn(name = "course_id")
    )
    private List<Course> courses;
}

@Entity
public class Course {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    // No reference back to Student
}

```

# Bidirectional
**Owning Side:** Contains the Foreign Key (or holds the Join Table).
**Inverse Side:** Uses `mappedBy`. It basically says: _"I don't manage the database column; go look at the field named 'X' on the other class."_

If `mappedBy` is not specified JPA exhibits weird behaviour such as creating two join tables.

One-to-Many / Many-to-One
- **Owner (`@ManyToOne`):** This is **always** the owning side because the FK physically lives here. It does not have a `mappedBy` option.
- **Inverse (`@OneToMany`):** Must use `mappedBy`.

> [!caution]
> `@JoinColumn` should never be used on the `mappedBy` side. Otherwise a duplicate foreign key is created
## Practical Implications
### Navigation Capabilities
```java
// You can only navigate ONE way
Post post = postRepository.findById(1L);
User author = post.getAuthor(); // ✅ This works

User user = userRepository.findById(1L);
List<Post> posts = user.getPosts(); // ❌ This doesn't work in uni- no field exists!

//Works both in bi
```
### Querying Flexibility
```java
// Can only query from Post to User
public interface PostRepository extends JpaRepository<Post, Long> {
    List<Post> findByUser(User user); // ✅ Works
    List<Post> findByUserId(Long userId); // ✅ Works
}

public interface UserRepository extends JpaRepository<User, Long> {
    List<User> findByPosts(Post post); // ❌ Only possible in bidirectional
}
```
### JSON Serialization Behavior
##### Unidirectional
```java
// When serializing to JSON, you get clean, one-way relationships
Post post = // {id: 1, title: "Hello", user: {id: 1, name: "John"}}
User user = // {id: 1, name: "John"} - no post data!
```
##### Bidirectional
```java
// Can cause infinite recursion in JSON!
Post post = // {id: 1, title: "Hello", user: {id: 1, name: "John", posts: [{id: 1, ...}]}}
// This creates circular reference: Post -> User -> Post -> User -> ...
```

**Solution:** You need `@JsonIgnore` or DTOs:
```java
@Entity
public class User {
    @OneToMany(mappedBy = "user")
    @JsonIgnore // Prevents infinite recursion
    private List<Post> posts;
}
```

### Performance Implications

##### Unidirectional `@OneToMany` (with @JoinColumn)
```java
// Adding to collection causes extra UPDATE
user.getPosts().add(newPost);
// Hibernate: INSERT INTO post (title) VALUES (?)
// Hibernate: UPDATE post SET user_id = ? WHERE id = ?
```

##### Bidirectional
```java
// More efficient - single INSERT
newPost.setUser(user);
// Hibernate: INSERT INTO post (title, user_id) VALUES (?, ?)
```

### Data Consistency Risks

##### Unidirectional
```java
// Simpler - less chance of inconsistency
post.setUser(user);
// Only one reference to maintain
```

##### Bidirectional
```java
// Risk of inconsistent state if you don't update both sides
post.setUser(user);
// Forgot: user.getPosts().add(post);
// Now in-memory state is inconsistent!
```

you can read more about this issue in the [[Appendices/Appendix#Bidirectional Inconsistency Issue|Appendix]]
## When to Choose Which

**Choose Unidirectional when:**
- Simple relationship, only need one-way navigation
- Avoiding JSON serialization complexity
- Team is new to JPA
- Performance is critical (use `@ManyToOne`)

**Choose Bidirectional when:**
- Need to navigate both ways in business logic
- Complex queries from both sides
- Willing to manage consistency carefully
- Using DTOs to avoid JSON issues

The key implication is that "awareness" determines **what you can do** with your entities in terms of navigation, querying, and relationships management.

> [!summary]
> In unidirectional the referencing side is the owning side except in `OneToMany` either a join table, or by specifying `@JoinColumn` to make the many side the owning side with a join column.
> In bidirectional relations you must use `mappedBy` to determine the inverse side. 
> 	`@JoinColumn` If specified it need to be on the owning side to override the join column name.
> 	`@ManyToOne` doesn’t support `mappedBy` attribute
> 	


#conceptual 