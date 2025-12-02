
A relationship can be **bidirectional** (each entity has a field referencing the other).
 In the database, there’s **only one actual foreign key column** (or one join table, in the case of many-to-many).
    
-JPA therefore needs to know:  
     **Which side "owns" the relationship (manages the foreign key / join table updates)?**

> [!definition]
>The **owning side** is the side that JPA looks at when deciding how to update the database.  
>The **inverse side** (`mappedBy`) just mirrors the relationship and doesn’t control the database mapping.
# How It Works by Relationship Type

### `@ManyToOne` & `@OneToMany`

- **Owning side = `@ManyToOne`** side (because the foreign key is on the "many" table). it does not have a mapped by option.
- The `@OneToMany(mappedBy = "...")` is always the inverse side.
    

```java
@Entity
class Order {
    @Id
    private Long id;

    @ManyToOne          // Owning side -> foreign key column "customer_id"
    @JoinColumn(name="customer_id")
    private Customer customer;
}

@Entity
class Customer {
    @Id
    private Long id;

    @OneToMany(mappedBy = "customer") // Inverse side
    private List<Order> orders;
}
```

---

### One-To-One
- Foreign key can be in **either table** (you choose).
- The side with `@JoinColumn` is the **owning side**.
- The other side uses `mappedBy`.
- in case of unidirectional, the side that has the `@OneToOne` annotation is the owning side
### Many-To-Many
- Always implemented via a **join table**.
- One side defines the `@JoinTable` → **owning side**.
- Other side uses `mappedBy`.
# Why This Matters

1. **Updates happen only through the owning side**
    
    - If you set a relationship only on the inverse side, JPA will ignore it unless you also update the owning side.
        
    - Common bug:
        
        ```java
        customer.getOrders().add(order); // only inverse side
        em.persist(customer); // order not updated in DB!
        ```
        
        You also need:
        
        ```java
        order.setCustomer(customer); // owning side
        ```
        
2. **`mappedBy` prevents duplicate mappings**
    
    - If you don’t declare an inverse side, JPA may create duplicate join columns or tables.


> [!tip]
>The **owning side = the one with `@JoinColumn` or `@JoinTable`**.  
>The **inverse side = the one with `mappedBy`**.
>For unidirectional: the annotated side is the owner by default    
>For `@OneToMany`:
 >
>- **Unidirectional** → annotated side is owning, but it usually creates a join table (not FK) unless explicitly overridden.      
>- **Bidirectional** → `@ManyToOne` side is the true owner.

#conceptual 