How to get started:

1. Both backend and frontend need to be up and running on the same device
2. Most of the integration effort is on frontend where they act like postman:
	1. they form a request specifying; get, post, body, params, headers etc...
	2. send this request to backend
	3. take the data returned in the form of JSON mapped into a model object
	4. Show this data correctly in the component
3. Then you'll need to handle in front end exceptions thrown by backend and their messages.
4. Another extra step if you have JWT security configured you'll need to:
	1. Add token to local storage
	2. create an interceptor to add it to request headers
	3. In case of logout you'll need to clear local storage.
	4. In case no token was found user needs to be redirected to login page
	5. Optional but clean step: make sure to encrypt anything in local storage rather than saving the token as is
5. Backend effort will take the form of resolving bugs found in backend while integrating.

# Sample
This is a small sample application to the above but I recommend watching these videos before you do anyting:
1. https://youtu.be/6qCeg6md0vc?si=FbGcTsR_DqqM_uDg
2. https://youtu.be/6Ndpf0JRSYg?si=XWNH2ZKu4Ib_g_lp
3. https://youtu.be/z666T_d9Mh4?si=s9qLNvVZK4Ft2G8U
4. https://youtu.be/h-yV0o7Zyyw?si=t552PZcXtKLXTwtV
5. https://youtu.be/QQeWxMdn_Zg?si=fuMUQyJRmm1bnMxu

## Frontend HTTP Client Integration

### Service Layer (user.service.ts)
```typescript
import { Injectable } from '@angular/core';
import { HttpClient, HttpParams, HttpHeaders } from '@angular/common/http';
import { Observable, throwError } from 'rxjs';
import { catchError, map } from 'rxjs/operators';

export interface User {
  id: number;
  name: string;
  email: string;
}

export interface ApiResponse<T> {
  data: T;
  message: string;
  status: string;
}

@Injectable({
  providedIn: 'root'
})
export class UserService {
  private apiUrl = 'http://localhost:8080/api';

  constructor(private http: HttpClient) { }

  // GET request with query params
  getUsers(page: number = 0, size: number = 10): Observable<User[]> {
    const params = new HttpParams()
      .set('page', page.toString())
      .set('size', size.toString());

    return this.http.get<ApiResponse<User[]>>(`${this.apiUrl}/users`, { params })
      .pipe(
        map(response => response.data),
        catchError(this.handleError)
      );
  }

  // POST request with body
  createUser(user: User): Observable<User> {
    const headers = new HttpHeaders({ 'Content-Type': 'application/json' });
    
    return this.http.post<ApiResponse<User>>(`${this.apiUrl}/users`, user, { headers })
      .pipe(
        map(response => response.data),
        catchError(this.handleError)
      );
  }

  // PUT request
  updateUser(id: number, user: User): Observable<User> {
    return this.http.put<ApiResponse<User>>(`${this.apiUrl}/users/${id}`, user)
      .pipe(
        map(response => response.data),
        catchError(this.handleError)
      );
  }

  // DELETE request
  deleteUser(id: number): Observable<void> {
    return this.http.delete<ApiResponse<void>>(`${this.apiUrl}/users/${id}`)
      .pipe(
        catchError(this.handleError)
      );
  }

  private handleError(error: any): Observable<never> {
    console.error('API Error:', error);
    
    let errorMessage = 'An unexpected error occurred';
    
    if (error.error?.message) {
      errorMessage = error.error.message;
    } else if (error.message) {
      errorMessage = error.message;
    }
    
    return throwError(() => new Error(errorMessage));
  }
}
```

### Component Usage (user.component.ts)
```typescript
import { Component, OnInit } from '@angular/core';
import { UserService, User } from './user.service';

@Component({
  selector: 'app-user',
  templateUrl: './user.component.html',
  styleUrls: ['./user.component.css']
})
export class UserComponent implements OnInit {
  users: User[] = [];
  loading = false;
  error: string | null = null;

  constructor(private userService: UserService) {}

  ngOnInit() {
    this.loadUsers();
  }

  loadUsers() {
    this.loading = true;
    this.error = null;

    this.userService.getUsers().subscribe({
      next: (data) => {
        this.users = data;
        this.loading = false;
      },
      error: (err) => {
        this.error = err.message;
        this.loading = false;
      }
    });
  }

  addUser() {
    const newUser: User = {
      id: 0,
      name: 'New User',
      email: 'new@example.com'
    };

    this.userService.createUser(newUser).subscribe({
      next: (user) => {
        this.users.push(user);
      },
      error: (err) => {
        this.error = err.message;
      }
    });
  }
}
```

### Template (user.component.html)
```html
<div class="container">
  <h2>Users</h2>
  
  <!-- Loading state -->
  <div *ngIf="loading" class="alert alert-info">
    Loading users...
  </div>
  
  <!-- Error state -->
  <div *ngIf="error" class="alert alert-danger">
    Error: {{ error }}
    <button (click)="loadUsers()" class="btn btn-sm btn-outline-danger ml-2">
      Retry
    </button>
  </div>
  
  <!-- Success state -->
  <div *ngIf="!loading && !error">
    <button (click)="addUser()" class="btn btn-primary mb-3">
      Add User
    </button>
    
    <div *ngIf="users.length === 0" class="alert alert-warning">
      No users found.
    </div>
    
    <div *ngIf="users.length > 0">
      <div *ngFor="let user of users" class="card mb-2">
        <div class="card-body">
          <h5 class="card-title">{{ user.name }}</h5>
          <p class="card-text">{{ user.email }}</p>
        </div>
      </div>
    </div>
  </div>
</div>
```

## 3. Error Handling & Backend Exception Mapping

**Backend Exception (Spring Boot):**
```java
@RestControllerAdvice
public class GlobalExceptionHandler {
    
    @ExceptionHandler(MethodArgumentNotValidException.class)
    public ResponseEntity<ErrorResponse> handleValidationExceptions(MethodArgumentNotValidException ex) {
        List<String> errors = ex.getBindingResult()
                .getFieldErrors()
                .stream()
                .map(error -> error.getField() + ": " + error.getDefaultMessage())
                .collect(Collectors.toList());
        
        ErrorResponse errorResponse = new ErrorResponse(
            "Validation Failed", 
            errors, 
            HttpStatus.BAD_REQUEST
        );
        
        return new ResponseEntity<>(errorResponse, HttpStatus.BAD_REQUEST);
    }
    
    @ExceptionHandler(Exception.class)
    public ResponseEntity<ErrorResponse> handleGeneralException(Exception ex) {
        ErrorResponse errorResponse = new ErrorResponse(
            ex.getMessage(), 
            Collections.emptyList(), 
            HttpStatus.INTERNAL_SERVER_ERROR
        );
        
        return new ResponseEntity<>(errorResponse, HttpStatus.INTERNAL_SERVER_ERROR);
    }
}

public class ErrorResponse {
    private String message;
    private List<String> details;
    private HttpStatus status;
    private LocalDateTime timestamp;
    
    // constructor, getters, setters
}
```

## 4. JWT Security Integration

### Auth Service (auth.service.ts)
```typescript
import { Injectable } from '@angular/core';
import { HttpClient } from '@angular/common/http';
import { BehaviorSubject, Observable } from 'rxjs';
import { tap } from 'rxjs/operators';
import * as CryptoJS from 'crypto-js';

const SECRET_KEY = 'your-secret-key';

@Injectable({
  providedIn: 'root'
})
export class AuthService {
  private currentUserSubject: BehaviorSubject<any>;
  public currentUser: Observable<any>;

  constructor(private http: HttpClient) {
    const user = this.getDecryptedItem('currentUser');
    this.currentUserSubject = new BehaviorSubject<any>(user);
    this.currentUser = this.currentUserSubject.asObservable();
  }

  login(credentials: {username: string, password: string}): Observable<any> {
    return this.http.post<any>('/api/auth/login', credentials)
      .pipe(
        tap(response => {
          if (response.token) {
            this.setEncryptedItem('token', response.token);
            this.setEncryptedItem('currentUser', response.user);
            this.currentUserSubject.next(response.user);
          }
        })
      );
  }

  logout() {
    this.removeItem('token');
    this.removeItem('currentUser');
    this.currentUserSubject.next(null);
  }

  getToken(): string | null {
    return this.getDecryptedItem('token');
  }

  isLoggedIn(): boolean {
    return !!this.getToken();
  }

  // Encryption methods
  private setEncryptedItem(key: string, value: any): void {
    const encrypted = CryptoJS.AES.encrypt(JSON.stringify(value), SECRET_KEY).toString();
    localStorage.setItem(key, encrypted);
  }

  private getDecryptedItem(key: string): any {
    const item = localStorage.getItem(key);
    if (!item) return null;
    
    try {
      const bytes = CryptoJS.AES.decrypt(item, SECRET_KEY);
      return JSON.parse(bytes.toString(CryptoJS.enc.Utf8));
    } catch (e) {
      this.logout();
      return null;
    }
  }

  private removeItem(key: string): void {
    localStorage.removeItem(key);
  }
}
```

### JWT Interceptor (jwt.interceptor.ts)
```typescript
import { Injectable } from '@angular/core';
import { HttpRequest, HttpHandler, HttpEvent, HttpInterceptor } from '@angular/common/http';
import { Observable } from 'rxjs';
import { AuthService } from './auth.service';

@Injectable()
export class JwtInterceptor implements HttpInterceptor {
    
    constructor(private authService: AuthService) {}
    
    intercept(request: HttpRequest<any>, next: HttpHandler): Observable<HttpEvent<any>> {
        const token = this.authService.getToken();
        
        if (token) {
            request = request.clone({
                setHeaders: {
                    Authorization: `Bearer ${token}`
                }
            });
        }
        
        return next.handle(request);
    }
}
```

### Auth Guard (auth.guard.ts)
```typescript
import { Injectable } from '@angular/core';
import { CanActivate, Router } from '@angular/router';
import { AuthService } from './auth.service';

@Injectable({
  providedIn: 'root'
})
export class AuthGuard implements CanActivate {
    
    constructor(private authService: AuthService, private router: Router) {}
    
    canActivate(): boolean {
        if (this.authService.isLoggedIn()) {
            return true;
        }
        
        this.router.navigate(['/login']);
        return false;
    }
}
```

### App Module Setup
```typescript
@NgModule({
  // ... other imports
  providers: [
    { provide: HTTP_INTERCEPTORS, useClass: JwtInterceptor, multi: true }
  ]
})
export class AppModule { }
```

## 5. Backend Integration Tips

1. **Use consistent API response format**
2. **Implement proper error handling with meaningful messages**
3. **Test endpoints with Postman first**
4. **Use DTOs for request/response mapping**
5. **Implement proper validation annotations**

# CORS Issue 
## Approach 1 (@CrossOrigin)

in spring add the following
``` Java
@Configuration
@EnableWebSecurity
public class SecurityConfig {

    @Bean
    public SecurityFilterChain securityFilterChain(HttpSecurity http) throws Exception {
        http
            .cors(cors -> cors.
	            configurationSource(corsConfigurationSource())) // Enable CORS
            .csrf(csrf -> csrf.disable()) // Disable CSRF for API endpoints
            .authorizeHttpRequests(authz -> authz
                .requestMatchers("/api/auth/**").permitAll()
                .requestMatchers("/api/public/**").permitAll()
                .requestMatchers("/api/**").authenticated()
                .anyRequest().permitAll()
            )
            .formLogin(form -> form.disable()) 
            // Disable form login for API
            .httpBasic(httpBasic -> httpBasic.disable()); 
            // Disable basic auth if using JWT

        return http.build();
    }

    @Bean
    public CorsConfigurationSource corsConfigurationSource() {
        CorsConfiguration configuration = new CorsConfiguration();
        
        // Allow Angular dev server
        configuration.setAllowedOrigins(Arrays.asList(
            "http://localhost:4200", 
            "http://127.0.0.1:4200",
            "https://your-production-domain.com" // Add production domain
        ));
        
        // Allow all HTTP methods
        configuration.setAllowedMethods(Arrays.asList(
            "GET", "POST", "PUT", "DELETE", "OPTIONS", "PATCH"
        ));
        
        // Allow all headers
        configuration.setAllowedHeaders(Arrays.asList(
            "Authorization", "Content-Type", "Accept", "X-Requested-With", 
            "Cache-Control", "Origin", "Access-Control-Request-Method", 
            "Access-Control-Request-Headers"
        ));
        
        // Allow credentials (cookies, authorization headers)
        configuration.setAllowCredentials(true);
        
        // Expose custom headers to frontend
        configuration.setExposedHeaders(Arrays.asList(
            "Authorization", "Content-Type", "Content-Disposition"
        ));
        
        // Set max age for preflight requests (in seconds)
        configuration.setMaxAge(3600L);
        
        UrlBasedCorsConfigurationSource source = 
	        new UrlBasedCorsConfigurationSource();
        source.registerCorsConfiguration("/api/**", configuration);
        source.registerCorsConfiguration("/auth/**", configuration);
        
        return source;
    }
}
```

### Common Issues

**Issue: Preflight (OPTIONS) requests blocked**
```java

// Make sure to allow OPTIONS method and disable CSRF
.configuration.setAllowedMethods(Arrays.asList("GET", "POST", "PUT", "DELETE", "OPTIONS"));
.csrf(csrf -> csrf.disable());
```

**Issue: Credentials not allowed**
```java

// Enable credentials
configuration.setAllowCredentials(true);

// And in Angular, make sure to include credentials:
HttpClient requests should include { withCredentials: true } if needed
```

**Issue: Headers not exposed**
```java

// Expose custom headers
configuration.setExposedHeaders(Arrays.asList("Authorization", "Custom-Header"));
```

## Approach 2 (Angular proxy)
This method is for development environment only not production but it is the recommended approach as setting CORS configs in spring can be a headache.
### Step 2: Create `proxy.conf.json`

``` json

{
  "/api": {
    "target": "http://localhost:8080",
    "secure": false,
    "changeOrigin": true,
    "logLevel": "debug",
    "pathRewrite": {
      "^/api": ""
    }
  }
}

```
### Step 3: Update `angular.json`

```json

"serve": {
  "builder": "@angular-devkit/build-angular:dev-server",
  "configurations": {
    "production": {
      "browserTarget": "frontend:build:production"
    },
    "development": {
      "browserTarget": "frontend:build:development",
      "proxyConfig": "proxy.conf.json"
    }
  },
  "defaultConfiguration": "development"
}
```

### Step 4: Update your Angular Service

```typescript

import { Injectable } from '@angular/core';
import { HttpClient } from '@angular/common/http';
import { Observable } from 'rxjs';

@Injectable({
  providedIn: 'root'
})
export class UserService {
  // Now using relative path - proxy will handle the redirect
  private apiUrl = '/api';

  constructor(private http: HttpClient) { }

  getUsers(): Observable<any[]> {
    return this.http.get<any[]>(`${this.apiUrl}/users`);
  }

  createUser(user: any): Observable<any> {
    return this.http.post<any>(`${this.apiUrl}/users`, user);
  }
}
```

> [!important]
> It is crucial that the api you use is relative to the API target in proxy config:`http://localhost:8080`, as this base will be added automatically to any request.

# Common Issues
