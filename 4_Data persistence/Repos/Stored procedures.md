# JPA

``` Java
public interface ProductRepository extends JpaRepository<Product, Integer> {
    
    // Option 1: Referencing entity procedure name
    @Procedure(name = "Product.getDetails")
    String getProductName(@Param("productId") int productId);
    
    // Option 2: Direct procedure name
    @Procedure(procedureName = "GET_PRODUCT_DETAILS")
    String getProductDetails(@Param("p_id") int productId);
    
    // Option 3: Database function
    @Procedure(function = true)
    Integer calculateProductValue(@Param("productId") int productId);
}
```

# JDBC template
``` Java
@Repository
public class ProductDao {
    
    @Autowired
    private JdbcTemplate jdbcTemplate;
    
    // Simple call
    public void callSimpleProcedure() {
        jdbcTemplate.execute("CALL procedure_name()");
    }
    
    // With parameters
    public void callProcedureWithParams(int id, String name) {
        jdbcTemplate.update("CALL update_product(?, ?)", id, name);
    }
    
    // Using SimpleJdbcCall (Recommended)
    public Map<String, Object> callProcedureUsingSimpleJdbcCall(int productId) {
        SimpleJdbcCall jdbcCall = new SimpleJdbcCall(jdbcTemplate)
            .withProcedureName("get_product_details")
            .withoutProcedureColumnMetaDataAccess()
            .declareParameters(
                new SqlParameter("p_id", Types.INTEGER),
                new SqlOutParameter("p_name", Types.VARCHAR),
                new SqlOutParameter("p_price", Types.DECIMAL)
            );
        
        Map<String, Object> params = new HashMap<>();
        params.put("p_id", productId);
        
        return jdbcCall.execute(params);
    }
}
```