# IoC Containers
> [!definition]
> This is the heart of the Spring Framework. It's responsible for managing the lifecycle of your application's objects (beans) and implementing DI. 


The container handles the entire lifecycle of a bean, including:
1. Instantiation: Creating the object.
2. Configuration: Injecting its dependencies.
3. Wiring: Connecting it to other beans.
4. Destruction: Managing its removal when the application shuts down.

 The two main types are `BeanFactory` and `ApplicationContext`.

> [!definition] Bean Factory
> This is the most basic and fundamental IoC (Inversion of Control) container in Spring. It provides the core functionality of bean management, including instantiation, configuration, and DI. 
> Think of it as a simple, no-frills factory that creates objects only when you explicitly ask for them. This is known as **lazy initialization**, which is a key characteristic.
    

> [!definition] ApplicationContext
> This is an enhanced and more powerful version of the BeanFactory. In fact, it is a **superset** of the BeanFactory, meaning it includes all of the BeanFactory's functionality plus a lot more. 
> It is designed for building **enterprise-level applications** and is the container used by default in Spring Boot. It provides features beyond just bean management.    

### The Key Differences

The main distinction between the two lies in their feature set and how they handle bean initialization.

| Feature               | BeanFactory                                                                                                                    | ApplicationContext                                                                                       |
| --------------------- | ------------------------------------------------------------------------------------------------------------------------------ | -------------------------------------------------------------------------------------------------------- |
| Definition            | Fundamental container providing basic functionality for managing beans.                                                        | Advanced container **extending BeanFactory with additional features.**                                   |
| Usage                 | Suitable for building standalone applications.                                                                                 | Suitable for building web applications, integrating with AOP modules, ORM, and distributed applications. |
| Bean Scopes Supported | Supports **only Singleton and Prototype** bean scopes.                                                                         | Supports all types of bean scopes, including Singleton, Prototype, Request, Session, etc.                |
| Annotation Support    | Does not support annotations; requires configuration in XML files.                                                             | **Supports annotation-based configuration for bean autowiring.**                                         |
| Internationalization  | Does not provide internationalization (i18n) functionality.                                                                    | Extends MessageSource interface to provide internationalization (i18n) functionality.                    |
| Event Handling        | Does not support event publication.                                                                                            | Supports event handling via the `ApplicationEvent` class and `ApplicationListener` interface.            |
| Bean Post Processing  | Requires manual registration of `BeanPostProcessors` and `BeanFactoryPostProcessors`.                                          | Automatically registers `BeanFactoryPostProcessor` and `BeanPostProcessor` at startup.                   |
| Initialization        | Creates bean objects on demand using **lazy initialization**. Beans are created only when the `getBean()` method is called.    | Loads all beans and creates objects at startup using **eager initialization**.                           |
| Resource Usage        | Provides basic features requiring less memory, **suitable for memory-critical** standalone applications like embedded systems. | Provides basic and advanced features, suitable for **enterprise applications**, requiring more memory.   |

In short, while `BeanFactory` is the foundational concept, the `ApplicationContext` is what you will almost always use in practice, especially with Spring Boot. Spring Boot's `@SpringBootApplication` annotation automatically configures and launches an `ApplicationContext` for you, handling all the complex setup behind the scenes.

# Types of dependency injection

Dependency injection provides at least three ways to inject a dependency:

Imagine a `NotificationService` that needs to send emails, and you have two different implementations: a `GmailService` and a `YahooService`.

``` Java
// The common interface
public interface EmailService {
    void sendEmail(String to, String subject, String body);
}

// Implementation 1
@Service("gmailService")
public class GmailService implements EmailService {
    @Override
    public void sendEmail(String to, String subject, String body) {
        System.out.println("Sending email via Gmail to " + to);
    }
}

// Implementation 2
@Service("yahooService")
public class YahooService implements EmailService {
    @Override
    public void sendEmail(String to, String subject, String body) {
        System.out.println("Sending email via Yahoo to " + to);
    }
}
```
#### 1. Field injection
Field injection is the most common but least recommended type. The dependent objects are injected directly into a class through a field using the `@Autowired` annotation. This approach is succinct but violates the **encapsulation principle** in object-oriented programming.


``` Java
@Service
public class NotificationService {
    @Autowired
    @Qualifier("gmailService")
    private EmailService emailService;

    public void sendWelcomeNotification(String userEmail) {
        emailService.sendEmail(userEmail, "Welcome!", "Thanks for joining!");
    }
}
```
##### Critique
You're right, this violates **encapsulation** because the `private` field is being set from outside the class by the framework, bypassing the class's own control. It also makes your class difficult to unit test without a Spring container, as you can't instantiate it with a simple constructor.
#### 1. Setter injection

``` Java
@Service
public class NotificationService {
    private EmailService emailService;

    @Autowired
    @Qualifier("yahooService")
    public void setEmailService(EmailService emailService) {
        this.emailService = emailService;
    }

    public void sendWelcomeNotification(String userEmail) {
        emailService.sendEmail(userEmail, "Welcome!", "Thanks for joining!");
    }
}
```

#### 1. Constructor injection
Constructor injection uses a constructor to receive and set the dependency. This is the **most recommended** approach by the Spring team because it ensures that an object is created with all its required dependencies, making it immutable and easy to test.

``` Java
@Service
public class NotificationService {
    private final EmailService emailService;

    @Autowired // Optional since Spring 4.3 for classes with a single constructor
    @Qualifier("gmailService")
    public NotificationService(EmailService emailService) {
        this.emailService = emailService;
    }

    public void sendWelcomeNotification(String userEmail) {
        emailService.sendEmail(userEmail, "Welcome!", "Thanks for joining!");
    }
}
```

Constructor injection makes dependencies **explicit and mandatory**. By using the `final` keyword, you can guarantee that the dependency cannot be changed after instantiation, which is a significant benefit for immutability. This design makes the class incredibly easy to test and reason about.

> [!error]
> If Spring encounters multiple beans that match the type of a dependency you're trying to autowire, it will throw a `NoUniqueBeanDefinitionException`.
> `Caused by: org.springframework.beans.factory.NoUniqueBeanDefinitionException:`
>`No qualifying bean of type ....:`
>`expected single matching bean but found 2:...`

# Auto-wire an Interface With Multiple Implementations

>[!definition]
> In Spring Boot, auto-wiring refers to the automatic injection of dependencies. When a class depends on an interface, Spring attempts to inject an implementation of that interface. 

However, when there are multiple implementations of a single interface, Spring needs clarification on which implementation to inject.
### ## Solving Ambiguity with `@Qualifier`
``` Java
@Service  
public class OrderService {  
  
	@Autowired  
	@Qualifier("paypalPaymentService") // Specify the bean name to inject 
	private PaymentService paymentService;  
  
	public void processOrder(double amount) {  
	paymentService.processPayment(amount);  
	}  
}
```

### Using `@Primary` to Set a Default Bean

The `@Primary` annotation marks a bean as the default candidate for injection when multiple beans of the same type are present. Let’s modify our example by marking `CreditCardPaymentService` as the primary bean:

``` Java
@Service  
@Primary // Mark this bean as the primary one  
public class CreditCardPaymentService implements PaymentService {  
    @Override  
    public void processPayment(double amount) {  
        System.out.println("Processing credit card payment of $" + amount);  
    }  
}  
  
@Service  
public class PaypalPaymentService implements PaymentService {  
    @Override  
    public void processPayment(double amount) {  
        System.out.println("Processing PayPal payment of $" + amount);  
    }  
}
```

> [!todo]
> Research
> - What is bean lifecycle?
> - How  and when to use:
> 	- `@PostConstruct` annotation or by implementing the `InitializingBean` interface.
> 	- `@PreDestroy` annotation or by implementing the `DisposableBean` interface.
# References
> [!cite]
> Application context vs Bean factory
> 
>```embed
title: "Understanding BeanFactory and ApplicationContext in Spring Boot: A Deep Dive"
image: "https://miro.medium.com/v2/resize:fit:1200/1*cd1Oh_X3nqcG_iEoFwpziw.png"
description: "When working with Spring Boot, you’ll often encounter terms like BeanFactory and ApplicationContext. These are the backbone of Spring’s…"
url: "https://medium.com/@bolot.89/understanding-beanfactory-and-applicationcontext-in-spring-boot-a-deep-dive-f7f92f3a16f5"
favicon: ""
aspectRatio: "46.33333333333333"
>```

#core 