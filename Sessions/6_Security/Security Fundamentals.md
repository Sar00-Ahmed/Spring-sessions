## Why security matters

Every system **connected to the internet is a potential target**. Attackers exploit vulnerabilities to steal data, hijack sessions, or disrupt services.


Securing your website is essential consequences can include
##### You and your customer’s information could be at risk

> [!quote]
> If a hacker gets his or her hands on this information, it would be fairly easy to **steal their identity and make fraudulent purchases.**

##### Revenue loss

> [!Quote] 
> When you have to send out a message to customers stating that you were the victim of a data breach and their financial information may have been compromised, **they are going to think twice before doing business with you again**.
If your site is marked as a security risk, you are going to not only have a reputation as a risk for customers, but also a risk for other websites.

##### cleanup is more expensive
##### Legal issues
In some cases, breaches may also result in legal penalties for failing to comply with data protection regulations like **GDPR**, for example.

## Should I make the jump from dev to cybersecurity?

- **Pin testers** test the system and provide reports of system vulnerabilities. you read it and then secure your system.
- **Frameworks** provide built in security developed by experts.

> [!Warning]
> 🚫 **Avoid DIY anything in security** (unless you really know what you’re doing) 

Your system will need to include some basic security:
	- encryption
	- authentication
	- authorisation

## Core concepts
### Hashing vs. Encryption

| **Hashing**                                                         | **Encryption**                                 |
| ------------------------------------------------------------------- | ---------------------------------------------- |
| **One-way** process (cannot be reversed).                           | **Two-way** process (can be decrypted).        |
| Used for **data integrity** (e.g., passwords, verifying downloads). | Used for **confidentiality** (e.g., messages). |
| Examples: `BCrypt`, `SHA-256`.                                      | Examples: `AES`, `RSA`.                        |


> [!NOTE] Self study
> What is symmetric and asymmetric encryption? 

### Authentication
check that you are who you claim to be
- **Passwords:** User provides a secret password → system compares it to a stored hash.
- **Tokens:** 
	1. User authenticates (e.g., with password) → server issues a token (e.g., JWT).
	2. Token is saved client-side (in cookies/localStorage).
	3. Token is sent with each request → server validates it.

> [!NOTE] Self study
>- two factor authentication
>- certificates
>- public private keys
>- OAuth
### Authorisation
Controlling what authenticated users can **access** based on their roles and permissions.

#### Role-Based Access Control (RBAC)
**How it works**:
- Each user is assigned a **role** (e.g., `ADMIN`, `USER`, `GUEST`).
- Roles have predefined **authorities** (permissions like `READ`, `WRITE`, `DELETE`).

---

#conceptual