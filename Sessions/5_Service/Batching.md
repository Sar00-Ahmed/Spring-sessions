# JDBC vs JPA
## JPA Batch Behaviour
``` java

@Transactional
public void jpaSaveAll(List<Employee> employees) {
    repository.saveAll(employees); // Looks simple but...
}
```
**What actually happens**
``` sql

-- With JPA saveAll() and IDENTITY generation:
INSERT INTO employees (name, department) VALUES ('John', 'IT');
INSERT INTO employees (name, department) VALUES ('Jane', 'HR');
INSERT INTO employees (name, department) VALUES ('Bob', 'Finance');
-- 3 separate round-trips to database!
-- Each insert waits for the generated ID
```

**Unless** → you configure batch behaviour
``` properties
spring.jpa.properties.hibernate.jdbc.batch_size=50
spring.jpa.properties.hibernate.order_inserts=true
spring.jpa.properties.hibernate.order_updates=true
```

> [!caution]
> - The `GenerationType.IDENTITY` strategy **disables batch inserts** because the database must generate the ID immediately upon insert, forcing a round-trip to retrieve the generated ID before moving to the next insert.
>- For effective batching in JPA, use `GenerationType.SEQUENCE` or `TABLE`.

# JPA overhead
## Dirty checking
``` Java
@Transactional
public void updateEmployeeSalary(Long employeeId, BigDecimal newSalary) {
    Employee employee = employeeRepository.findById(employeeId).orElseThrow();
    employee.setSalary(newSalary); // That's it! No explicit save call needed
    
    // JPA automatically detects the salary change and 
    // generates: UPDATE employees SET salary = ? WHERE id = ?
    // when the transaction commits
}
```

In a transactional context after the transaction ends - JPA performs dirty checking:
- Compares current state with original snapshot
- Detects changes
- Generates update statements

This is done for every entity thus is inefficient in case of batches  

``` Java
@Transactional
public void processLargeDataset(List<Long> employeeIds) {
    List<Employee> employees = 
    employeeRepository.findAllById(employeeIds);
    
    // JPA must check EVERY field of EVERY entity for changes
    // For 1000 employees with 10 fields each = 10,000 comparisons!
    
    employees.forEach(emp -> {
        // Even if we don't change anything, JPA still checks
        emp.getDepartment(); 
        // Just reading still incurs checking overhead
    });
}
```

> [!todo] Research
> What does this annotation `@DynamicUpdate` do? 

> [!summary]
> 
| Complexity                  | JPA/Hibernate                                                                      | JdbcTemplate                                                                                  |
| --------------------------- | ---------------------------------------------------------------------------------- | --------------------------------------------------------------------------------------------- |
| **Identity Generation**     | Breaks batching with `IDENTITY`                                                    | Uses manual IDs or sequences                                                                  |
| **Persistence Context**     | Tracks all entities **(memory overhead)**<br>saved at L1 cache level               | No entity tracking                                                                            |
| **Dirty Checking**          | Checks every field for changes<br>- you can detach entities to avoid this bahviour | Direct SQL - no checking                                                                      |
| **Relationship Management** | Complex with cascades, order issues                                                | Simple - you control SQL order                                                                |
| **Batching**                | needs explicit configuration                                                       | single roundtrip                                                                              |
| **Failure handling**        |   Stops on first failure<br>You lose information about which specific record caused the failure                                                                                  | Stops on first failure<br>You lose information about which specific record caused the failure |

