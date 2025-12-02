## Authorisation Configuration
The security configuration extends `WebSecurityConfigurerAdapter`:

```java
@Configuration
@EnableWebSecurity
public class ApplicationSecurityConfig extends WebSecurityConfigurerAdapter {
    
    @Override
    protected void configure(HttpSecurity http) throws Exception {
        http
            .csrf().disable() // Enable for stateful apps
            .authorizeRequests()
                .antMatchers("/public/**").permitAll()
                .antMatchers("/admin/**").hasRole("ADMIN")
                .anyRequest().authenticated()
            .and()
            .formLogin(); // Or other auth method
    }
}
```

Key decisions:
1. **CSRF Protection**: Required for session-based auth
2. **Public Endpoints**: Explicitly permit access
3. **Role/Authority Checks**: Apply at URL or method level

### Method-Level Control
Use annotations for granular authorization:
```java
@PreAuthorize("hasRole('ADMIN') or hasAuthority('WRITE')")
public void sensitiveOperation() {
    // Method logic
}
```


# Roles and Authorities
#### Enum roles and authorities
``` Java
public enum ApplicationUserPermission {
    STUDENT_READ("student:read"),
    STUDENT_WRITE("student:write"),
    COURSE_READ("course:read"),
    COURSE_WRITE("course:write");

    private final String permission;

    ApplicationUserPermission(String permission) {
        this.permission = permission;
    }

    public String getPermission() {
        return permission;
    }

}
```

``` Java

public enum ApplicationUserRole {
    STUDENT(Sets.newHashSet()),
    ADMIN(Sets.newHashSet(COURSE_READ, COURSE_WRITE, STUDENT_READ, STUDENT_WRITE)),
    ADMINTRAINEE(Sets.newHashSet(COURSE_READ, STUDENT_READ));

    private final Set<ApplicationUserPermission> permissions;

    ApplicationUserRole(Set<ApplicationUserPermission> permissions) {
        this.permissions = permissions;
    }

    public Set<ApplicationUserPermission> getPermissions() {
        return permissions;
    }

	//to extract permissions of a role
    public Set<SimpleGrantedAuthority> getGrantedAuthorities() {
        Set<SimpleGrantedAuthority> permissions = getPermissions().stream()
                .map(permission -> new SimpleGrantedAuthority(permission.getPermission()
                ))
                .collect(Collectors.toSet());
        permissions.add(new SimpleGrantedAuthority("ROLE_" + this.name()));
        return permissions;
    }
}
```
#### configurations
``` Java

@Configuration
@EnableWebSecurity
@EnableGlobalMethodSecurity(prePostEnabled = true) //enables preauthorise in apis
public class ApplicationSecurityConfig extends WebSecurityConfigurerAdapter {
    private final PasswordEncoder passwordEncoder;
    private final CustomUserService customUserService;

    @Autowired
    public ApplicationSecurityConfig(
    PasswordEncoder passwordEncoder,
    CustomUserService customUserService) {
        this.passwordEncoder = passwordEncoder;
        this.customUserService = customUserService;
    }

    @Override
    protected void configure(HttpSecurity http) throws Exception {
        http
                .authorizeRequests()
                .antMatchers("/", "index", "/css/*", "/js/*").permitAll()
                .antMatchers("/api/**").hasRole(STUDENT.name()) // any api
                // starting with /api need to have student role to access
                
	            .antMatchers(HttpMethod.DELETE, "/management/api/**")
		            .hasAuthority(COURSE_WRITE.getPermission())
		        //notice that we here specified http method and authorities
		        //instead of roles
		        
				.antMatchers(HttpMethod.POST, "/management/api/**")
					.hasAuthority(COURSE_WRITE.getPermission())
					
				.antMatchers(HttpMethod.PUT, "/management/api/**")
					.hasAuthority(COURSE_WRITE.getPermission())
					
				.antMatchers("/management/api/**")
					.hasAnyRole(ADMIN.name(), ADMINTRAINEE.name())
				 	
                .anyRequest()
                .authenticated()
                .and()
                .httpBasic();
    }

    @Override
    protected void configure(AuthenticationManagerBuilder auth) 
    throws Exception {
        auth.authenticationProvider(daoAuthenticationProvider());
    }

    @Bean
    public DaoAuthenticationProvider daoAuthenticationProvider() {
        DaoAuthenticationProvider provider = new DaoAuthenticationProvider();
        provider.setPasswordEncoder(passwordEncoder);
        provider.setUserDetailsService(customUserService);
        return provider;
    }

}
```


> [!Tip] 
> Matchers are checked in order, the first one to indeed match will be executed and the rest will be neglected

#### Apis

``` Java

package com.example.demo.student;

import org.springframework.security.access.prepost.PreAuthorize;
import org.springframework.web.bind.annotation.*;

import java.util.Arrays;
import java.util.List;

@RestController
@RequestMapping("management/api/v1/students")
public class StudentManagementController {

    private static final List<Student> STUDENTS = Arrays.asList(
            new Student(1, "James Bond"),
            new Student(2, "Maria Jones"),
            new Student(3, "Anna Smith")
    );

    @GetMapping
    @PreAuthorize("hasAnyRole('ROLE_ADMIN', 'ROLE_ADMINTRAINEE')") 
    public List<Student> getAllStudents() {
        System.out.println("getAllStudents");
        return STUDENTS;
    }

    @PostMapping
    @PreAuthorize("hasAuthority('student:write')")
    public void registerNewStudent(@RequestBody Student student) {
        System.out.println("registerNewStudent");
        System.out.println(student);
    }

    @DeleteMapping(path = "{studentId}")
    @PreAuthorize("hasAuthority('student:write')")
    public void deleteStudent(@PathVariable("studentId") Integer studentId) {
        System.out.println("deleteStudent");
        System.out.println(studentId);
    }

    @PutMapping(path = "{studentId}")
    @PreAuthorize("hasAuthority('student:write')")
    public void updateStudent(@PathVariable("studentId") Integer studentId, @RequestBody Student student) {
        System.out.println("updateStudent");
        System.out.println(String.format("%s %s", studentId, student));
    }
}

```


> [!tip] 
> You don't need to handle authorisation for apis in both controller and configuration

#### Should follow any of this format
`hasRole('ROLE_rolename1') `
`hasAnyRole('ROLE_rolename1', 'ROLE_rolename2') `
`hasAuthority('authority1')`
`hasAnyAuthority('authority1, authority2')`

# Extra: Ownership checks
Let's say you have a scenario where you want to check if a certain user owns this data thus having access to it. Like a user accessing his profile page he should only see his own data regardless of his authorities and role.

This can be done using `@PreAuthorize` with SpEL expression

``` Java
@GetMapping("/users/{userId}")
@PreAuthorize("#userId == authentication.principal.id") // SpEL expression
public User getUser(@PathVariable Long userId) {
    return userService.findById(userId);
}
```

**or you can manually check the `userId` from the security context holder in your service**

#core