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



# Basic authentication

![[Pasted image 20250816154658.png]]

- need to send username and password with every request encoded as base64
- no logout
``` JSON
{
	"username": user,
	"password": aydwgayudwasd-awyasyhaiwhsu-bawshauhsuw,
	"_CSRF": sfsdsfsfddsds-sfdsfsdfsd-bawshaadwadw
}
```
#### configurations file

``` Java
@Configuration
@EnableWebSecurity
@EnableGlobalMethodSecurity(prePostEnabled = true) // allows checking pre authorize annotation
public class ApplicationSecurityConfig extends WebSecurityConfigurerAdapter {

    private final PasswordEncoder passwordEncoder;

    @Autowired
    public ApplicationSecurityConfig(PasswordEncoder passwordEncoder) {
        this.passwordEncoder = passwordEncoder;
    }

    @Override
    protected void configure(HttpSecurity http) throws Exception {
        http
                .csrf().disable() // TODO: renable 
                .authorizeRequests()
                .antMatchers("/", "index", "/css/*", "/js/*")
                .permitAll() //all the previous are permitted without auth
                .anyRequest() // the rest
                .authenticated() //requires to be authenticated
                .and()
                .httpBasic(); // basic auth
    }

}
```


> [!NOTE] Self study
> What is CSRF and how to prevent this attack? 
> Is CSRF needed for jwt tokens?
> 
> What is SQL injection and how to protect against it?
#### **Works With Angular?** ✅ Yes, but not ideal

- **How it Works**
    - Frontend encodes credentials in Base64 (`Authorization: Basic <credentials>` header).
    - Sent with every request (no session/cookies).
- **Pros** Simple to implement.
- **Cons**
    - No built-in logout (must clear browser cache).
    - Credentials exposed if not using HTTPS.
- **Best For**: Internal APIs, quick prototypes.

**Angular Implementation**
``` typescript
// Add to HTTP interceptor
const authReq = req.clone({
  headers: req.headers.set(
    'Authorization',
    'Basic ' + btoa(username + ':' + password)
});
```


# Form based authentication

![[Pasted image 20250816200901.png]]

``` JSON
{
	"JSESSIONID": aydwgayudwasd-awyasyhaiwhsu-bawshauhsuw
}
```

> [!tip] 
> Default validity of the sessionID is 30 minutes

Form based login allows logout, spring security saves the session ID in an **in-memory database** but it can be (and better be) configured to save in an external database otherwise **if the server restarts all the session ids will be lost**.

``` Java
@Configuration
@EnableWebSecurity
@EnableGlobalMethodSecurity(prePostEnabled = true) // allows checking pre authorize annotation
public class ApplicationSecurityConfig extends WebSecurityConfigurerAdapter {

    private final PasswordEncoder passwordEncoder;

    @Autowired
    public ApplicationSecurityConfig(PasswordEncoder passwordEncoder) {
        this.passwordEncoder = passwordEncoder;
    }

    @Override
    protected void configure(HttpSecurity http) throws Exception {
        http
                .csrf().disable() // TODO: renable
                .authorizeRequests()
                .antMatchers("/", "index", "/css/*", "/js/*")
                .permitAll() 
                .anyRequest() 
                .authenticated() 
                .and()
                .formLogin(); // Form base authentication auth 🌟
                .and()
                .remeberMe() //defaults to 2 weeks 🌟 
	                .tokenValidity((int)TimeUnit.DAYS.toSeconds(21)) 
	                //overrides remember me token validity to 21 days
	                .key("secret key")
    }

}
```

#### Form data
``` Java
{
	"username": user,
	"password": awuidhuan-abwhaadw-abwhdhau,
	"remember-me": on
}
```

this is the form data that needs to be sent to backend to be detected automatically if you want to change these keys you can use:
`.passwordParameter("customPasswordKey")`
`.usernameParameter("customUsernameKey")`
`.rememberMeParameter("customrememberMeKey")`

This will allow spring to recognise:
``` Java
{
	"customUsernameKey": user,
	"customPasswordKey": awuidhuan-abwhaadw-abwhdhau,
	"customrememberMeKey": on
}
```

#### **Works With Angular?** ⚠️ Possible, but not recommended

- **How it Works**
    - Spring generates a session cookie (`JSESSIONID`) after login.
    - **Requires server-side sessions (stateful).**
- **Pros**
    - Built-in logout (`POST /logout`).
    - Supports `Remember-Me`.
- **Cons**
    - Tight coupling with server-side templates (Thymeleaf).
    - CSRF tokens required (complex for SPAs).
- **Best For**: Traditional server-rendered apps (Spring MVC + JSP/Thymeleaf).

**Alternative for SPAs**: Use **JWT** or **OAuth2** instead.

# Remember me
To use send in form data remeber-me key 
Keeps a cookie that is also stored in the in-memory database. This cookie contains:
- username
- expiration time
- MD5 hash of the above 2 values for verification.

> [!Tip] 
> For secret keys it's usually best to set them in `.env` files to be encrypted as you absolutely should not have hard-coded passwords or keys in your code.
> 
> or you can at-least put them as a variable in your properties file

# Logout
``` Java
http
  .logout(logout -> logout
    .logoutUrl("/logout")                 // Custom logout URL
    .clearAuthentication(true)            // Clears SecurityContext
    .invalidateHttpSession(true)          // Destroys session
    .deleteCookies("JSESSIONID", "remember-me")  // Clears cookies
    .logoutSuccessUrl("/login")           // Redirect after logout
  );
```


---

> [!NOTE] Self study
> What are CSRF & XSS attacks?
> Where to store tokens securely (HttpOnly cookies vs. LocalStorage, session storage)?


#core