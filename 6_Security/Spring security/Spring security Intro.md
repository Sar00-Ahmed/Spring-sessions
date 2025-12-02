as we've previously mentioned spring provides pre-made implementations for security and adding to that they are configurable.

Frameworks are developed by experienced individuals they give you pre-made libraries for you to use directly. with constant development and enhancements.
## How Spring Security Integrates with Spring Boot  

  You just need to include the `spring-boot-starter-security` dependency in your `pom.xml` (Maven) or `build.gradle` (Gradle).  
  ```xml
  <dependency>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-starter-security</artifactId>
  </dependency>
  ```
  - This pulls in all necessary Spring Security libraries (`spring-security-core`, `spring-security-web`, etc.).  

- **Auto-Configuration**:  
  - Enables security for all endpoints.  
  - Sets up a default `UserDetailsService` with an in-memory user (`user` + random password logged at startup).  
  - Configures basic authentication (HTTP Basic).  
  - `UsernamePasswordAuthenticationFilter` (for form login).  
  - `BasicAuthenticationFilter` (for HTTP Basic auth).  
  - `CsrfFilter` (CSRF protection).  
  - `SessionManagementFilter` (session handling).  

- **Customization**:  
  You can override defaults by:  
  - Defining a custom `SecurityFilterChain` bean.  
  - Extending `WebSecurityConfigurerAdapter` (deprecated in Spring Security 5.7+).  
  - Providing your own `UserDetailsService`.  

#### What’s NOT Premade?  
- **Database-backed user storage** (you configure this).  
- **JWT/OAuth2** (requires additional setup).  
- **Custom login pages** (you design them).  
s sensible defaults (but you can override everything).  


> [!warning]
> Current code syntax might be deprecated 
### Best Practices
1. Always use password encryption
2. Prefer role-based hierarchy
3. Secure sensitive endpoints with multiple factors
4. Regularly audit security configurations

For JWT implementations, consider:
- Token expiration policies
- Refresh token strategy
- Secure storage (HttpOnly cookies)
- storing extra claims to prevent stealing tokens

Spring supports multiple authentication schemes:

| Method     | Best For        | Configuration Example                |
| ---------- | --------------- | ------------------------------------ |
| Basic Auth | APIs            | `.httpBasic()`                       |
| Form Login | Traditional web | `.formLogin()`                       |
| JWT        | SPAs/APIs       | Custom filter + `.addFilterBefore()` |
| OAuth2     | Social login    | `.oauth2Login()`                     |

**Remember-Me**:
- Enable with `.rememberMe()`
- Configurable expiration (default: 2 weeks)

**Logout**:
- Configure with `.logout()`
- Can be stateful (session) or stateless (JWT)
#### Password Configuration
``` Java

@Configuration
public class PasswordConfig {

    @Bean
    public PasswordEncoder passwordEncoder() {
        return new BCryptPasswordEncoder(10);
    }
}
```

Now you can inject the bean `PasswordEncoder` wherever you want to encode the password. this will probably be in you user service in the registration service right before you save the changes to your database



# Resources & extra topics
1. https://www.youtube.com/watch?v=her_7pa0vrg&t=2557s
2. https://www.baeldung.com/
3. https://developer.mozilla.org/en-US/docs/Learn_web_development/Extensions/Server-side/First_steps/Website_security
4. Multi-Factor Authentication (MFA)
5. LDAP Authentication
6. Actuator Security (Securing `/actuator` endpoints)
7. CORS & Security Headers (Configuring CSP, HSTS) 

> [!hint]
> You may need to know about CORS to be able to integrate with angular. Otherwise spring might block requests coming from it.

Credits for the code and documentation to Amigos code. He has a lot of amazing tutorials on Spring so check out his channel!

#index