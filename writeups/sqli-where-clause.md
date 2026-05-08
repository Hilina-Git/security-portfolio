# SQL Injection - Hidden Data Retrieval via WHERE Clause Manipulation

**Finding ID:** VULN-2026-001  
**Severity:** Medium  
**Date:** May 7, 2026  
**Tester:** Hilina Gebre  
**Tools Used:** Browser (no special tools required)  
**Status:** Confirmed / Resolved (Lab Environment)

---

## Vulnerability Description

SQL Injection is a critical web application vulnerability where an attacker manipulates SQL queries by injecting malicious input into user-controlled fields. When an application fails to sanitize input, the injected code is executed directly by the database engine.

In this finding, the application's product filter passes the `category` parameter directly into a SQL query without sanitization. This allows an attacker to manipulate the WHERE clause and retrieve hidden or unreleased data that should not be publicly accessible.

---

## Impact

- Unauthorized access to hidden or unreleased product records
- Potential exposure of sensitive database contents
- Bypass of application-level access controls
- No authentication or special tools required to exploit

---

## Steps to Reproduce

1. Navigate to the application and click any product category (e.g. Accessories)
2. Observe the URL changes to:
```
/filter?category=Accessories
```
3. Modify the URL directly in the browser address bar to:
```
/filter?category='+OR+1=1--
```
4. Press Enter
5. Observe that all products including hidden and unreleased items are now displayed

---

## What Happened Behind the Scenes

The application constructs the following SQL query using unsanitized input:

```sql
SELECT * FROM products WHERE category = 'Accessories' AND released = 1
```

The `released = 1` condition normally hides unreleased products from users.

After injecting the payload, the query becomes:

```sql
SELECT * FROM products WHERE category = '' OR 1=1--' AND released = 1
```

- `'` closes the category string
- `OR 1=1` makes the condition always true, returning ALL records
- `--` comments out the rest of the query including the `released = 1` filter

The database returns all products regardless of their release status.

---

## Remediation

### 1. Use Parameterized Queries (Primary Fix)

Never concatenate user input directly into SQL queries.

**Vulnerable code (Java):**
```java
String query = "SELECT * FROM products WHERE category = '" + category + "' AND released = 1";
```

**Secure code (Java Prepared Statement):**
```java
PreparedStatement stmt = conn.prepareStatement(
    "SELECT * FROM products WHERE category = ? AND released = 1");
stmt.setString(1, category);
ResultSet rs = stmt.executeQuery();
```

### 2. Input Validation
- Validate and sanitize all user-supplied input server-side
- Reject or strip special characters such as `'`, `--`, `;` from query parameters
- Use an allowlist of acceptable category values where possible

### 3. Least Privilege Database Accounts
- Application database accounts should only have permissions required for normal operation
- Remove administrative and destructive permissions from application-level users

### 4. Additional Defensive Layers
- Deploy a Web Application Firewall (WAF) to detect and block SQL injection patterns
- Implement proper error handling - never expose raw database errors to users
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
