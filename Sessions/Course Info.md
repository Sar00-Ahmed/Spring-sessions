# Introduction
- name
- background/ experience
- unrelated or loosely related skill
- Expectations from training
# Trainer
> [!profile] Sara Ahmed 
> Junior fullstack developer in ntg apps
> Other skills: graphic design, teaching skills

# Objectives
- Get a full conceptual view of spring’s core components
- Be able to use spring framework and easily extend it
- Explain complex topics that often take time to understand or have little resource on especially when you don’t know what you are looking for.
- leetcode sql problems
## Secondary objectives
- Learning to use **Git**
- Learning to use **Postman** for testing
- Learning to use **Docker** (pull and use db images)
- Learning to use **PGadmin**
- Learning to use some **Intelliji tools for debugging**
- Learning some presentation skills
- Learning some organisational skills
# Methods
- Conceptual explanation.
- Summary presentations at the beginning of every session
- Milestone quizzes
- Project sessions at the very end
	- involves them pushing code to central git  repo
- Code samples
- coding session at the end of milestones
- For Highly skilled individuals → advanced topics that will not be covered will be assigned for them to research and present.
# Not covered
- I will not give you documentation on every spring feature because this is impossible rather, when you need something search for it. we will focus on core components only.
- I will not dive into how spring works on the inside rather focus on how to use this black box as much as possible. Though we may need to do this for some topics.

# Course Guide: Choose Your Learning Path

This course is designed for developers at different stages of their journey. Please select the path that best describes you to get the most out of the material.

| Your Level                       | Your Focus                                                                                                                                            |
| -------------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------- |
| **New to Web Development**       | Concentrate on the core conceptual topics to build a strong foundation.                                                                               |
| **Web Developer, New to Spring** | Follow the main topics and focus on applying the explained concepts. You can safely ignore "Further Reading," "References," and "Appendices" for now. |
| **Familiar with Spring**         | You are encouraged to explore the "References" and "Appendices"—you will find them highly valuable for deepening your knowledge.                      |

### Other Common Scenarios

##### What if I'm new to Java?
If you have a background in Object-Oriented Programming (OOP), you should be fine. You may just need a little extra time to get accustomed to Java's syntax.

##### What if I'm new to programming?
**I do not recommend starting with a framework like Spring.** While it is technically possible, it requires a tremendous amount of self-learning and extra effort to catch up on fundamental programming concepts. It is better to learn the basics of programming first.

##### Why can’t we start with full project code?
This is similar to the question of why do we start conceptual. 

- A developer must learn concepts because they are universal across frameworks and languages. This approach does not just give you spring in your pocket but any other framework you might want to learn in the future.
- Spring is a huge Black Box with continuous updates that change its syntax. if you are fixated on code at the start when errors and bugs appear you will be stumped.
- Spring also is a full structure that starts once its fully configured. You can’t implement a part and leave another to test. and giving a full implementation will create more questions than it answers

but if you must I would recommend watching a one hour video for anyone that goes through a full project, no need to fully understand it. All you need is an overview.
# Schedule

| Session # | Title                           | Content                                                                                                                                                                                                                                    | Assignments                                                                            | Advanced                                             |
| --------- | ------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ | -------------------------------------------------------------------------------------- | ---------------------------------------------------- |
| 1         | Introduction                    | -Web app vs web server & request life cycle<br>- A comparison between frameworks<br>- IOC & dependency injection<br>- Spring containers, beans and configs                                                                                 | research topics<br><br>watch a short amigos code video  creating a full project        | AOP<br>Logging: sl4j & md5<br><br>Custom annotations |
| 2         | Controllers                     | - Http request<br>- Spring Controllers<br>- DTOs and Validations<br>- Interceptors<br>- Global Exception Handler<br>- Using postman                                                                                                        | creating a sample controller that returns static hard coded data                       | Using swagger for documentation                      |
| 3         | DB                              | - ERD diagram<br>- DDL, DML, DQL<br>- Nested queries <br>- Aggregation<br>- Solve leetcode problems                                                                                                                                        | Design an ERD diagram for a complex system with given requirements                     |                                                      |
| 4         | Orm                             | - History of: ORM, hibernate, jpa and spring<br>- Entities<br>- Creating relations<br>- Bidirectional vs unidirectional<br>- serialisation fix                                                                                             | research topics                                                                        |                                                      |
| 5         | Practice Project 1              | create entities for ERD diagram that was designed                                                                                                                                                                                          | reasearch topics<br>+<br>create the rest of entities for ERD diagram that was designed |                                                      |
| 6         | Repositories                    | - creating repositories<br>- JPQL vs native queries<br>- Pagination<br>- Optional return type                                                                                                                                              | full CRUD for an entity                                                                |                                                      |
| 7         | Practice project 2              |                                                                                                                                                                                                                                            |                                                                                        |                                                      |
| 8         | service layer                   | - Entity life cycle<br>- Caching                                                                                                                                                                                                           |                                                                                        | Transactional                                        |
| 9         | Security                        | - Encryption, Authentication, Authorisation<br> - JWT authentication                                                                                                                                                                       | add security to previous project                                                       | Advanced: Search for OAuth                           |
| 10        | Practice project 3              |                                                                                                                                                                                                                                            |                                                                                        |                                                      |
| 11        | Backend callenges and solutions | - Race conditions in multi threading (DB locking)<br>- Efficient Large Data Processing (JDBC streaming, pagination)<br>- Email verification request time (Async operations)<br>- Email notifications (message queues)<br>- scheduled tasks | research                                                                               |                                                      |





