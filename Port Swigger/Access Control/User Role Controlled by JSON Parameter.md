# CTF Write-Up: Access Control Lab 4 – User Role Controlled by JSON Parameter

## Challenge Overview

* Platform: PortSwigger Web Security Academy
* Category: Access Control
* Difficulty: Apprentice
* Objective: Escalate privileges by modifying a JSON request.

---

## Initial Analysis

Several request parameters were modified without success, including URL parameters and headers.

Attention then shifted to the profile update functionality.

Updating the email address generated a JSON request that included user information.

---

## Discovery

Inspecting the intercepted request revealed JSON fields similar to:

```json
{
  "email": "user@example.com",
  "roleid": 1
}
```

The presence of the `roleid` field suggested that user roles were being controlled through client-supplied JSON data.

---

## Exploitation

The intercepted request was modified by changing the role identifier:

```json
{
  "email": "user@example.com",
  "roleid": 2
}
```

The modified request was accepted, granting administrator privileges.

Once elevated, the administrator endpoint was accessed and the following request was executed:

```text
/ admin/delete?username=carlos
```

The target user was successfully deleted, completing the lab.

---

## Key Takeaways

* Client-supplied JSON parameters must never determine authorization levels.
* Role validation should always be enforced on the server.
* Sensitive fields should not be trusted simply because they are hidden from the user interface.

---

## Tools Used

* Burp Suite
* Burp Repeater
