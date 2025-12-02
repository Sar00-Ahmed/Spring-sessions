- `@OneToOne`
- `@ManyToOne`
- `@OneToMany`
- `@ManyToMany`

Three more important parameters that are also available to customize your entity relationships are:
    - `fetch` dictates how items are retrieved from the database: eagerly (pre-emptively) or lazily (when they are needed).
    - `optional` allows you to make the relationship optional or required. equivalent to mandatory concept in relations 
    - `cascade` dictates which actions also get applied to child entities.

# One-to-one

``` Java
@Entity
@Table(name = "drivers")
@NoArgsConstructor
@Getter
@Setter
public class Driver {

    @Id
    @GeneratedValue(strategy = GenerationType.AUTO)
    private Long id;

    @Column(nullable = false, updatable = false)
    private String name;

	// Driver is the owning side (it has the foreign key)
    @OneToOne 
    private Car car;
}

```

``` Java
@Entity
@Table(name = "cars")
@NoArgsConstructor
@Getter
@Setter
public class Car {

    @Id
    @GeneratedValue(strategy = GenerationType.AUTO)
    private Long id;

    @Column(nullable = false)
    private String brand;

    @Column(name = "horsepower")
    private int horsepower;

} 

```
###### Defaults
1. The name of the join column is the name of the field referencing the child entity, with **_id** added to the end. For this example, it would be `car_id`.
2. One-to-one relationships are optional by default.
3. Eager fetching
4. No actions cascade or propagate through to children.

##### Bidirectional
``` Java
@Entity
@Table(name = "cars")
@NoArgsConstructor
@Getter
@Setter
public class Car {
	//code

    // note that the annotation and field are new
    @OneToOne(
            // indicates that this is the child side of a 
            // relationship and refers to the field in the Driver 
            // class that defines the relationship there
            mappedBy = "car"
    )
    private Driver driver;
}

```

# One-to-many & Many-to-one
## One-to-many
1. Annotate the field with `@OneToMany`: The annotation is placed on the "one" side of the relationship. It indicates that one instance of this entity (the parent) can be associated with multiple instances of another entity (the children).
    
2. Ensure the field is a type of Collection: The field that is annotated with @OneToMany should be a `List<T>`, `Set<T>`, or any other `Collection<T>`.
    
3. Optional - Set up a join column: **The default behavior for a @OneToMany relationship is to use a join table**, but you can optionally use the `@JoinColumn` annotation to use a join column instead. 

> [!warning]
> using `@oneToMany` in a unidirectional relation with `@JoinColumn` can cause performance issues and is not the recommended approach you can read more about this in the [[Sessions/4_Data persistence/Appendix#Unidirectional `@OneToMany` performance|Appendix]]

``` Java
@Entity
@Getter
@Setter
@NoArgsConstructor
public class Post {

    @Id
    @GeneratedValue
    private Long id;

    @Column(nullable = false, updatable = false)
    private String username;

    @Column(nullable = false)
    private String content;

    @OneToMany(cascade = CascadeType.ALL, fetch = FetchType.LAZY)
	@JoinColumn(name = "post_id")
    private List<Comment> comments;
}
```

``` Java
@Entity
@Getter
@Setter
@NoArgsConstructor
public class Comment {

    @Id
    @GeneratedValue
    private Long id;

    @Column(nullable = false, updatable = false)
    private String username;

    @Column(nullable = false)
    private String content;
}

```

> [!important]
> It was previously stated that the owning side of a relationship typically contains the join column in its database table. However,  `@OneToMany` relationships **are an exception**, where the **join column is found in the table of the entity on the "many" side**, in this case the Comments table.

## Many-to-one
``` Java
@Entity
@Getter
@Setter
@NoArgsConstructor
public class Post {

    @Id
    @GeneratedValue
    private Long id;

    @Column(nullable = false, updatable = false)
    private String username;

    @Column(nullable = false)
    private String content;
}
```

```java
@Entity
@Getter
@Setter
@NoArgsConstructor
public class Comment {

    @Id
    @GeneratedValue
    private Long id;

    @Column(nullable = false, updatable = false)
    private String username;

    @Column(nullable = false)
    private String content;

    @ManyToOne(cascade = CascadeType.ALL, optional = false)
    //no need to configure join column
    private Post post;
}
```
## Bidirectional
**@ManyToOne Side**: This is usually considered the "parent" or "owning" side of the relationship, since this entity will have a foreign key column in its table that points to the primary key of the entity on the "one" side. **`@ManyToOne` does not offer a `mappedBy` attribute.**
    
 **@OneToMany Side**: This is usually the inverse or "child" side of the relationship, using the `mappedBy` attribute to indicate such. 

``` Java
@Entity
@Getter
@Setter
@NoArgsConstructor
public class Post {

	//code

    // this annotation references the configuration 
    // on the post field in the Comment class
    @OneToMany(mappedBy = "post")
    private Set<Comment> comments;
}

@Entity
@Getter
@Setter
@NoArgsConstructor
public class Comment {

	//code

    @ManyToOne(
            cascade = CascadeType.ALL,
            optional = false
    )
    private Post post;
}
```

> [!attention]
> When saving changes, **actions should take place from the owning side** rather than directly modifying the `Set<Comment>` in Post. This will ensure consistent and reliable synchronization of the relationship in the database.
> ```java
// correct approach
// setting the relationship on the owning-side
comment.setPost(post);
saveComment(comment);
>
>// instead of
>post.getComments().add(post);
>savePost(post);
>```
# Many-to-many
## Unidirectional
``` Java
@Entity
@Getter
@Setter
@NoArgsConstructor
public class Post {

	//code

   // set up many-to-many relationship with the Location class
   @ManyToMany(fetch = FetchType.EAGER, cascade = CascadeType.ALL)
   private Set<Location> locations;
}

```

``` Java
@Entity
@Getter
@Setter
@NoArgsConstructor
public class Location {

   @Id
   @GeneratedValue
   private Long id;

   @Column(nullable = false)
   private String name;

   @Column(nullable = false)
   private Long latitude;

   @Column(nullable = false)
   private Long longitude;
}

```

## Bidirectional
``` Java
@Entity
@Getter
@Setter
@NoArgsConstructor
public class Post {

	//code

   @ManyToMany(fetch = FetchType.EAGER, cascade = CascadeType.ALL)
   private Set<Location> locations;
}

```

``` Java
@Entity
@Getter
@Setter
@NoArgsConstructor
public class Location {

	//code

   // this annotation refers to the 
   // locations field in the Post class
   @ManyToMany(mappedBy = "locations")
   private Set<Post> posts;
}

```

## Join table

``` Java
@ManyToMany
// start join table configuration using @JoinTable
@JoinTable(
        // change join table name
        name = "post_location_join_table",
        // specify a column named post_username referencing 
        // the username column in the posts table
        joinColumns = @JoinColumn(
                name = "post_username",
                referencedColumnName = "username")
)
private Set<Location> locations;

```

``` Java
@ManyToMany
// start join table configuration using @JoinTable
@JoinTable(
        //change join table name
        name = "post_location_join_table",
        // specify a column named location_latitude referencing 
        // the latitude column in the locations table instead of the id
        inverseJoinColumns = @JoinColumn(
                name = "location_latitude",
                referencedColumnName = "latitude"))
private Set<Location> locations;

```

> [!todo]
> Research
> Lets say we have two entities students and courses with many to many relation enrolment. This relation has the attributes: enrolment date & grade.
> - How can we represent this data? 
> - How do we make a foreign key to be the primary key?

# `@JoinColumn` configs
``` Java
@JoinColumn(name = "user_id")

//Indicates whether the join column can contain duplicate values.
@JoinColumn(unique = true)

@JoinColumn(name = "final_join_column", updatable = false)

//cases where advanced column configuration (beyond the capabilities of JPA/Hibernate) are required.
@JoinColumn(columnDefinition = "VARCHAR(255) DEFAULT 'N/A'")

@JoinColumn(name = "required_join_column", nullable = false)

//By default, the join column references the primary key column in the other table.
@JoinColumn(name = "address_zip_code", referencedColumnName = "zipcode")
private Address address;

```
# Summary
> [!summary]
> Contents- In any bidirectional relation you need to use mapped by to determine the non-owning side. `OneToMany(mappedby ="owninSidePropertyName")`
>
>- you can add `@JoinColumn` to set custom name, **optional**.
>- Join table is used in many to many also op
>
>
|           | Uni                                                                                  | Bi                                                                   |
| --------- | ------------------------------------------------------------------------------------ | -------------------------------------------------------------------- |
| OneToOne  | - In unidirectional, the aware side is the **owning side**<br>                       | the inverse side of mappedby                                         |
| OneToMany | - In unidirectional you need to add  `@JoinColumn` to **avoid creating join table**. | many side has foreign key if join table was not created              |
| ManyToOne | will have the foreign key                                                            | - **Does not have mapped by** option thus must always be owning side |

# References
> [!cite]
> Has excellent detailed explanation on spring relations
> ```embed
title: "Spring Data JPA Entity Relationships"
image: "https://codingnomads.com/images/27977d60-c4a8-40ba-c3c7-c75c67db7900/1200x630"
description: "Explore Spring Data JPA entity relationships and annotations such as @OneToOne, @ManyToOne, @OneToMany, and @ManyToMany. "
url: "https://codingnomads.com/spring-data-jpa-entity-relationships"
favicon: ""
aspectRatio: "52.5"
>```

> [!cite]
> ```embed
title: "@MapsId Annotation in Hibernate | Baeldung"
image: "https://www.baeldung.com/wp-content/uploads/2021/01/On-Baeldung-5.jpg"
description: "Learn how to use the Hibernate annotation @MapsId to implement the shared primary key strategy."
url: "https://www.baeldung.com/hibernate-mapsid-annotation"
favicon: ""
aspectRatio: "52.33333333333333"
>```

> [!cite]
> Cascade types
>```embed
title: "Fetching"
image: "data:image/svg+xml;base64,PHN2ZyBjbGFzcz0ibGRzLW1pY3Jvc29mdCIgd2lkdGg9IjgwcHgiICBoZWlnaHQ9IjgwcHgiICB4bWxucz0iaHR0cDovL3d3dy53My5vcmcvMjAwMC9zdmciIHZpZXdCb3g9IjAgMCAxMDAgMTAwIiBwcmVzZXJ2ZUFzcGVjdFJhdGlvPSJ4TWlkWU1pZCI+PGcgdHJhbnNmb3JtPSJyb3RhdGUoMCkiPjxjaXJjbGUgY3g9IjgxLjczNDEzMzYxMTY0OTQxIiBjeT0iNzQuMzUwNDU3MTYwMzQ4ODIiIGZpbGw9IiNlMTViNjQiIHI9IjUiIHRyYW5zZm9ybT0icm90YXRlKDM0MC4wMDEgNDkuOTk5OSA1MCkiPgogIDxhbmltYXRlVHJhbnNmb3JtIGF0dHJpYnV0ZU5hbWU9InRyYW5zZm9ybSIgdHlwZT0icm90YXRlIiBjYWxjTW9kZT0ic3BsaW5lIiB2YWx1ZXM9IjAgNTAgNTA7MzYwIDUwIDUwIiB0aW1lcz0iMDsxIiBrZXlTcGxpbmVzPSIwLjUgMCAwLjUgMSIgcmVwZWF0Q291bnQ9ImluZGVmaW5pdGUiIGR1cj0iMS41cyIgYmVnaW49IjBzIj48L2FuaW1hdGVUcmFuc2Zvcm0+CjwvY2lyY2xlPjxjaXJjbGUgY3g9Ijc0LjM1MDQ1NzE2MDM0ODgyIiBjeT0iODEuNzM0MTMzNjExNjQ5NDEiIGZpbGw9IiNmNDdlNjAiIHI9IjUiIHRyYW5zZm9ybT0icm90YXRlKDM0OC4zNTIgNTAuMDAwMSA1MC4wMDAxKSI+CiAgPGFuaW1hdGVUcmFuc2Zvcm0gYXR0cmlidXRlTmFtZT0idHJhbnNmb3JtIiB0eXBlPSJyb3RhdGUiIGNhbGNNb2RlPSJzcGxpbmUiIHZhbHVlcz0iMCA1MCA1MDszNjAgNTAgNTAiIHRpbWVzPSIwOzEiIGtleVNwbGluZXM9IjAuNSAwIDAuNSAxIiByZXBlYXRDb3VudD0iaW5kZWZpbml0ZSIgZHVyPSIxLjVzIiBiZWdpbj0iLTAuMDYyNXMiPjwvYW5pbWF0ZVRyYW5zZm9ybT4KPC9jaXJjbGU+PGNpcmNsZSBjeD0iNjUuMzA3MzM3Mjk0NjAzNiIgY3k9Ijg2Ljk1NTE4MTMwMDQ1MTQ3IiBmaWxsPSIjZjhiMjZhIiByPSI1IiB0cmFuc2Zvcm09InJvdGF0ZSgzNTQuMjM2IDUwIDUwKSI+CiAgPGFuaW1hdGVUcmFuc2Zvcm0gYXR0cmlidXRlTmFtZT0idHJhbnNmb3JtIiB0eXBlPSJyb3RhdGUiIGNhbGNNb2RlPSJzcGxpbmUiIHZhbHVlcz0iMCA1MCA1MDszNjAgNTAgNTAiIHRpbWVzPSIwOzEiIGtleVNwbGluZXM9IjAuNSAwIDAuNSAxIiByZXBlYXRDb3VudD0iaW5kZWZpbml0ZSIgZHVyPSIxLjVzIiBiZWdpbj0iLTAuMTI1cyI+PC9hbmltYXRlVHJhbnNmb3JtPgo8L2NpcmNsZT48Y2lyY2xlIGN4PSI1NS4yMjEwNDc2ODg4MDIwNyIgY3k9Ijg5LjY1Nzc5NDQ1NDk1MjQxIiBmaWxsPSIjYWJiZDgxIiByPSI1IiB0cmFuc2Zvcm09InJvdGF0ZSgzNTcuOTU4IDUwLjAwMDIgNTAuMDAwMikiPgogIDxhbmltYXRlVHJhbnNmb3JtIGF0dHJpYnV0ZU5hbWU9InRyYW5zZm9ybSIgdHlwZT0icm90YXRlIiBjYWxjTW9kZT0ic3BsaW5lIiB2YWx1ZXM9IjAgNTAgNTA7MzYwIDUwIDUwIiB0aW1lcz0iMDsxIiBrZXlTcGxpbmVzPSIwLjUgMCAwLjUgMSIgcmVwZWF0Q291bnQ9ImluZGVmaW5pdGUiIGR1cj0iMS41cyIgYmVnaW49Ii0wLjE4NzVzIj48L2FuaW1hdGVUcmFuc2Zvcm0+CjwvY2lyY2xlPjxjaXJjbGUgY3g9IjQ0Ljc3ODk1MjMxMTE5NzkzIiBjeT0iODkuNjU3Nzk0NDU0OTUyNDEiIGZpbGw9IiM4NDliODciIHI9IjUiIHRyYW5zZm9ybT0icm90YXRlKDM1OS43NiA1MC4wMDY0IDUwLjAwNjQpIj4KICA8YW5pbWF0ZVRyYW5zZm9ybSBhdHRyaWJ1dGVOYW1lPSJ0cmFuc2Zvcm0iIHR5cGU9InJvdGF0ZSIgY2FsY01vZGU9InNwbGluZSIgdmFsdWVzPSIwIDUwIDUwOzM2MCA1MCA1MCIgdGltZXM9IjA7MSIga2V5U3BsaW5lcz0iMC41IDAgMC41IDEiIHJlcGVhdENvdW50PSJpbmRlZmluaXRlIiBkdXI9IjEuNXMiIGJlZ2luPSItMC4yNXMiPjwvYW5pbWF0ZVRyYW5zZm9ybT4KPC9jaXJjbGU+PGNpcmNsZSBjeD0iMzQuNjkyNjYyNzA1Mzk2NDE1IiBjeT0iODYuOTU1MTgxMzAwNDUxNDciIGZpbGw9IiNlMTViNjQiIHI9IjUiIHRyYW5zZm9ybT0icm90YXRlKDAuMTgzNTUyIDUwIDUwKSI+CiAgPGFuaW1hdGVUcmFuc2Zvcm0gYXR0cmlidXRlTmFtZT0idHJhbnNmb3JtIiB0eXBlPSJyb3RhdGUiIGNhbGNNb2RlPSJzcGxpbmUiIHZhbHVlcz0iMCA1MCA1MDszNjAgNTAgNTAiIHRpbWVzPSIwOzEiIGtleVNwbGluZXM9IjAuNSAwIDAuNSAxIiByZXBlYXRDb3VudD0iaW5kZWZpbml0ZSIgZHVyPSIxLjVzIiBiZWdpbj0iLTAuMzEyNXMiPjwvYW5pbWF0ZVRyYW5zZm9ybT4KPC9jaXJjbGU+PGNpcmNsZSBjeD0iMjUuNjQ5NTQyODM5NjUxMTc2IiBjeT0iODEuNzM0MTMzNjExNjQ5NDEiIGZpbGw9IiNmNDdlNjAiIHI9IjUiIHRyYW5zZm9ybT0icm90YXRlKDEuODY0NTcgNTAgNTApIj4KICA8YW5pbWF0ZVRyYW5zZm9ybSBhdHRyaWJ1dGVOYW1lPSJ0cmFuc2Zvcm0iIHR5cGU9InJvdGF0ZSIgY2FsY01vZGU9InNwbGluZSIgdmFsdWVzPSIwIDUwIDUwOzM2MCA1MCA1MCIgdGltZXM9IjA7MSIga2V5U3BsaW5lcz0iMC41IDAgMC41IDEiIHJlcGVhdENvdW50PSJpbmRlZmluaXRlIiBkdXI9IjEuNXMiIGJlZ2luPSItMC4zNzVzIj48L2FuaW1hdGVUcmFuc2Zvcm0+CjwvY2lyY2xlPjxjaXJjbGUgY3g9IjE4LjI2NTg2NjM4ODM1MDYiIGN5PSI3NC4zNTA0NTcxNjAzNDg4NCIgZmlsbD0iI2Y4YjI2YSIgcj0iNSIgdHJhbnNmb3JtPSJyb3RhdGUoNS40NTEyNiA1MCA1MCkiPgogIDxhbmltYXRlVHJhbnNmb3JtIGF0dHJpYnV0ZU5hbWU9InRyYW5zZm9ybSIgdHlwZT0icm90YXRlIiBjYWxjTW9kZT0ic3BsaW5lIiB2YWx1ZXM9IjAgNTAgNTA7MzYwIDUwIDUwIiB0aW1lcz0iMDsxIiBrZXlTcGxpbmVzPSIwLjUgMCAwLjUgMSIgcmVwZWF0Q291bnQ9ImluZGVmaW5pdGUiIGR1cj0iMS41cyIgYmVnaW49Ii0wLjQzNzVzIj48L2FuaW1hdGVUcmFuc2Zvcm0+CjwvY2lyY2xlPjxhbmltYXRlVHJhbnNmb3JtIGF0dHJpYnV0ZU5hbWU9InRyYW5zZm9ybSIgdHlwZT0icm90YXRlIiBjYWxjTW9kZT0ic3BsaW5lIiB2YWx1ZXM9IjAgNTAgNTA7MCA1MCA1MCIgdGltZXM9IjA7MSIga2V5U3BsaW5lcz0iMC41IDAgMC41IDEiIHJlcGVhdENvdW50PSJpbmRlZmluaXRlIiBkdXI9IjEuNXMiPjwvYW5pbWF0ZVRyYW5zZm9ybT48L2c+PC9zdmc+"
description: "Fetching https://www.baeldung.com/jpa-cascade-types"
url: "https://www.baeldung.com/jpa-cascade-types"
favicon: ""
>```

> [!cite]
> JPA CascadeType.REMOVE vs orphanRemoval
>```embed
title: "JPA CascadeType.REMOVE vs orphanRemoval | Baeldung"

description: "Learn about the difference between JPA CascadeType.REMOVE and orphanRemoval for deleting entities."
url: "https://www.baeldung.com/jpa-cascade-remove-vs-orphanremoval"
favicon: ""
aspectRatio: "52.3109243697479"
>```

#core