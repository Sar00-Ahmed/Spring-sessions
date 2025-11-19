### Core Components
Spring Security handles user identity through:
- `UserDetails` interface (defines user properties)
- `UserDetailsService` (loads user-specific data)

**Key User Attributes**:
- Username
- Password (always encrypted)
- Roles (e.g., ADMIN, USER)
- Authorities (granular permissions)
- Account status (enabled/disabled)
# User Entity

``` Java
public class ApplicationUser implements UserDetails { //must implement user details to be recognised by spring security

//security info that need to be included
    private final String username;
    private final String password;
    private final Set<? extends GrantedAuthority> grantedAuthorities;
    private final boolean isAccountNonExpired;
    private final boolean isAccountNonLocked;
    private final boolean isCredentialsNonExpired;
    private final boolean isEnabled;
    
//other info you want to save for you user

    @Override
    public Collection<? extends GrantedAuthority> getAuthorities() {
        return grantedAuthorities;
    }

    @Override
    public String getPassword() {
        return password;
    }

    @Override 
    public String getUsername() {
        return username; //can return email if you want login to be with email
    }

    @Override
    public boolean isAccountNonExpired() {
        return isAccountNonExpired;
    }

    @Override
    public boolean isAccountNonLocked() {
        return isAccountNonLocked;
    }

    @Override
    public boolean isCredentialsNonExpired() {
        return isCredentialsNonExpired;
    }

    @Override
    public boolean isEnabled() {
        return isEnabled;
    }
}
```

# User Service

you **do need to implement and set a `UserDetailsService`** for the `DaoAuthenticationProvider` to work. The `DaoAuthenticationProvider` uses it to:
  - Load user details (username, password, authorities) from your database.
  - Validate credentials during login.

```java
@Service
public class CustomUserDetailsService implements UserDetailsService {
    @Autowired
    private final UserRepository userRepository; // Your JPA/DB layer

    @Override
    public UserDetails loadUserByUsername(String username) 
        throws UsernameNotFoundException {
        
        User user = userRepository.findByUsername(username)
            .orElseThrow(() -> new UsernameNotFoundException("User not found"));

        return new org.springframework.security.core.userdetails.User(
            user.getUsername(),
            user.getPassword(),
            user.getAuthorities() // Convert your DB roles to `GrantedAuthority`
        );
    }
}
```

#### **Step 2: Configure It in `DaoAuthenticationProvider`**
Exactly as you did in your snippet:
```java
@Bean
public DaoAuthenticationProvider daoAuthenticationProvider() {
    DaoAuthenticationProvider provider = new DaoAuthenticationProvider();
    provider.setPasswordEncoder(passwordEncoder);
    provider.setUserDetailsService(customUserDetailsService); // Inject your service
    return provider;
}
```

> [!help]
> Notice here that there that in the configurations there's a password encoder that needs to be provided, more on that later


#core 