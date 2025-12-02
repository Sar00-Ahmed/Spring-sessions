`JdbcTemplate` is a *thin* abstraction over JDBC. 

It **does not** provide caching, lazy loading, or a full object-relational mapping like Hibernate. **It is not ORM**

**It is also 4x faster than JPA**
#### Raw JDBC
``` Java

public List<Employee> findEmployeesByDepartment(String department) {
    Connection connection = null;
    PreparedStatement statement = null;
    ResultSet resultSet = null;
    List<Employee> employees = new ArrayList<>();
        
    try {
       // 1. Open connection
       connection = DriverManager.getConnection(   
       "jdbc:postgresql://localhost:5432/mydb", "user", "password");
            
        // 2. Create PreparedStatement
        String sql = 
        "SELECT id, name, department, salary FROM employees WHERE department = ?";
        statement = connection.prepareStatement(sql);
            
        // 3. Manually set parameters
        statement.setString(1, department);
            
        // 4. Execute query
        resultSet = statement.executeQuery();
            
        // 5. Iterate through ResultSet
        while (resultSet.next()) {
            Employee employee = new Employee();
            employee.setId(resultSet.getLong("id"));
            employee.setName(resultSet.getString("name"));
            employee.setDepartment(resultSet.getString("department"));
            employee.setSalary(resultSet.getBigDecimal("salary"));
            employees.add(employee);
        }    
    } catch (SQLException e) {
        // 6. Handle SQLExceptions (checked exception)
        System.err.println("Database error: " + e.getMessage());
        throw new RuntimeException("Failed to fetch employees", e);
    } finally {
        // 7. Close all resources in finally block
        try {
            if (resultSet != null) resultSet.close();
        } catch (SQLException e) {
            System.err.println("Error closing result set: " 
            + e.getMessage());
        }
        
	    try {
            if (statement != null) statement.close();
        } catch (SQLException e) {
            System.err.println("Error closing statement: " 
            + e.getMessage());
        }
        try {
            if (connection != null) connection.close();
        } catch (SQLException e) {
            System.err.println("Error closing connection: " 
            + e.getMessage());
        }
    }
        
    return employees;
}
```

**`JdbcTemplate` solves this by:**
- Managing the lifecycle of connections and statements.
- It converts JDBC's checked `SQLException` into Spring's unchecked, more informative `DataAccessException` hierarchy. This is a huge benefit.
# Sample

``` Java
@Configuration
public class DatabaseConfig {

    @Bean
    public DataSource dataSource() {
        HikariDataSource dataSource = new HikariDataSource();
        dataSource.setJdbcUrl("jdbc:postgresql://localhost:5432/mydb");
        dataSource.setUsername("user");
        dataSource.setPassword("password");
        return dataSource;
    }

    @Bean
    public JdbcTemplate jdbcTemplate(DataSource dataSource) {
        return new JdbcTemplate(dataSource);
    }
}
```

> [!tip]
alternatively and preferably you can set the connection properties in the properties file
##### Update Operations (`INSERT`, `UPDATE`, `DELETE`)
```java
@Autowired
private JdbcTemplate jdbcTemplate;

public void createEmployee(String name, String department) {
    String sql = "INSERT INTO employees (name, department) VALUES (?, ?)";
    // Returns number of rows affected
    // jdbc.update(sql, params->?)
    int rows = jdbcTemplate.update(sql, name, department);
    System.out.println(rows + " row(s) inserted.");
}

public void updateEmployeeDepartment(Long id, String newDepartment) {
    String sql = "UPDATE employees SET department = ? WHERE id = ?";
    jdbcTemplate.update(sql, newDepartment, id);
}
```
##### Querying for Single Values
Use `queryForObject()` when you expect a single result (e.g., count, a specific column).

```java
public int getEmployeeCount() {
    String sql = "SELECT COUNT(*) FROM employees";
    // queryForObject(sql, Class<T> requiredType)
    return jdbcTemplate.queryForObject(sql, Integer.class);
}

public String getEmployeeName(Long id) {
    String sql = "SELECT name FROM employees WHERE id = ?";
    return jdbcTemplate.queryForObject(sql, String.class, id);
}
```

> [!attention]
> **`queryForObject` vs `query`:**
> Use `queryForObject` for exactly one row. `queryForObject` will throw an exception if the result size is not 1.
> If the query might return zero or multiple rows, use `query` and handle the `List`. 
# Named Parameters

The standard `JdbcTemplate` only uses positional parameters (`?`). For complex queries with many parameters, this can be hard to read and maintain.

Spring provides `NamedParameterJdbcTemplate`, which allows you to use named placeholders (e.g., `:name`).

```java
@Autowired
private NamedParameterJdbcTemplate namedParameterJdbcTemplate;

public void updateEmployee(Employee emp) {
    String sql = "UPDATE employees SET name = :name, department = :dept WHERE id = :id";
    
    // Use a Map to supply parameters
    Map<String, Object> params = new HashMap<>();
    params.put("name", emp.getName());
    params.put("dept", emp.getDepartment());
    params.put("id", emp.getId());
    
    namedParameterJdbcTemplate.update(sql, params);
}
```
# Mappers
. You need a **RowMapper** to convert each row in the `ResultSet` into a domain object.

```java
public class Employee {
    private Long id;
    private String name;
    private String department;
    // ... constructors, getters, setters
}
```

```java
// Option 1: Implement RowMapper in your DAO method
public Employee findEmployeeById(Long id) {
    String sql = "SELECT * FROM employees WHERE id = ?";
    return jdbcTemplate.queryForObject(sql, new RowMapper<Employee>() {
        @Override
        public Employee mapRow(ResultSet rs, int rowNum) throws SQLException {
            Employee employee = new Employee();
            employee.setId(rs.getLong("id"));
            employee.setName(rs.getString("name"));
            employee.setDepartment(rs.getString("department"));
            return employee;
        }
    }, id);
}

// Option 2 (Recommended): Use a Lambda Expression (Java 8+)
public Employee findEmployeeById(Long id) {
    String sql = "SELECT * FROM employees WHERE id = ?";
    return jdbcTemplate.queryForObject(sql, (rs, rowNum) -> {
        Employee employee = new Employee();
        employee.setId(rs.getLong("id"));
        employee.setName(rs.getString("name"));
        employee.setDepartment(rs.getString("department"));
        return employee;
    }, id);
}
```

**For `List<>`**
```java
public List<Employee> findAllEmployees() {
    String sql = "SELECT * FROM employees";
    return jdbcTemplate.query(sql, (rs, rowNum) -> {
        Employee employee = new Employee();
        employee.setId(rs.getLong("id"));
        employee.setName(rs.getString("name"));
        employee.setDepartment(rs.getString("department"));
        return employee;
    });
}
```

> [!tip]
> If your database column names match your Java class property names (e.g., `department` column -> `department` field), you can use built-in mapper `BeanPropertyRowMapper` 
>``` java
	> public List<Employee> findAllEmployees() {
    >String sql = "SELECT * FROM employees";
    >// BeanPropertyRowMapper.newInstance(Employee.class) also works
    >return jdbcTemplate.query(sql, new BeanPropertyRowMapper<>(Employee.class));
	>}
>```

# Batching
##### Without Batching (Slow)
``` java
// 1000 separate round-trips to database
for (Employee emp : employees) {
    jdbcTemplate.update("INSERT ...", emp.getName(), emp.getDepartment());
}
```
##### With Batching (Fast)
``` java

// Single round-trip with 1000 statements
jdbcTemplate.batchUpdate("INSERT ...", employees, 1000, (ps, emp) -> {
    ps.setString(1, emp.getName());
    ps.setString(2, emp.getDepartment());
});
```

# When to use
- For heavy select queries where we want to better performance
- Creating tables at runtime that are not mapped to entities
- Dealing with external databases thus needing to create a datasource for this external db

# What will you lose
|Feature|JPA|JDBC|
|---|---|---|
|**Relationship Mapping**|Automatic|Manual|
|**Caching**|Built-in|None|
|**Lazy Loading**|Automatic|Manual|
|**Schema Management**|Automatic|Manual|
|**Type Safety**|Good|Poor|
|**Database Portability**|Excellent|Poor|
|**Optimistic Locking**|Automatic|Manual|
|**Development Speed**|Fast|Slow|
|**Boilerplate**|Minimal|Extensive|
|**Advanced Features**|Rich set|None|

> [!summary]
> **Choose JDBC when:** You need maximum performance, control over SQL.

#core 