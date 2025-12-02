# Race conditions in multi threading
> [!quote] Consider the following:
> Three users sent a request at the same time to the back end of a flight booking system. one user only should be able to book the seat. In a multi threaded web environment, a separate thread is created for each request. 
> **How can you prevent multiple tickets from being raised for every requester?**

> [!quote] Consider the following:
> Three users sent a request at the same time to the back end of a HR system. 5 users only should be able to book an interview per day. In a multi threaded web environment, a separate thread is created for each request. 
> **How can you prevent more than 5 interviews from being raised?**
# # Efficient Large Data Processing
> [!quote] Consider the following:
>When working with large datasets in a database, two common challenges arise:
>**Out-of-Memory (OOM) Errors**  
>- Loading an entire table (e.g., millions of rows) into RAM causes memory exhaustion.  
>**Pagination Issues with Concurrent Writes**  
   >- Traditional `LIMIT/OFFSET` pagination breaks when data changes between page fetches.  
 >  - Example:  
>```sql
-- Page 1: Rows 1-100
SELECT * FROM table ORDER BY id LIMIT 100 OFFSET 0;  
-- If a new row is inserted, Page 2 (OFFSET 100) may skip or duplicate rows.
>```
>**How can we have  unbroken pages without overloading the backend?** 
# Email verification request time
> [!quote] Consider the following:
> A user sends a registration request for the back end. The back end should register the user, send a verification email and return a response to the user.
> The problem is: 
> - Email sending takes time and will slow down the request. 
> - Not only that but the sending may fail due to network connection
> **How can we send the email without slowing down the request and avoid connection problems?**
# Emails notification to subscribed users
> [!quote] Consider the following:

In a flight booking system, you need to send email notifications to subscribed users when seats become available at specific times (e.g., price drops, seat releases, or promotional periods). Which causes Application crashes due to resource exhaustion

##### Email Service Provider Limitations
- **Gmail/Google Workspace**:
    - 500 emails per day (free) / 2,000 per day (paid)
    - Rate limit: 100 recipients per message
    - **Account suspension risk** if limits are exceeded repeatedly
- **SMTP Server Limits**: Most providers impose strict hourly/daily limits
- **IP Reputation Damage**: Sending too many emails quickly can blacklist your IP
- **Spam Filter Triggering**: Sudden email volume spikes trigger spam filters

> **How can we send these emails without overloading the back end?**
> **Email System Must:**
> 1. **Respect Provider Limits**: Never exceed Gmail/SMTP rate limits 
> 2. **Handle 10,000+ Emails**: Scale for large subscriber bases
> 3. **Guarantee Delivery**: Ensure important notifications are never lost 
> 4. **Prevent Account Lockouts**: Implement smart retry with exponential backoff 
> 5. **Maintain IP Reputation**: Space emails naturally, avoid spam patterns

# Orphaned children in many to many relations
> [!quote] Consider the following:
> In an e-commerce system, tags of products need to be stored to optimise search. the relation between the products and tags are many-to-many. It is required to have a page of tags organised alphabetically and by number of relations. 
> 
> The issue is, if all products  of a certain tags are detached from it such that this tag has become orphaned when a user click on this tag for search an empty page appears which is not a nice user experience.
> 
> How can we ensure these orphaned tags do not appear in search results?
# Send overdue Notices
> [!quote] Consider the following:
> It is required to send notifications to all users in the database 1 day before their book return is overdue. 
> How can we implement this?
# Handling Batch failures
> [!quote] Consider the following:
> Say you use JDBC template to send batch operations to the database. but once a procedure fails the whole process fails with no way to tell which operation exactly failed. How can we ensure that one failure does not affect the others or implement retry mechanisms that does not repeat the whole operation?

#conceptual 