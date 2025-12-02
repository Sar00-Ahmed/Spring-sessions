> [!definition]
> It is the **entry point for an application that handles incoming requests.** Think of it as a **receptionist**.

A controller's main job is to:
##### Listen
It listens for requests to specific URLs (like `GET /users` or `POST /products`).
##### Coordinate
**It doesn't contain the core business logic.** Instead, it delegates the request to the appropriate service layer to perform the actual work (like saving a user to a database or calculating something).  
##### Respond
After the work is done, it packages the result and sends a response back to the client. This response can be a web page, JSON data, or an error code.
## `@Controller` Annotation
 The main annotation for defining a controller. It is used for traditional MVC applications that return views.

``` Java
import org.springframework.stereotype.Controller;
import org.springframework.ui.Model;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.servlet.ModelAndView;

@Controller
public class ViewController {

    @GetMapping("/welcome")
    public ModelAndView showWelcomePage() {
        ModelAndView mav = new ModelAndView("welcome-page");
        mav.addObject("message", "Welcome to our website!");
        return mav; // This returns the name of the HTML template file.
    }
}
```

The method returns a `String` or `ModelAndView`, which is resolved by a **`ViewResolver`** to a specific HTML page (e.g., `welcome-page.html`).
## `@RestController` Annotation
It is a convenience annotation that combines `@Controller` and `@ResponseBody`. This is the one you'll use for building modern RESTful APIs that return data directly (e.g., JSON or XML) rather than a view.

``` Java
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RestController;

@RestController
public class ApiController {

    @GetMapping("/api/greeting")
    public Greeting getGreeting() {
        return new Greeting("Hello, RESTful World!");
    }
}
// This is the simple POJO that will be converted to JSON.
class Greeting {
    private String content;

    public Greeting(String content) {
        this.content = content;
    }

    public String getContent() {
        return content;
    }
}
```

## Collecting HTTP data

When an HTTP request arrives at your Spring Boot application, the framework's job is to take these raw components and map them to your controller's method parameters. This is where those annotations we discussed come in handy:

- The **HTTP Method** maps to annotations like `@GetMapping`, `@PostMapping`, etc.
    
- The **Request URI** maps to the value in `@RequestMapping` or its more specific variants.
    
- **Path variables** (e.g., `/users/{id}`) are extracted using `@PathVariable`.
    
- **Query parameters** are captured using `@RequestParam`.
    
- The **Request Headers** can be read using `@RequestHeader`.
    
- The **Request Body** is automatically deserialized into a Java object using `@RequestBody`.

``` Java
import org.springframework.http.ResponseEntity;
import org.springframework.web.bind.annotation.*;
import java.util.Map;

@RestController
@RequestMapping("/api/data")
public class DataCollectionController {

    //can also be written as: 
    //@RequestMapping(value = "/users/{userId}", method = RequestMethod.GET)
    @GetMapping("/users/{userId}")
    public ResponseEntity<String> getUserData(
            @PathVariable("userId") Long userId,
            @RequestParam(value = "name", required = false) String name,
            @RequestHeader Map<String, String> headers) {

        //code....

        // A simple response confirming data was collected
        String responseMessage = //....
        return ResponseEntity.ok(responseMessage);
    }

    // Demonstrates handling the request body
    @PostMapping("/create-user")
    public ResponseEntity<User> createUser(@RequestBody UserDTO user) {
        
        //code....

        return ResponseEntity.ok(user);
    }
}
```

#core