
> [!definition]
> Contents**Unidirectional**: Only one entity has a field pointing to the other
**Bidirectional**: Both entities have fields pointing to each other

## Practical Implications

### Navigation Capabilities
##### Unidirectional
```java
// You can only navigate ONE way
Post post = postRepository.findById(1L);
User author = post.getAuthor(); // ✅ This works

User user = userRepository.findById(1L);
List<Post> posts = user.getPosts(); // ❌ This doesn't work - no field exists!
```
##### Bidirectional
```java
// You can navigate BOTH ways
Post post = postRepository.findById(1L);
User author = post.getUser(); // ✅ This works

User user = userRepository.findById(1L);
List<Post> posts = user.getPosts(); // ✅ This also works!
```

### Querying Flexibility
##### Unidirectional `@ManyToOne`
```java
// Can only query from Post to User
public interface PostRepository extends JpaRepository<Post, Long> {
    List<Post> findByUser(User user); // ✅ Works
    List<Post> findByUserId(Long userId); // ✅ Works
}

public interface UserRepository extends JpaRepository<User, Long> {
    List<User> findByPosts(Post post); // ❌ No such field in User!
}
```
##### Bidirectional
```java
// Can query from both sides
public interface PostRepository extends JpaRepository<Post, Long> {
    List<Post> findByUser(User user); // ✅ Works
}

public interface UserRepository extends JpaRepository<User, Long> {
    List<User> findByPostsTitleContaining(String title); // ✅ Also works!
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
### Schema Generation Differences

**Unidirectional `@OneToMany` (default):**
```sql
-- Creates a JOIN TABLE (often unwanted)
CREATE TABLE user_posts (
    user_id BIGINT,
    post_id BIGINT
);
```

**Unidirectional `@OneToMany` with @JoinColumn:**
```sql
-- Foreign key in the "many" table (usually preferred)
CREATE TABLE post (
    id BIGINT,
    user_id BIGINT  -- Foreign key to user
);
```
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

#conceptual 