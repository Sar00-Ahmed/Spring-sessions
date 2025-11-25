## What is a Web Application?


``` mermaid
flowchart LR
    subgraph A [Client Side]
        Browser
    end

    subgraph B [Network]
        Internet
    end

    subgraph C [Server Side]
        WebServer
        AppServer[Application Server]
    end

    Browser -- "HTTP Request" --> Internet
    Internet -- "Request" --> WebServer
    WebServer -- "Request" --> AppServer

    AppServer -- "Response" --> WebServer
    WebServer -- "Response" --> Internet
    Internet -- "HTTP Response" --> Browser
```
##### Web Application
> [!definition]
> Is a **software program that runs on a remote server** and is accessed through a web browser over the internet. Unlike traditional apps that require installation on your device.
    
##### Web Server
A web server (like Nginx or Apache) **serves static content**, while an application server (like Tomcat or JBoss) runs and manages the business logic of your application.

| Web Server                                                 | Application Server                       |
| ---------------------------------------------------------- | ---------------------------------------- |
| Handles user requests and directs them to the right place. | Processes tasks and generates responses. |
| Apache HTTP Server, Nginx, etc..                           | Apache Tomcat, WildFly                          |


> [!note]
> **Nginx** also provides load balancer, and HTTP cache, making it a common choice for high-traffic websites and microservices architectures.

## WAR vs. JAR
Describe the packaging formats. 

A **WAR** (Web Application Archive) is for web applications, **meant to be deployed on an application server.** 

A **JAR** (Java Archive) is a standalone, executable file that includes everything needed to run the application, including an embedded server. 
- - - 

> [!todo]
> **Research**
> - What is MVC architecture?
> - Does spring use MVC? How?
> - Does our stack angular and spring follow the MVC architecture?

# References

> [!cite]
> This contains a summary of what modules spring contains and what they do:
> https://www.geeksforgeeks.org/advance-java/introduction-to-spring-framework/

> [!cite]
> contains a list of annotations
> https://dev.to/pablocavalcanteh/spring-framework-overview-of-main-modules-and-annotations-4b12

> [!cite]
> Dispatcher servlet
> https://medium.com/@vino7tech/understanding-dispatcherservlet-in-spring-mvc-f49e034de016

> [!cite]
> Spring Architecture
> https://www.geeksforgeeks.org/springboot/spring-boot-architecture/
> https://medium.com/@udaypatil318/spring-boot-architecture-39935654ce5c

#conceptual 