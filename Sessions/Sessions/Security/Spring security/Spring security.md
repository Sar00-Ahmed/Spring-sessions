as we've previously mentioned spring provides pre-made implementations for security and adding to that they are configurable.

Frameworks are developed by experienced individuals they give you pre-made libraries for you to use directly. with constant development and enhancements.
## How Spring Security Integrates with Spring Boot  
**Spring Security** integrates seamlessly with **Spring Boot** via **Maven** (or **Gradle**). Spring Boot provides **auto-configuration** and **starter dependencies** to simplify setup.  

#### **Key Integration Points:**
- **Dependency Management**:  
  You just need to include the `spring-boot-starter-security` dependency in your `pom.xml` (Maven) or `build.gradle` (Gradle).  
  ```xml
  <dependency>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-starter-security</artifactId>
  </dependency>
  ```
  - This pulls in all necessary Spring Security libraries (`spring-security-core`, `spring-security-web`, etc.).  

- **Auto-Configuration**:  
  Spring Boot automatically:  
  - Enables security for all endpoints.  
  - Sets up a default `UserDetailsService` with an in-memory user (`user` + random password logged at startup).  
  - Configures basic authentication (HTTP Basic).  

- **Customization**:  
  You can override defaults by:  
  - Defining a custom `SecurityFilterChain` bean.  
  - Extending `WebSecurityConfigurerAdapter` (deprecated in Spring Security 5.7+).  
  - Providing your own `UserDetailsService`.  

---

### **2. Is Spring Security a "Premade Package"?**  
✅ **Yes!** It’s a **modular framework** bundled with Spring Boot, but you can customize every part.  

#### **What’s Included by Default?**  
- **Core Security Filters**:  
  - `UsernamePasswordAuthenticationFilter` (for form login).  
  - `BasicAuthenticationFilter` (for HTTP Basic auth).  
  - `CsrfFilter` (CSRF protection).  
  - `SessionManagementFilter` (session handling).  

- **Default Behaviors**:  
  - All endpoints are secured.  
  - Auto-generated login page (if using form login).  
  - Password is logged at startup (for dev convenience).  

#### **What’s NOT Premade?**  
- **Database-backed user storage** (you configure this).  
- **JWT/OAuth2** (requires additional setup).  
- **Custom login pages** (you design them).  

---
### **Key Takeaways**  
1. **Auto-configuration** provides sensible defaults (but you can override everything).  


> [!warning]
> Current code syntax might be deprecated 

# Table of contents
[[Passwords]]
[[User Details]]
[[Roles and Authorities]]
[[Simple Authentication methods]]
[[JWT auth]]
### Best Practices
1. Always use password encryption
2. Prefer role-based hierarchy
3. Secure sensitive endpoints with multiple factors
4. Regularly audit security configurations

For JWT implementations, consider:
- Token expiration policies
- Refresh token strategy
- Secure storage (HttpOnly cookies)


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