> [!definition]
> **`@Configuration` indicates that a class contains `@Bean` definition methods**, which the Spring container can process to generate Spring Beans for use in the application. This annotation is part of the Spring Core framework.

### Some use cases
##### Grouping Properties into Type-Safe Configuration 
This is achieved by using the **`@ConfigurationProperties`** annotation on a class. While this class often goes hand-in-hand with `@Configuration` classes that use those properties, its main purpose is to create a strongly-typed object from a set of related properties in your external files (`.properties` or `.yml`).    
##### Conditional Bean Creation for Different Profiles
This is a key and advanced usage of `@Configuration` classes. By placing a **`@Profile`** annotation on a configuration class, you tell Spring to only process the beans and logic within that class if a specific environment profile (e.g., `dev` or `prod`) is active. This is an essential practice for managing different configurations for different environments.

> [!todo]
> **Research & Implement**
> Search for how to use `@Profile` and try to implement it

##### Creating and Managing Beans
This is the core function of a class annotated with **`@Configuration`**. Inside these classes, you use **`@Bean`** methods to define and instantiate objects (beans) that will be managed by the Spring IoC container. This is the primary mechanism for setting up your application's components.


``` Java
    import org.springframework.context.annotation.Bean;
    import org.springframework.context.annotation.Configuration;
    import org.springframework.security.crypto.bcrypt.BCryptPasswordEncoder;
    import org.springframework.security.crypto.password.PasswordEncoder;

    @Configuration
    public class SecurityConfig {

        @Bean
        public PasswordEncoder passwordEncoder() {
            return new BCryptPasswordEncoder();
        }
    }
```
# Beans

> [!definition]
> In Spring, a bean is any object that is managed by the Spring IoC (Inversion of Control) container. Think of beans as the fundamental building blocks of your application. Instead of you creating these objects yourself with the new keyword, the Spring container takes over this responsibility.


**`@Bean` Methods:** Methods inside `@Configuration` classes annotated with `@Bean` tell Spring "Take the object returned by this method and add it to your container so I can inject it elsewhere." This is used for configuring third-party libraries (like a `RestTemplate`, `Jackson ObjectMapper`) or any custom bean not annotated with `@Component`.
## scopes
They define the **lifecycle** of a bean and **how many instances** of it are created and managed by the Spring IoC container. They determine whether a single shared instance is used for the entire application or if new instances are created for each request.

| Scope         | Description                                                           | Behavior                                                                                                                      | Use Case                                                                                                   |
| ------------- | --------------------------------------------------------------------- | ----------------------------------------------------------------------------------------------------------------------------- | ---------------------------------------------------------------------------------------------------------- |
| **Singleton** | The default scope.                                                    | The Spring container creates only **one instance** of the bean per application, and it is shared and reused for all requests. | **Stateless services**, repositories, and utility classes.                                                 |
| **Prototype** | A new instance of the bean is created **every time it is requested**. | The container does not manage the full lifecycle; once created, it's up to the client to manage it.                           | **Stateful beans**, or objects that are not thread-safe.                                                   |
| **Request**   | A new instance of the bean is created for each **HTTP request**.      | The bean is valid for the duration of a single web request.                                                                   | Storing data specific to a single web request, like user session data or temporary configurations.         |
| **Session**   | A new instance of the bean is created for each **HTTP session**.      | The bean is valid for the duration of a user's web session.                                                                   | Storing user-specific data that persists across multiple requests in a session (e.g., shopping cart data). |
``` Java
import org.springframework.context.annotation.Scope;
import org.springframework.stereotype.Service;
import org.springframework.web.context.WebApplicationContext;

@Service
@Scope(value = WebApplicationContext.SCOPE_PROTOTYPE)
public class RequestCounterService {
    private int counter = 0;

    public void incrementCounter() {
        this.counter++;
    }

    public int getCounter() {
        return this.counter;
    }
}

```

# References
> [!cite]
> ```embed
title: "Advanced Dependency Injection in Spring: @Qualifier and @Primary"
image: "https://miro.medium.com/v2/da:true/resize:fit:1024/0*pEH7c6lWMDEUD0hZ"
description: "Dependency Injection (DI) is at the heart of the Spring Framework, helping developers build loosely coupled, testable, and flexible…"
url: "https://medium.com/devdomain/advanced-dependency-injection-in-spring-qualifier-and-primary-08eef7d9b0cb"
favicon: ""
aspectRatio: "100"
>```

> [!cite]
> ```embed
title: "Using the @Configuration annotation :: Spring Framework"
image: "https://docs.spring.io/spring-framework/reference/_/img/algolia-light.svg"
description: ""
url: "https://docs.spring.io/spring-framework/reference/core/beans/java/configuration-annotation.html"
favicon: ""
aspectRatio: "14.333333333333334"
>```

#core 