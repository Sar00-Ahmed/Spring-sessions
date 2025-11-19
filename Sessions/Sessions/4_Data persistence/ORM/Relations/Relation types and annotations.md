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
> using `@oneToMany` in a unidirectional relation with `@JoinColumn` can cause performance issues and is not the recommended approach you can read more about this in the [[Appendices/Appendix#Unidirectional `@OneToMany` performance|Appendix]]

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

#core