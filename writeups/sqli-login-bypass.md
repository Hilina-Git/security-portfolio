# SQL Injection - Authentication Bypass via Login Form

**Finding ID:** VULN-2026-002  
**Severity:** Critical  
**CVSS Score:** 9.8  
**Date:** May 7, 2026  
**Tester:** Hilina Gebre  
**Tools Used:** Burp Suite Community Edition  
**Status:** Confirmed / Resolved (Lab Environment)

---

## Vulnerability Description

SQL Injection is a critical web application vulnerability where an attacker injects malicious SQL code into user-supplied input fields to manipulate database queries. This can allow unauthorized access to sensitive data, bypass authentication, modify records, or delete database contents.

In this finding, the application's login form passes the username field directly into a SQL authentication query without sanitization. This allows an attacker to comment out the password verification logic entirely - bypassing authentication and gaining administrative access without any valid credentials.

This vulnerability is rated Critical because:
- It allows complete authentication bypass with no valid credentials required
- It grants administrative-level access to the application
- It could expose all user data, session tokens, and sensitive records
- It requires no special expertise and is exploitable remotely by any unauthenticated attacker

---

## Impact

- Full authentication bypass as any user including administrator
- Unauthorized access to sensitive user data and application functionality
- Potential for account takeover, data exfiltration, and privilege escalation
- Complete compromise of application access controls

---

## Steps to Reproduce

1. Navigate to the application login page at `/login`
2. Open Burp Suite and ensure Intercept is ON under the Proxy tab
3. In the Username field enter:
```
administrator'--
```
4. In the Password field enter any value (e.g. anything)
5. Click Log In
6. In Burp Suite intercept, observe the POST request body:
```
username=administrator'--&password=anything
```
7. Forward the request
8. Observe that authentication is bypassed and access is granted as the administrator account with no valid password

---

## What Happened Behind the Scenes

The application constructs the following SQL query using unsanitized input:

```sql
SELECT * FROM users WHERE username='[input]' AND password='[input]'
```

After injecting the payload `administrator'--`, the query becomes:

```sql
SELECT * FROM users WHERE username='administrator'--' AND password='anything'
```

- `'` closes the username string
- `--` comments out the remainder of the query including the entire password check
- The database finds the administrator record and returns it without ever verifying the password
- The application grants full administrative access

---

## Remediation

### 1. Use Parameterized Queries (Primary Fix)

Never concatenate user input directly into SQL queries.

**Vulnerable code (Java):**
```java
String query = "SELECT * FROM users WHERE username='" + username + 
               "' AND password='" + password + "'";
```

**Secure code (Java Prepared Statement):**
```java
PreparedStatement stmt = conn.prepareStatement(
    "SELECT * FROM users WHERE username = ? AND password = ?");
stmt.setString(1, username);
stmt.setString(2, password);
ResultSet rs = stmt.executeQuery();
```

### 2. Input Validation and Sanitization
- Validate all user input server-side - never rely on client-side validation alone
- Reject or strip special characters such as `'`, `--`, `;` from authentication fields
- Implement an allowlist of acceptable input formats

### 3. Least Privilege Database Accounts
- Application database accounts should only have SELECT, INSERT, UPDATE permissions on required tables
- Remove DROP, DELETE, and administrative permissions from application-level database users
- Use separate database accounts for read and write operations where possible

### 4. Additional Defensive Layers
- Deploy a Web Application Firewall (WAF) to detect and block SQL injection patterns
- Implement proper error handling - never expose raw database error messages to users
- Enable logging and alerting on repeated failed authentication attempts
- Conduct regular secure code reviews and penetration testing as part of the SDLC

---

## References

- [OWASP Top 10 - A03:2021 Injection](https://owasp.org/Top10/A03_2021-Injection/)
- [OWASP SQL Injection Prevention Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/SQL_Injection_Prevention_Cheat_Sheet.html)
- CWE-89: Improper Neutralization of Special Elements in SQL Commands
- NIST SP 800-53 SI-10: Information Input Validation
- PortSwigger Web Security Academy - SQL Injection Labs

---

*This writeup was produced as part of authorized security lab practice. All testing was performed in a controlled environment.*
