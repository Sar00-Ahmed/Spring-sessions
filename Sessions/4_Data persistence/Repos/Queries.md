
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
>- Count how many employees in department “HR” have a salary greater than 10000.  
>- Check if any orders exist with status “SHIPPED” and deliveryDate before today.  
>- Delete all products that are **inactive** and have `stockQuantity` less than 5.  
>- Find all customers from Egypt whose phone number **starts with** “+20” and sort them by `lastName` ascending.

# `@Query` annotation
#core