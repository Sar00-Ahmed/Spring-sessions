# Global Exception handler
This intercepts any thrown exception and handles it.

The `@RestControllerAdvice` annotation is a combination of:
- `@ControllerAdvice` – Used for **global exception handling**
- `@ResponseBody` – Ensures that responses are **returned as JSON** instead of views
``` Java
@RestControllerAdvice  
public class GlobalExceptionHandler {  
  
	@ExceptionHandler(IllegalArgumentException.class)  
	public ResponseEntity<String> 
	handleIllegalArgument(IllegalArgumentException ex) {  
		return new ResponseEntity<>("Invalid input: " 
			+ ex.getMessage(), HttpStatus.BAD_REQUEST);  
	}  
}
```

`@ExceptionHandler` specifies the type of exception that is caught. and can carry multiple exceptions
```java
  @ExceptionHandler(value = {ResourceNotFoundException.class, CertainException.class})
```

you will usually need to create an error message class that carries the error response.
```java
public class ErrorMessage {
  private int statusCode;
  private Date timestamp;
  private String message;
  private String description;
  private String tx; //transaction ID usually printed in logs, 
  //so the user knows where to search in logs

  public ErrorMessage(int statusCode, Date timestamp, String message, String description) {
    this.statusCode = statusCode;
    this.timestamp = timestamp;
    this.message = message;
    this.description = description;
  }
```

# Custom exception
Its is a best practice to create custom exceptions to handle them separately in the global exception handler and so that it becomes easier to identify the source issue. 

You can do this by having your class extend `RuntimeException`

```java
public class ResourceNotFoundException extends RuntimeException {

  private static final long serialVersionUID = 1L;

  public ResourceNotFoundException(String msg) {
    super(msg);
  }
}
```

#core 