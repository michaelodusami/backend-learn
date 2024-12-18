### **What is CSRF (Cross-Site Request Forgery)?**

**CSRF (Cross-Site Request Forgery)** is a type of web security attack where a malicious actor tricks a user into performing an unwanted action on a trusted website, where the user is already authenticated. It exploits the trust that the website has in the user’s browser.

---

### **How Does CSRF Work?**
1. **User Logs In:**
   - A user logs into a legitimate website (e.g., a bank) and their browser stores a session or authentication token (like a cookie).

2. **Malicious Website Exploits the Login:**
   - The user visits a malicious website while still logged into the trusted website in another tab.
   - The malicious website tricks the browser into making a request to the trusted website (e.g., transferring money, deleting an account).

3. **Request Executes as the User:**
   - Since the user is already logged in, the trusted website thinks the request is legitimate, and the malicious action is performed.

---

### **In Layman’s Terms**

Think of CSRF as a **fraudulent action by someone impersonating you**:
1. You’re logged into your bank account on one tab.
2. In another tab, you visit a shady site that secretly sends a request to your bank to transfer money to the attacker’s account.
3. The bank trusts your session and processes the transfer, even though you never intended to do it.

---

### **Example of a CSRF Attack**

Imagine a bank website has a URL for transferring money:
```
POST https://examplebank.com/transfer
Body: recipient=attacker&amount=1000
```

An attacker creates a malicious form:
```html
<form action="https://examplebank.com/transfer" method="POST">
    <input type="hidden" name="recipient" value="attacker">
    <input type="hidden" name="amount" value="1000">
    <button type="submit">Click Here!</button>
</form>
```

When the user, who is already logged into their bank account, clicks the button, the browser sends the request to the bank, and money is transferred without their consent.

---

### **How Does Spring Security Protect Against CSRF?**

Spring Security provides **CSRF protection** by generating a unique **CSRF token** for each session or request. This token must be included in all state-changing requests (e.g., POST, PUT, DELETE). The server validates this token before processing the request.

1. **CSRF Token Generation:**
   - A unique CSRF token is generated when a user’s session starts.
   - The token is sent to the client (browser) in the HTML response.

2. **CSRF Token Validation:**
   - For state-changing requests (POST, PUT, DELETE), the client must include the CSRF token in the request (typically as a hidden form field or HTTP header).
   - If the token is missing or doesn’t match, the server rejects the request.

---

### **CSRF Protection Workflow**

1. **User Visits Website:**
   - A CSRF token is included in the response (e.g., as a hidden field in forms or as a header).

2. **User Submits a Request:**
   - The request includes the CSRF token.

3. **Server Validates the Token:**
   - If the token matches the one stored on the server, the request is processed.
   - If the token is missing or invalid, the server rejects the request.

---

### **Enabling CSRF Protection in Spring Security**

CSRF protection is enabled by default in Spring Security. Here's a basic configuration:

#### **1. Default CSRF Protection**
```java
@Configuration
@EnableWebSecurity
public class SecurityConfig extends WebSecurityConfigurerAdapter {

    @Override
    protected void configure(HttpSecurity http) throws Exception {
        http
            .csrf().and() // CSRF protection is enabled by default
            .authorizeRequests()
            .antMatchers("/login", "/register").permitAll()
            .anyRequest().authenticated();
    }
}
```

#### **2. CSRF Token in Forms**
When CSRF is enabled, Spring Security automatically adds a hidden CSRF token field to HTML forms created with the Spring Framework:
```html
<input type="hidden" name="_csrf" value="unique-csrf-token">
```

#### **3. Sending CSRF Token with AJAX**
For AJAX requests, you need to include the CSRF token in the request headers:
```javascript
$.ajax({
    url: "/example",
    type: "POST",
    headers: {
        'X-CSRF-TOKEN': $('meta[name="_csrf"]').attr('content')
    },
    data: {
        key: "value"
    },
    success: function(response) {
        console.log("Success");
    }
});
```

---

### **Disabling CSRF Protection**
In some cases, such as for public APIs, you might want to disable CSRF protection:
```java
@Configuration
@EnableWebSecurity
public class SecurityConfig extends WebSecurityConfigurerAdapter {

    @Override
    protected void configure(HttpSecurity http) throws Exception {
        http
            .csrf().disable() // Disables CSRF protection
            .authorizeRequests()
            .antMatchers("/public-api/**").permitAll()
            .anyRequest().authenticated();
    }
}
```

---

### **Best Practices to Avoid CSRF Attacks**
1. **Enable CSRF Protection:**
   - Always enable CSRF protection for state-changing actions.

2. **Validate CSRF Tokens:**
   - Ensure the server validates CSRF tokens for every request.

3. **Use SameSite Cookies:**
   - Set cookies to `SameSite=Lax` or `SameSite=Strict` to restrict their use in cross-site requests.

4. **Avoid GET for State Changes:**
   - Only use POST, PUT, or DELETE for actions that change data.

5. **Custom Headers for APIs:**
   - For REST APIs, use custom headers to validate requests instead of relying solely on cookies.

---

### **Example of CSRF-Enabled Application**

#### **Configuration**
```java
@Configuration
@EnableWebSecurity
public class SecurityConfig extends WebSecurityConfigurerAdapter {

    @Override
    protected void configure(HttpSecurity http) throws Exception {
        http
            .csrf().and()
            .authorizeRequests()
            .antMatchers("/login", "/register").permitAll()
            .anyRequest().authenticated();
    }
}
```

#### **Login Form**
```html
<form action="/login" method="POST">
    <input type="text" name="username" placeholder="Username" />
    <input type="password" name="password" placeholder="Password" />
    <input type="hidden" name="_csrf" value="${_csrf.token}" />
    <button type="submit">Login</button>
</form>
```

#### **Token in Response Header**
You can include the CSRF token in a meta tag for JavaScript to use:
```html
<meta name="_csrf" content="${_csrf.token}">
<meta name="_csrf_header" content="${_csrf.headerName}">
```

---

### **Summary**
| **Feature**         | **Description**                              |
|----------------------|----------------------------------------------|
| **What is CSRF?**    | An attack that tricks authenticated users into performing unwanted actions. |
| **Protection in Spring** | Validates a CSRF token for state-changing requests. |
| **Default Behavior** | Enabled by default in Spring Security.       |
| **Use Cases**        | Necessary for web forms, not typically for APIs. |
| **Best Practices**   | Enable CSRF protection, use secure cookies, and validate tokens. |

