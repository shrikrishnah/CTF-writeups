# CTF Write-Up: Access Control Lab 3 – User Role Controlled by Request Parameter

## Challenge Overview

* Platform: PortSwigger Web Security Academy
* Category: Access Control
* Difficulty: Apprentice
* Objective: Escalate privileges by manipulating a user-controlled parameter.

---

## Initial Analysis

After logging into the application, multiple request parameters were tested without success.

Reviewing the intercepted login request revealed a parameter controlling administrative privileges:

```text
Admin=False
```

This suggested that the application trusted a client-supplied value to determine user permissions.

---

## Exploitation

The login request was intercepted using Burp Suite.

The parameter was modified from:

```text
Admin=False
```

to

```text
Admin=True
```

The modified request granted administrative privileges.

To complete the challenge, the request was changed to:

```text
GET /admin/delete?username=carlos
```

After following the HTTP 302 redirect, the user **carlos** was deleted successfully.

---

## Key Takeaways

* Authorization decisions should never rely on client-controlled parameters.
* Privilege information must always be verified on the server.
* Hidden request parameters are not a secure access control mechanism.

---

## Tools Used

* Burp Suite
* Burp Repeater
