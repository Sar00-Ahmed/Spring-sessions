# IOC
> [!definition]
> Inversion of Control is a programming **design principle** that inverts the flow of control to achieve some design purposes.

- To decouple the execution of a task from implementation.
- To focus a module on the task it is designed for.
- To free modules from assumptions about how other systems do what they do and instead rely on contracts.
- To prevent side effects when replacing a module.

**This is to avoid tight coupling between classes.**

You can also register a **listener** to a framework and hand over the control to the framework. Then, let the framework do the job for you and call back to your listener on a specific event that happens. In other words, the flow of control is inverted from you to the framework. Sometimes, IoC is facetiously referred to as the **“Hollywood Principle: Don’t call us, we’ll call you”**.

![[Pasted image 20250914165409.png]]

## Dependency Injection

> [!definition]
> This is the practical implementation of IoC. It's the process of supplying a dependency (a "bean") to a component at runtime. This can be done via constructor injection (the preferred method), setter injection, or field injection.

![[Pasted image 20250914164329.png]]
![[Pasted image 20250914164409.png]]

### Types of dependency injection

Dependency injection provides at least three ways to inject a dependency:


> [!todo]
> **Implement**
>Imagine a `NotificationService` that needs to send emails, and you have two different implementations: a `GmailService` and a `YahooService` That need to be specified at run time depending on user Input.


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
Field injection is the most common but least recommended type. The dependent objects are injected directly into a class through a field. 

``` Java
public class NotificationService {
    public EmailService emailService;

    public void sendWelcomeNotification(String userEmail) {
        emailService.sendEmail(userEmail, "Welcome!", "Thanks for joining!");
    }
}

public class Sender{
	
	private NotificationService notificationService
	
	public sendNotification(){
		notificationService.emailService = new GmailService();
		notificationService.sendWelcomeNotification("user@gmail.com");
	}
}
```
##### Critique
This approach is succinct but violates the **encapsulation principle** in object-oriented programming because the `private` field is being set from outside the class. 
#### 2. Setter injection
Setter injection uses a public setter method to inject the dependency. This is a good choice for **optional dependencies** or when you need the ability to change the dependency at runtime.

``` Java
public class NotificationService {
    private EmailService emailService;

    // Public setter to inject the dependency
    public void setEmailService(EmailService emailService) {
        this.emailService = emailService;
    }

    public void sendWelcomeNotification(String userEmail) {
        if (emailService != null) {
            emailService.sendEmail(userEmail, "Welcome!", "Thanks for joining!");
        }
    }
}

// How you would use it
// NotificationService notificationService = new NotificationService();
// notificationService.setEmailService(new YahooService());
// notificationService.sendWelcomeNotification("user@example.com");
```
##### Critique
You've correctly identified that this allows for changing behavior at runtime. This can be a benefit, but it also means the object is mutable and might exist in an invalid state if the setter is never called, making it **not ideal for mandatory dependencies**.
#### 3. Constructor injection
Constructor injection uses a constructor to receive and set the dependency. This is the **most recommended** approach by the Spring team because it ensures that an object is created with all its required dependencies, making it immutable and easy to test.

``` Java
	public class NotificationService {
    private final EmailService emailService;

    // The dependency is injected through the constructor
    public NotificationService(EmailService emailService) {
        this.emailService = emailService;
    }

    public void sendWelcomeNotification(String userEmail) {
        emailService.sendEmail(userEmail, "Welcome!", "Thanks for joining!");
    }
}

// How you would use it
// EmailService yahooService = new YahooService();
// NotificationService notificationService = new NotificationService(yahooService);
// notificationService.sendWelcomeNotification("user@example.com");
```

Constructor injection makes dependencies **explicit and mandatory**.
# References
> [!cite]
>```embed
title: "Design Patterns"
image: "https://refactoring.guru/images/refactoring/social/facebook-share-preview.png?id=dbf9e98269595be86eb668f365be6868"
description: "Design Patterns are typical solutions to commonly occurring problems in software design. They are blueprints that you can customize to solve a particular design problem in your code."
url: "https://refactoring.guru/design-patterns"
favicon: ""
aspectRatio: "52.3109243697479"
>```

> [!cite] 
>```embed
title: "Dependency Injection(DI) Design Pattern - GeeksforGeeks"
image: "https://media.geeksforgeeks.org/wp-content/cdn-uploads/gfg_200x200-min.png"
description: "Your All-in-One Learning Portal: GeeksforGeeks is a comprehensive educational platform that empowers learners across domains-spanning computer science and programming, school education, upskilling, commerce, software tools, competitive exams, and more."
url: "https://www.geeksforgeeks.org/system-design/dependency-injectiondi-design-pattern/"
favicon: ""
aspectRatio: "100"
>```

#conceptual 