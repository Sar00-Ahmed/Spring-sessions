# Infinite Recursion during Serialisation In bidirectional relations

```java
@Entity
public class Author {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    private String name;
    
    @OneToMany(mappedBy = "author", cascade = CascadeType.ALL)
    private List<Book> books = new ArrayList<>();
}

@Entity
public class Book {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    private String title;
    
    @ManyToOne
    @JoinColumn(name = "author_id")
    private Author author;
}
```

When the controller tries to map from or to JSON, serialising an `Author`, Jackson tries to serialise `books`, each `Book` tries to serialise its `author`, and so on… 

```json
// Infinite recursion!
{
  "id": 1,
  "name": "J.K. Rowling",
  "books": [
    {
      "id": 1,
      "title": "Harry Potter",
      "author": {
        "id": 1,
        "name": "J.K. Rowling",
        "books": [
          {
            "id": 1,
            "title": "Harry Potter",
            "author": {
              "id": 1,
              "name": "J.K. Rowling",
              "books": [...]
            }
          }
        ]
      }
    }
  ]
}
```

thus throwing this exception in your logs. notice the keywords:
**Could not write JSON: Infinite recursion (`StackOverflowError`)**

``` logs
2023-10-15 14:30:22.ERROR 12345 --- [nio-8080-exec-1] o.a.c.c.C.[.[.[/].[dispatcherServlet]    : Servlet.service() for servlet [dispatcherServlet] in context with path [] threw exception [Request processing failed; nested exception is org.springframework.http.converter.HttpMessageNotWritableException: Could not write JSON: Infinite recursion (StackOverflowError); nested exception is com.fasterxml.jackson.databind.JsonMappingException: Infinite recursion (StackOverflowError) (through reference chain: com.example.Author["books"]-org.hibernate.collection.internal.PersistentBag[0]-com.example.Book["author"]->com.example.Author["books"]-org.hibernate.collection.internal.PersistentBag[0]-com.example.Book["author"]->...)] with root cause

java.lang.StackOverflowError: null
	at com.fasterxml.jackson.databind.ser.std.BeanSerializer.serialize(BeanSerializer.java:151)
	at com.fasterxml.jackson.databind.ser.BeanPropertyWriter.serializeAsField(BeanPropertyWriter.java:727)
	// ... thousands of lines ...
 ```

> [!Attention]
> Note that this issue is not specific only to JSON serialisation, whenever object need to be serialised to any format so you will need to handle accordingly and may need different annotations than the one discussed.

# Solutions

### `@JsonIgnore` (Simple but Limited)
```java
@Entity
public class Book {
    // ... other fields
    
    @ManyToOne
    @JoinColumn(name = "author_id")
    @JsonIgnore // Prevents serialization of author in Book
    private Author author;
}
```

**Pros**: Simple, quick fix
**Cons**: You lose the relationship data entirely

### `@JsonManagedReference` and `@JsonBackReference`
```java
@Entity
public class Author {
    @Id
    private Long id;
    
    @OneToMany(mappedBy = "author")
    @JsonManagedReference // This is the "forward" part
    private List<Book> books = new ArrayList<>();
}

@Entity
public class Book {
    @Id
    private Long id;
    
    @ManyToOne
    @JoinColumn(name = "author_id")
    @JsonBackReference // This is the "back" part (won't be serialized)
    private Author author;
}
```

**Pros**: Clean, maintains relationship in one direction
**Cons**: Can't serialize both sides simultaneously

> [!attention]
> A problem occurs when using `@JsonBackReference` with collections. **`@JsonBackReference` cannot be used on collections** - it only works on single-valued properties. which mean you cannot use this fix for:
> - many to many relations
> - many to one relations when you want the list side to be ignored

### @JsonIgnoreProperties (More Flexible)
```java
@Entity
public class Author {
    @Id
    private Long id;
    
    @OneToMany(mappedBy = "author")
    @JsonIgnoreProperties("author") // Ignore author field in Book
    private List<Book> books = new ArrayList<>();
}

@Entity
public class Book {
    @Id
    private Long id;
    
    @ManyToOne
    @JoinColumn(name = "author_id")
    @JsonIgnoreProperties("books") // Ignore books field in Author
    private Author author;
}
```

**Pros**: More control, can be used selectively
**Cons**: Can become complex with deep relationships

### `@JsonIdentityInfo` - The Powerful Alternative
It uses object identity to break cycles.
```java
@Entity
@JsonIdentityInfo(
    generator = ObjectIdGenerators.PropertyGenerator.class,
    property = "id"
)
public class Author {
    @Id
    private Long id;
    private String name;
    
    @OneToMany(mappedBy = "author", cascade = CascadeType.ALL)
    private List<Book> books = new ArrayList<>();
}

@Entity
@JsonIdentityInfo(
    generator = ObjectIdGenerators.PropertyGenerator.class,
    property = "id"
)
public class Book {
    @Id
    private Long id;
    private String title;
    
    @ManyToOne
    @JoinColumn(name = "author_id")
    private Author author;
}
```

**With @JsonIdentityInfo:**
```json
{
  "id": 1,
  "name": "J.K. Rowling",
  "books": [
    {
      "id": 1,
      "title": "Harry Potter",
      "author": 1  // Just the ID instead of full object
    }
  ]
}
```

you can read more on it’s advanced usages in the [[Appendices/Appendix#Advanced `@JsonIdentityInfo` Scenarios|Appendix]]
# Other solutions

- DTO Pattern (Recommended for Complex Applications)
- Custom Serialisation
## Best Practices

### 1. **Use DTO Pattern for APIs**
```java
@Service
public class AuthorService {
    public AuthorDTO getAuthorWithBooks(Long id) {
        Author author = authorRepository.findById(id)
            .orElseThrow(() -> new EntityNotFoundException("Author not found"));
        return new AuthorDTO(author);
    }
}
```

### 2. **Lazy Loading Considerations**
```java
@Entity
public class Author {
    @OneToMany(mappedBy = "author", fetch = FetchType.LAZY)
    private List<Book> books = new ArrayList<>();
}

// Use @Transactional in service layer to handle lazy loading
@Service
@Transactional
public class AuthorService {
    public Author getAuthorWithBooks(Long id) {
        Author author = authorRepository.findById(id).orElseThrow();
        // Force initialization of lazy collection
        author.getBooks().size(); 
        return author;
    }
}
```

### 3. **Custom Query for Performance**
```java
public interface AuthorRepository extends JpaRepository<Author, Long> {
    @Query("SELECT a FROM Author a LEFT JOIN FETCH a.books WHERE a.id = :id")
    Optional<Author> findByIdWithBooks(@Param("id") Long id);
}
```

## Recommendation

For production applications, I recommend:
1. **Use DTO pattern** for most cases - it gives you full control and separates API contract from persistence model
2. **Use @JsonIgnoreProperties** for simple internal APIs
3. **Avoid bidirectional relationships** unless absolutely necessary - consider if you really need navigation in both directions

The DTO pattern, while requiring more code, provides the best separation of concerns and flexibility for API evolution.
## Comparison Table

| Annotation                                   | Pros                                                      | Cons                                                                    | Use Case                                                        |
| -------------------------------------------- | --------------------------------------------------------- | ----------------------------------------------------------------------- | --------------------------------------------------------------- |
| `@JsonManagedReference`/`@JsonBackReference` | Simple, explicit control                                  | Can't use on collections, only one bidirectional relationship per class | Simple parent-child relationships                               |
| `@JsonIdentityInfo`                          | Handles complex graphs, works with multiple relationships | Can be less explicit, references by ID only                             | Complex object graphs with multiple bidirectional relationships |
| `@JsonIgnore`                                | Very simple                                               | Loses data completely                                                   | When you don't need the relationship in JSON                    |
| `@JsonIgnoreProperties`                      | Flexible, explicit                                        | Manual configuration                                                    | Selective serialization control                                 |

# References
> [!cite]
> ```embed
title: "Fetching"
image: "data:image/svg+xml;base64,PHN2ZyBjbGFzcz0ibGRzLW1pY3Jvc29mdCIgd2lkdGg9IjgwcHgiICBoZWlnaHQ9IjgwcHgiICB4bWxucz0iaHR0cDovL3d3dy53My5vcmcvMjAwMC9zdmciIHZpZXdCb3g9IjAgMCAxMDAgMTAwIiBwcmVzZXJ2ZUFzcGVjdFJhdGlvPSJ4TWlkWU1pZCI+PGcgdHJhbnNmb3JtPSJyb3RhdGUoMCkiPjxjaXJjbGUgY3g9IjgxLjczNDEzMzYxMTY0OTQxIiBjeT0iNzQuMzUwNDU3MTYwMzQ4ODIiIGZpbGw9IiNlMTViNjQiIHI9IjUiIHRyYW5zZm9ybT0icm90YXRlKDM0MC4wMDEgNDkuOTk5OSA1MCkiPgogIDxhbmltYXRlVHJhbnNmb3JtIGF0dHJpYnV0ZU5hbWU9InRyYW5zZm9ybSIgdHlwZT0icm90YXRlIiBjYWxjTW9kZT0ic3BsaW5lIiB2YWx1ZXM9IjAgNTAgNTA7MzYwIDUwIDUwIiB0aW1lcz0iMDsxIiBrZXlTcGxpbmVzPSIwLjUgMCAwLjUgMSIgcmVwZWF0Q291bnQ9ImluZGVmaW5pdGUiIGR1cj0iMS41cyIgYmVnaW49IjBzIj48L2FuaW1hdGVUcmFuc2Zvcm0+CjwvY2lyY2xlPjxjaXJjbGUgY3g9Ijc0LjM1MDQ1NzE2MDM0ODgyIiBjeT0iODEuNzM0MTMzNjExNjQ5NDEiIGZpbGw9IiNmNDdlNjAiIHI9IjUiIHRyYW5zZm9ybT0icm90YXRlKDM0OC4zNTIgNTAuMDAwMSA1MC4wMDAxKSI+CiAgPGFuaW1hdGVUcmFuc2Zvcm0gYXR0cmlidXRlTmFtZT0idHJhbnNmb3JtIiB0eXBlPSJyb3RhdGUiIGNhbGNNb2RlPSJzcGxpbmUiIHZhbHVlcz0iMCA1MCA1MDszNjAgNTAgNTAiIHRpbWVzPSIwOzEiIGtleVNwbGluZXM9IjAuNSAwIDAuNSAxIiByZXBlYXRDb3VudD0iaW5kZWZpbml0ZSIgZHVyPSIxLjVzIiBiZWdpbj0iLTAuMDYyNXMiPjwvYW5pbWF0ZVRyYW5zZm9ybT4KPC9jaXJjbGU+PGNpcmNsZSBjeD0iNjUuMzA3MzM3Mjk0NjAzNiIgY3k9Ijg2Ljk1NTE4MTMwMDQ1MTQ3IiBmaWxsPSIjZjhiMjZhIiByPSI1IiB0cmFuc2Zvcm09InJvdGF0ZSgzNTQuMjM2IDUwIDUwKSI+CiAgPGFuaW1hdGVUcmFuc2Zvcm0gYXR0cmlidXRlTmFtZT0idHJhbnNmb3JtIiB0eXBlPSJyb3RhdGUiIGNhbGNNb2RlPSJzcGxpbmUiIHZhbHVlcz0iMCA1MCA1MDszNjAgNTAgNTAiIHRpbWVzPSIwOzEiIGtleVNwbGluZXM9IjAuNSAwIDAuNSAxIiByZXBlYXRDb3VudD0iaW5kZWZpbml0ZSIgZHVyPSIxLjVzIiBiZWdpbj0iLTAuMTI1cyI+PC9hbmltYXRlVHJhbnNmb3JtPgo8L2NpcmNsZT48Y2lyY2xlIGN4PSI1NS4yMjEwNDc2ODg4MDIwNyIgY3k9Ijg5LjY1Nzc5NDQ1NDk1MjQxIiBmaWxsPSIjYWJiZDgxIiByPSI1IiB0cmFuc2Zvcm09InJvdGF0ZSgzNTcuOTU4IDUwLjAwMDIgNTAuMDAwMikiPgogIDxhbmltYXRlVHJhbnNmb3JtIGF0dHJpYnV0ZU5hbWU9InRyYW5zZm9ybSIgdHlwZT0icm90YXRlIiBjYWxjTW9kZT0ic3BsaW5lIiB2YWx1ZXM9IjAgNTAgNTA7MzYwIDUwIDUwIiB0aW1lcz0iMDsxIiBrZXlTcGxpbmVzPSIwLjUgMCAwLjUgMSIgcmVwZWF0Q291bnQ9ImluZGVmaW5pdGUiIGR1cj0iMS41cyIgYmVnaW49Ii0wLjE4NzVzIj48L2FuaW1hdGVUcmFuc2Zvcm0+CjwvY2lyY2xlPjxjaXJjbGUgY3g9IjQ0Ljc3ODk1MjMxMTE5NzkzIiBjeT0iODkuNjU3Nzk0NDU0OTUyNDEiIGZpbGw9IiM4NDliODciIHI9IjUiIHRyYW5zZm9ybT0icm90YXRlKDM1OS43NiA1MC4wMDY0IDUwLjAwNjQpIj4KICA8YW5pbWF0ZVRyYW5zZm9ybSBhdHRyaWJ1dGVOYW1lPSJ0cmFuc2Zvcm0iIHR5cGU9InJvdGF0ZSIgY2FsY01vZGU9InNwbGluZSIgdmFsdWVzPSIwIDUwIDUwOzM2MCA1MCA1MCIgdGltZXM9IjA7MSIga2V5U3BsaW5lcz0iMC41IDAgMC41IDEiIHJlcGVhdENvdW50PSJpbmRlZmluaXRlIiBkdXI9IjEuNXMiIGJlZ2luPSItMC4yNXMiPjwvYW5pbWF0ZVRyYW5zZm9ybT4KPC9jaXJjbGU+PGNpcmNsZSBjeD0iMzQuNjkyNjYyNzA1Mzk2NDE1IiBjeT0iODYuOTU1MTgxMzAwNDUxNDciIGZpbGw9IiNlMTViNjQiIHI9IjUiIHRyYW5zZm9ybT0icm90YXRlKDAuMTgzNTUyIDUwIDUwKSI+CiAgPGFuaW1hdGVUcmFuc2Zvcm0gYXR0cmlidXRlTmFtZT0idHJhbnNmb3JtIiB0eXBlPSJyb3RhdGUiIGNhbGNNb2RlPSJzcGxpbmUiIHZhbHVlcz0iMCA1MCA1MDszNjAgNTAgNTAiIHRpbWVzPSIwOzEiIGtleVNwbGluZXM9IjAuNSAwIDAuNSAxIiByZXBlYXRDb3VudD0iaW5kZWZpbml0ZSIgZHVyPSIxLjVzIiBiZWdpbj0iLTAuMzEyNXMiPjwvYW5pbWF0ZVRyYW5zZm9ybT4KPC9jaXJjbGU+PGNpcmNsZSBjeD0iMjUuNjQ5NTQyODM5NjUxMTc2IiBjeT0iODEuNzM0MTMzNjExNjQ5NDEiIGZpbGw9IiNmNDdlNjAiIHI9IjUiIHRyYW5zZm9ybT0icm90YXRlKDEuODY0NTcgNTAgNTApIj4KICA8YW5pbWF0ZVRyYW5zZm9ybSBhdHRyaWJ1dGVOYW1lPSJ0cmFuc2Zvcm0iIHR5cGU9InJvdGF0ZSIgY2FsY01vZGU9InNwbGluZSIgdmFsdWVzPSIwIDUwIDUwOzM2MCA1MCA1MCIgdGltZXM9IjA7MSIga2V5U3BsaW5lcz0iMC41IDAgMC41IDEiIHJlcGVhdENvdW50PSJpbmRlZmluaXRlIiBkdXI9IjEuNXMiIGJlZ2luPSItMC4zNzVzIj48L2FuaW1hdGVUcmFuc2Zvcm0+CjwvY2lyY2xlPjxjaXJjbGUgY3g9IjE4LjI2NTg2NjM4ODM1MDYiIGN5PSI3NC4zNTA0NTcxNjAzNDg4NCIgZmlsbD0iI2Y4YjI2YSIgcj0iNSIgdHJhbnNmb3JtPSJyb3RhdGUoNS40NTEyNiA1MCA1MCkiPgogIDxhbmltYXRlVHJhbnNmb3JtIGF0dHJpYnV0ZU5hbWU9InRyYW5zZm9ybSIgdHlwZT0icm90YXRlIiBjYWxjTW9kZT0ic3BsaW5lIiB2YWx1ZXM9IjAgNTAgNTA7MzYwIDUwIDUwIiB0aW1lcz0iMDsxIiBrZXlTcGxpbmVzPSIwLjUgMCAwLjUgMSIgcmVwZWF0Q291bnQ9ImluZGVmaW5pdGUiIGR1cj0iMS41cyIgYmVnaW49Ii0wLjQzNzVzIj48L2FuaW1hdGVUcmFuc2Zvcm0+CjwvY2lyY2xlPjxhbmltYXRlVHJhbnNmb3JtIGF0dHJpYnV0ZU5hbWU9InRyYW5zZm9ybSIgdHlwZT0icm90YXRlIiBjYWxjTW9kZT0ic3BsaW5lIiB2YWx1ZXM9IjAgNTAgNTA7MCA1MCA1MCIgdGltZXM9IjA7MSIga2V5U3BsaW5lcz0iMC41IDAgMC41IDEiIHJlcGVhdENvdW50PSJpbmRlZmluaXRlIiBkdXI9IjEuNXMiPjwvYW5pbWF0ZVRyYW5zZm9ybT48L2c+PC9zdmc+"
description: "Fetching https://codingnomads.com/spring-json-serialization#jackson-json-mapping"
url: "https://codingnomads.com/spring-json-serialization#jackson-json-mapping"
favicon: ""
>```

> [!cite]
> 
>```embed
title: "Jackson Annotations For Java Application - GeeksforGeeks"
image: "https://media.geeksforgeeks.org/wp-content/cdn-uploads/gfg_200x200-min.png"
description: "Your All-in-One Learning Portal: GeeksforGeeks is a comprehensive educational platform that empowers learners across domains-spanning computer science and programming, school education, upskilling, commerce, software tools, competitive exams, and more."
url: "https://www.geeksforgeeks.org/java/jackson-annotations-for-java-application/"
favicon: ""
aspectRatio: "100"
>```


#core 