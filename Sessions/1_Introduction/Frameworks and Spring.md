# Frameworks
> [!definition]
> It is a pre-built structure that provides a foundation for developing software applications. Think of it like a **blueprint** or a **skeleton** for a house. Instead of having to design and build every wall, pipe, and wire from scratch, a framework **gives you the core structure and tools you need. You then fill in the custom parts—your specific business logic—on top of that foundation.**

The main benefits of using a framework are:

- **Speed:** It saves you from writing repetitive, boilerplate code for common tasks like handling web requests, connecting to a database, or managing security.
    
- **Consistency:** Frameworks enforce a standardised structure and best practices, which makes the code easier to read, maintain, and for a team to collaborate on as it uses design patterns.
    
- **Reliability:** The core components of a framework are typically well-tested and robust, reducing the chance of bugs and security vulnerabilities.

A key concept associated with frameworks is **"Inversion of Control" (IoC)**. This means the framework takes control of the application's flow, calling your code when it needs it. **This is the opposite of how you work with a simple library, where you explicitly call the library's functions from your code.**

### Popular frameworks
| Feature            | **Spring Boot**                                                                                                                                       | **Laravel**                                                                                                                                   | **Express.js**                                                                                                                                                                      |     |
| ------------------ | ----------------------------------------------------------------------------------------------------------------------------------------------------- | --------------------------------------------------------------------------------------------------------------------------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | --- |
| **Language**       | Java                                                                                                                                                  | PHP                                                                                                                                           | JavaScript                                                                                                                                                                          |     |
| **Philosophy**     | **Opinionated**                                                                                                                                       | **Developer-Friendly**                                                                                                                        | **Minimalist**                                                                                                                                                                      |     |
| **Core Strength**  | Robustness, Stability, Scalability                                                                                                                    | Rapid Application Development                                                                                                                 | High Concurrency, Performance                                                                                                                                                       |     |
| **Ideal For**      | Enterprise systems, Microservices, Large-scale applications                                                                                           | Web applications, CMS, E-commerce                                                                                                             | REST APIs, Real-time apps, SPAs                                                                                                                                                     |     |
| **Learning Curve** | **Steep:** Requires understanding of Java and the Spring ecosystem.                                                                                   | **Medium:** Elegant syntax, but has a lot of built-in components.                                                                             | **Low:** Simple API, but requires more manual setup.                                                                                                                                |     |
| **Advantages**     | - **Auto-configuration** reduces boilerplate. <br> - Huge ecosystem for complex needs. <br> - **Robust security** and performance.                    | - **Eloquent ORM** for easy database work. <br> - Artisan CLI for fast development. <br> - Batteries-included for common tasks.               | - **Blazing fast** for I/O-intensive tasks. <br> - Unifies front-end and back-end languages. <br> - **Extremely flexible** and lightweight.                                         |     |
| **Disadvantages**  | - Can have a **large memory footprint** and slow startup. <br> - Initial complexity can be intimidating. <br> - "Magic" can make debugging difficult. | - Can have **performance overhead**. <br> - Frequent major releases can require maintenance. <br> - Less ideal for large-scale microservices. | - **Lack of opinion** can lead to inconsistent codebases. <br> - No built-in features (e.g., ORM, authentication). <br> - Single-threaded nature can block **CPU-intensive tasks**. |     |
| **Analogy**        | A professional-grade toolkit for building a high-rise building 🧰.                                                                                    | A prefabricated house kit 🏡.                                                                                                                 | A powerful, customizable engine ⚙️.                                                                                                                                                 |     |
> [!tip]
> **Node.js** is a **JavaScript runtime environment**. It's the engine that allows you to run JavaScript code outside of a web browser on a server or a command line. It's the equivalent of the **Java Virtual Machine (JVM)**.

> [!tip]
>**Maven** and **npm** are both **package managers** and **build automation tools**. They are used to manage project dependencies, automate tasks like compiling code, running tests, and packaging applications.
> 
 >   - **Maven** is the standard for the Java ecosystem. It uses a **Project Object Model (POM)** file (`pom.xml`) to define project details and dependencies.
>        
  >  - **npm** (Node Package Manager) is the default package manager for Node.js. It uses a `package.json` file to list project dependencies and scripts.
  
*why are we making this comparison?*
In case any of you came from a different framework other than spring.

# Spring Framework
> [!definition] 
> It is one of the most popular and powerful frameworks in the Java ecosystem, **widely used in the development of enterprise, web, and microservices applications.** 
> 
> Originally created to facilitate the development of **decoupled** Java applications through **dependency injection**, the Spring Framework has evolved into a complete ecosystem with dozens of specialized modules.

[[Spring Structure.canvas|Spring Structure]]

> [!todo]
> **Research**
> - What is spring boot? 
> - How is it different from spring as in what makes it more powerful?
> - How to create a spring boot project
> - Does spring boot’s configurations make the project rigid and limit customisation?

> [!seealso]
> ```embed
title: "How Spring Boot Auto Configuration Works with Spring's @EnableAutoConfiguration Annotations"
image: "https://i.ytimg.com/vi/6u6PJXTb1cQ/maxresdefault.jpg"
description: "One of the most compelling features of Spring and Spring Boot is AutoConfiguration.The Spring EnableAutoConfiguration annotation, and the various AutoConfig ..."
url: "https://www.youtube.com/watch?v=6u6PJXTb1cQ"
favicon: ""
aspectRatio: "56.25"
>```

#conceptual 