# XSS
> [!definition]
> A vulnerability where an attacker injects **malicious JavaScript** into your website so it runs in the victim’s browser.

**Goal of attacker:**
- Steal cookies
- Steal tokens
- Deface UI
- Perform actions as the user
- Redirect user

XSS can happens when **untrusted user input is inserted into a page without escaping**.
Knowing that Angular normally esacpes this (_Unless if you used innerHTML_)

But does that mean Back end doesn’t need to sanitise input?

User posts a comment to your API:

``` json
{   
	"comment": "<script>alert('hacked')</script>" 
}
```

Your backend stores this **unmodified** in the database.
Then Angular fetches and displays it using:

``` html
{{ comment }}
```

Angular escapes it → safe.
But what if any developer **accidentally** uses:

``` html
<div [innerHTML]="comment"></div>
```

Boom → **XSS**, and the payload came directly from your backend. 
# CSRF
> [!definition]
>  vulnerability where an attacker tricks the user’s browser into sending **authenticated requests** to your site **without the user knowing**. CSRF is about _using the victim’s existing session_ to make **unwanted requests**.

**Key requirement:**
- The victim must be **logged in** to the target site.
- Browser must automatically send **cookies**.
## Fix
``` Java
@Bean
SecurityFilterChain securityFilterChain(HttpSecurity http) throws Exception {
    http
        .csrf(csrf -> csrf.enable())
        .authorizeHttpRequests(auth -> auth.anyRequest().authenticated());
    return http.build();
}

```

On every session-based POST/PUT/DELETE request Spring expects a CSRF token. If missing → 403 Forbidden

Spring sends a CSRF token in cookies to front-end `XSRF-TOKEN`
Angular **automatically** reads the cookie `XSRF-TOKEN`  and sends it as a header: `X-XSRF-TOKEN: <value>` this means it does not require configuration on angular’s side.

> [!tip]
> In a stateless app storing your JWT token  in local storage or session storage bypasses this issue entirely because browsers don’t add them automatically in requests.
# References
> [!cite]
>```embed
title: "Do cookies protect tokens against XSS attacks?"
image: "https://stackoverflow.com/Content/Sites/stackoverflow/Img/apple-touch-icon@2.png?v=73d79a89bded"
description: "I'm building a JWT-based (JSON Web Token)  authentication mechanism for an browser-based Javascript web app, working with a stateless server (no user-sessions!) and I want to know, once and for all..."
url: "https://stackoverflow.com/questions/36980058/do-cookies-protect-tokens-against-xss-attacks"
favicon: ""
aspectRatio: "100"
>```


#conceptual 