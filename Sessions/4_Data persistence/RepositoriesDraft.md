#todo 
- [x] Optional return type
- [x] custom queries 
- [x] native queries 
- [ ] jdbc template
- [x] pagination
- [ ] batches
- [ ] procedures
- [x] properties persistance settings in spring


# Query methods

# Optional return type

### Phase 4: Advanced Topics

9. **Native Queries** → When and why to use them
    
10. **JDBC Template** → For complex SQL or performance needs

#core 

``` Java
@GetMapping("/api/data")
public String getData(@RequestHeader Map <String, String> headers) {

    List<String> cookieValues = headers.get("Cookie");
    return "Processing cookies: " + cookieValues;
}
```

