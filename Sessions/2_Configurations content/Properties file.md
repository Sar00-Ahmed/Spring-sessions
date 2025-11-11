The files `application.properties` and `application.yml` are the primary way to manage **externalised configuration** in a Spring Boot application. This means you can change your application's settings without having to recompile the code.

Think of them as a central place for your application's adjustable settings, such as the server port, database connection URLs, API keys, or logging levels. They are not meant for hard-coding values directly into your source code.

There are two main ways to use these files to inject configuration values into your Spring beans:
# `@Value` Annotation

The `@Value` annotation is a simple way to inject a single property from your configuration file directly into a field within a Spring bean. It's best used for injecting individual, simple values.

Code Sample:

Let's say your application.properties file contains:

``` properties
app.service.name=MyWebApp
app.service.port=8080
```

You can inject these values like this:

``` Java
import org.springframework.beans.factory.annotation.Value;
import org.springframework.stereotype.Component;

@Component
public class AppConfig {
    @Value("app.service.name")
    private String serviceName;

    @Value("${app.service.port}")
    private int port;

    // Getters and other methods
}
```

# `@ConfigurationProperties` Annotation
This is the recommended approach for more complex configurations. It allows you to bind **multiple related properties to a single, type-safe Java object**. This is more robust and prevents common errors like typos.

Code Sample:

If your `application.yml` file has a nested structure like this:

``` yaml
app:
  server:
    port: 8080
    host: localhost
```

You can create a class to bind these properties to:

``` Java
import org.springframework.boot.context.properties.ConfigurationProperties;
import org.springframework.stereotype.Component;

@Component
@ConfigurationProperties(prefix = "app.server")
public class AppServerProperties {
    private int port;
    private String host;

    // Getters and setters
}
```

Spring will automatically create an instance of `AppServerProperties` and populate its fields with the values from your configuration file. This makes your configuration strongly typed and easy to use.

- - - 
# Conceptual notes

> [!tip] Global variables Vs Properties 
> A **global variable** is a piece of data stored in memory that is accessible and can be changed from anywhere within a program. This creates **tight coupling** because any part of the code can read or write to it, making it difficult to track changes and leading to unpredictable behavior. This is generally considered an anti-pattern in modern software development.
>
>A **properties file**, on the other hand, is a mechanism for **externalized configuration**. It separates configuration data (like database URLs or port numbers) from the application's source code. This data is loaded by a framework (like Spring) at startup and is typically **read-only** at runtime. The values are injected into specific, well-defined components. This design promotes **loose coupling** and makes the application easier to manage and deploy across different environments (e.g., development, testing, and production) without code changes.

> [!tip] Constants Vs properties
> Properties files differ from constants in their fundamental purpose and how they're handled within an application. **Constants** are fixed, unchanging values that are hard-coded directly into the source code and are typically used for **values that are universally true**, like mathematical constants (e.g., PI) or unchanging identifiers (e.g., `final int STATUS_SUCCESS = 1`). They are an integral part of the program's logic.
>
>In contrast, **properties files** are used for **configuration**, which refers to values that are expected to **change based on the environment or deployment context.** For example, a database URL or a server port will likely be different in a development environment compared to a production environment. Storing these values externally allows a developer to change these settings without recompiling or redeploying the application, promoting flexibility and a clean separation of concerns. This approach is a core principle of cloud-native and microservice architectures.

#core 