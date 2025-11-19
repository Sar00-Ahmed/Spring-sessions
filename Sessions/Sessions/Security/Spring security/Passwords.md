## Password Security
Storing passwords securely is **critical** to prevent breaches (e.g., plaintext leaks, rainbow table attacks). Here’s how to do it right in Spring Security:

> [!Warning]
> Avoid outdated algorithms like **MD5** or **SHA-1** (crackable in seconds).

### BCrypt	
General-purpose (most secure) `BCryptPasswordEncoder`.

``` java
@Bean
public PasswordEncoder passwordEncoder() {
    return new BCryptPasswordEncoder(12); // Strength=12 (default=10)
}
```
    
- Adjust **cost factor** (strength) based on hardware (higher = slower hashing).


> [!Note] Self study 
> What are salts? 
> Why are they used?
> Does Bcrypt use them? or do you need to implement them manually?

### Enforce Password Policies

- **Minimum length** (e.g., 12+ characters).
- **Block common passwords** (e.g., "password123").
- **Require mixed case, numbers, symbols**.

``` java

public class CustomPasswordValidator implements PasswordPolicy {
    @Override
    public void validate(String password) {
        if (password.length() < 12) {
            throw new IllegalArgumentException("Password too short!");
        }
        // Add more checks (e.g., regex for complexity)
    }
}
```

###  Secure Transmission (HTTPS + TLS)

- **Always use HTTPS** to prevent MITM attacks.
    
- **Disable HTTP** in production:
    
``` yaml
    # application.properties
    server.ssl.enabled=true
```


> [!NOTE] Self study
> What is MITM attack?
### Rate-Limit Login Attempts

Prevent brute-force attacks:

``` java

http.formLogin()
    .and()
    .sessionManagement()
        .maximumSessions(1)
        .maxSessionsPreventsLogin(true)
    .and()
    .and()
    .authenticationProvider(authenticationProvider())
    .addFilter(new RateLimitFilter()); // Custom filter
```

> [!Warning]
> ❗Never ever log passwords 
> ❗**Even with BCrypt, always use HTTPS**. Without it, passwords can be intercepted during login.


#core