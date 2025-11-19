![[Pasted image 20250817115525.png]]

> [!warning]
> If a hacker steals the token they can impersonate the user, for as long as the token is valid

In stateless systems (JWT/OAuth2), **logout is client-side** since there's no server session.

You can visit [JWT wbsite](https://www.jwt.io/) to inspect your token
## Dependencies

will need to add extra libraries for jwt
```XML
  
<!-- JJWT Library (For JWT Creation/Validation) -->
<dependency>
    <groupId>io.jsonwebtoken</groupId>
    <artifactId>jjwt-api</artifactId>
    <version>0.11.5</version> <!-- Use latest version -->
</dependency>
<dependency>
    <groupId>io.jsonwebtoken</groupId>
    <artifactId>jjwt-impl</artifactId>
    <version>0.11.5</version>
    <scope>runtime</scope>
</dependency>
<dependency>
    <groupId>io.jsonwebtoken</groupId>
    <artifactId>jjwt-jackson</artifactId>
    <version>0.11.5</version>
    <scope>runtime</scope>
</dependency>
```

# Request filters

![[Pasted image 20250817122605.png]]

Filters should extend `OncePerRequestFilter`

we'll need two filters:
- Authentication (verifies user and creates token)
- Token verification (verifies jwt token)
## Authentication filter

``` Java

public class JwtUsernameAndPasswordAuthenticationFilter extends UsernamePasswordAuthenticationFilter {

    private final AuthenticationManager authenticationManager;
    private final JwtConfig jwtConfig;
    private final SecretKey secretKey;

    public JwtUsernameAndPasswordAuthenticationFilter(
    AuthenticationManager authenticationManager,
    JwtConfig jwtConfig,
    SecretKey secretKey) {
        this.authenticationManager = authenticationManager;
        this.jwtConfig = jwtConfig;
        this.secretKey = secretKey;
    }

    @Override
    public Authentication attemptAuthentication(HttpServletRequest request,
    HttpServletResponse response) throws AuthenticationException {

        try {
            UsernameAndPasswordAuthenticationRequest authenticationRequest = 
            new ObjectMapper()
                    .readValue(request.getInputStream(), 
                    UsernameAndPasswordAuthenticationRequest.class);

            Authentication authentication = 
            new UsernamePasswordAuthenticationToken(
                    authenticationRequest.getUsername(), //also called principal
                    authenticationRequest.getPassword() //also called credentials
            );

            Authentication authenticate = 
            authenticationManager.authenticate(authentication);
            return authenticate;

        } catch (IOException e) {
            throw new RuntimeException(e);
        }

    }
```

After authentication we need to override the method that will place the jwt token in request header.

``` Java
    @Override
    protected void successfulAuthentication(HttpServletRequest request,
     HttpServletResponse response, FilterChain chain,
     Authentication authResult) throws IOException, ServletException {
        String token = Jwts.builder()
                .setSubject(authResult.getName())
                
                .claim("authorities", authResult.getAuthorities())
                //you can set any claims you need here 
                //but mostly it's authorities
                
                .setIssuedAt(new Date())
                .setExpiration(java.sql.
		            Date.valueOf(LocalDate.now().
			            plusDays(jwtConfig.getTokenExpirationAfterDays())))
                
                .signWith(secretKey)
                .compact();

        response.addHeader(jwtConfig.getAuthorizationHeader(), 
        jwtConfig.getTokenPrefix() + token);
    }
}
```

In request header you'll find this:
``` JSON
{
	Authorization: "Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJzdWIiOiIxMjM0NTY3ODkwIiwibmFtZSI6IkpvaG4gRG9lIiwiYWRtaW4iOnRydWUsImlhdCI6MTUxNjIzOTAyMn0.KMUFsIDTnFmyG3nMiGM6H9FNFUROf3wh7SmqJp-QV30"
}
```

> [!tip]
> Notice that this filter does not contain the line
> `filterChain.doFilter(request, response);`
> That's because it's a special filter; an authentication filter. 
> 
> Where if the user successfully logs in the success handler is called `successfulAuthentication()` which we overrided to return a jwt token in the header
> 
> If the authentication fails 401 is returned to the user. 

![[Pasted image 20250817183953.png]]

## JWT token verification filter

```Java
public class JwtTokenVerifier extends OncePerRequestFilter {

    private final SecretKey secretKey;
    private final JwtConfig jwtConfig;

    public JwtTokenVerifier(SecretKey secretKey,
                            JwtConfig jwtConfig) {
        this.secretKey = secretKey;
        this.jwtConfig = jwtConfig;
    }

    @Override
    protected void doFilterInternal(HttpServletRequest request,
                                    HttpServletResponse response,
                                    FilterChain filterChain) 
                                    throws ServletException, IOException {

        String authorizationHeader =
         request.getHeader(jwtConfig.getAuthorizationHeader());

        if (Strings.isNullOrEmpty(authorizationHeader) ||
         !authorizationHeader.startsWith(jwtConfig.getTokenPrefix())) {
	            filterChain.doFilter(request, response);
	            return;
        }

        String token = authorizationHeader.
				        replace(jwtConfig.getTokenPrefix(), "");
				        //remover "Bearer " part
```

After extracting the token from the header we parse the token to extract user information and claims.

``` Java
        try {

            Jws<Claims> claimsJws = Jwts.parser()
                    .setSigningKey(secretKey)
                    .parseClaimsJws(token);

            Claims body = claimsJws.getBody();

            String username = body.getSubject();

//get authorities
            var authorities = (List<Map<String, String>>) 
					            body.get("authorities");

            Set<SimpleGrantedAuthority> simpleGrantedAuthorities = 
	            authorities.stream()
                    .map(m -> new SimpleGrantedAuthority(m.get("authority")))
                    .collect(Collectors.toSet());

//get credentials
            Authentication authentication = new 
	            UsernamePasswordAuthenticationToken(
                    username,
                    null, //don't pass passwords
                    simpleGrantedAuthorities
	            );

//update security context holder
            SecurityContextHolder.getContext().setAuthentication(authentication);

        } catch (JwtException e) {
            throw new IllegalStateException(String.format("Token %s cannot be trusted", token));
        }

		//any filter needs this to call next filter
        filterChain.doFilter(request, response);
    }
}
```

> [!note] Self study
> Why do we need to call this line at the end? 
> `filterChain.doFilter(request, response);`
> 
> Hint: read about chain of command design pattern
> 
> Streams library Java

> [!note]
> What are intercepters?
> How are they different from filters? Which execute first?
> Can they be used outside of MVC context?
# Config

``` Java
@Configuration
@EnableWebSecurity
@EnableGlobalMethodSecurity(prePostEnabled = true)
public class ApplicationSecurityConfig extends WebSecurityConfigurerAdapter {

    private final PasswordEncoder passwordEncoder;
    private final ApplicationUserService applicationUserService;
    
    private final SecretKey secretKey; // key for jwt encryption
    // service that extracts constants from properties file
    private final JwtConfig jwtConfig; 

    @Autowired
    public ApplicationSecurityConfig(...) {
	    //constructor and injection ommitted for readability
    }

    @Override
    protected void configure(HttpSecurity http) throws Exception {
        http
                .csrf().disable()
                .sessionManagement()
                    .sessionCreationPolicy(SessionCreationPolicy.STATELESS)
                //Disables HTTP sessions No `JSESSIONID` cookie will be created
                .and()

		        //filters
                .addFilter(new JwtUsernameAndPasswordAuthenticationFilter(
	                authenticationManager(), jwtConfig, secretKey))  
	                
	//Authentication filter Automatically added first in the auth filter chain
                .addFilterAfter(new JwtTokenVerifier(secretKey, jwtConfig),

			//This forces this filter to run immediately after login filter
	                JwtUsernameAndPasswordAuthenticationFilter.class)

                .authorizeRequests()
                .antMatchers("/", "index", "/css/*", "/js/*").permitAll()
                .antMatchers("/api/**").hasRole(STUDENT.name())
                .anyRequest()
                .authenticated();
    }

	//Other configs
}
```

![[Pasted image 20250817180932.png]]

## Setting secret keys

keys and configurations can be placed in spring properties file
```properties

application.jwt.secretKey=securesecuresecuresecuresecuresecuresecuresecuresecuresecuresecure
application.jwt.tokenPrefix=Bearer 
application.jwt.tokenExpirationAfterDays=10
```

``` Java

@ConfigurationProperties(prefix = "application.jwt")
public class JwtConfig {

    private String secretKey;
    private String tokenPrefix;
    private Integer tokenExpirationAfterDays;

    public JwtConfig() {
    }

	//place getters and setters for all variables or use lombok

    public String getAuthorizationHeader() {
        return HttpHeaders.AUTHORIZATION;
    }
}
```

``` Java
@Configuration
public class JwtSecretKey {

    private final JwtConfig jwtConfig;

    @Autowired
    public JwtSecretKey(JwtConfig jwtConfig) {
        this.jwtConfig = jwtConfig;
    }

    @Bean
    public SecretKey secretKey() {
        return Keys.hmacShaKeyFor(jwtConfig.getSecretKey().getBytes());
    }
}
```

## Security context

Since we updated the security context in filter and added the user information that was extracted from the token there. These are some ways to access this context to extract user info. 

### Within controller
``` Java
@GetMapping("/user-info")
public String getUserInfo(Principal principal) {
    String username = principal.getName();
    return "Current user: " + username;
}
```

``` Java
@GetMapping("/user-details")
public String getUserDetails(Authentication authentication) {
    String username = authentication.getName();
    Collection<? extends GrantedAuthority> authorities = authentication.getAuthorities();
    
    return "User: " + username + " | Roles: " + authorities;
}
```

``` Java
@GetMapping("/user")
public String getCurrentUser(@AuthenticationPrincipal UserDetails userDetails) {
    return "Current user: " + userDetails.getUsername();
}
```

### Within service
``` Java
@Service
public class MyService {
    public void doSomething() {
        Authentication authentication = SecurityContextHolder.getContext().getAuthentication();
        String username = authentication.getName();
        Collection<? extends GrantedAuthority> authorities = authentication.getAuthorities();
        
        // Your business logic
    }
}
```

### Accessing extra claims

``` Java
@GetMapping("/user-claims")
public Map<String, Object> getUserClaims(Authentication authentication) {
    Jwt jwt = (Jwt) authentication.getPrincipal();
    return jwt.getClaims();  // Returns all claims from the JWT
}
```

# Extra Security
## Token binding

You can make your JWT tokens more secure by adding **contextual claims** like device/browser information. This technique is called **token binding** and helps prevent token theft/reuse.

``` Java
public String generateToken(HttpServletRequest request, String username) {
    return Jwts.builder()
        .setSubject(username)
        // Standard claims
        .setIssuedAt(new Date())
        .setExpiration(new Date(System.currentTimeMillis() + EXPIRATION_MS))
        
        // Contextual claims
        .claim("ip", request.getRemoteAddr())
        .claim("userAgent", request.getHeader("User-Agent"))
        .claim("deviceId", generateDeviceFingerprint(request)) 
        // Custom fingerprint
        
        .signWith(SignatureAlgorithm.HS256, SECRET_KEY)
        .compact();
}

private String generateDeviceFingerprint(HttpServletRequest request) {
    // Combine IP + User-Agent + other immutable client data
    String data = request.getRemoteAddr() + request.getHeader("User-Agent");
    return DigestUtils.sha256Hex(data); // Creates a unique fingerprint
}
```

to validate this token in validation filter

``` Java
protected void doFilterInternal(HttpServletRequest request, ...) {
    // ... existing token parsing logic
    
    Claims claims = Jwts.parser()
        .setSigningKey(SECRET_KEY)
        .parseClaimsJws(token)
        .getBody();

    // Validate contextual claims
    if (!claims.get("ip").equals(request.getRemoteAddr()) || 
        !claims.get("userAgent").equals(request.getHeader("User-Agent"))) {
        throw new JwtException("Token context mismatch");
    }
}
```

| Approach               | Security Benefit            | User Impact              |
| ---------------------- | --------------------------- | ------------------------ |
| **IP Binding**         | Prevents geographic attacks | Fails on mobile networks |
| **User-Agent Binding** | Detects device changes      | Fails on browser updates |
| **Token Versioning**   | Instant revocation          | Requires DB lookup       |


## Short-Lived JWTs + Blacklist For logout

1. **Client**: Discards the JWT (remove from memory/localStorage).
2. **Server**: Maintains a short-lived token (e.g., 15-30 mins) + optional blacklist (e.g., Redis).

``` Java
// Logout endpoint (optional blacklist)
@PostMapping("/logout")
public void logout(@RequestHeader("Authorization") String token) {
    String jwt = token.replace("Bearer ", "");
    tokenBlacklist.add(jwt); // Store in Redis (expire after token TTL)
}
```

``` Java
http
  .logout(logout -> logout
    .logoutUrl("/api/logout")
    .logoutSuccessHandler((req, res, auth) -> res.setStatus(200))
    .permitAll()
  )
  .addFilterBefore(new JwtAuthFilter(), UsernamePasswordAuthenticationFilter.class);
```


#core