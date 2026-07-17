# CTF Write-Up: Access Control Lab 6 – Method-Based Access Control Can Be Circumvented

## Challenge Overview

* Platform: PortSwigger Web Security Academy
* Category: Access Control
* Difficulty: Practitioner
* Objective: Exploit flawed HTTP method-based authorization to promote the user **wiener** to an administrator.

---

## Initial Analysis

The administrator credentials provided by the lab were used to understand how user roles were managed.

After logging in as **administrator**, the user management panel contained functionality to change a user's role.

The request responsible for promoting a user was intercepted using Burp Suite and sent to **Repeater**.

---

## Initial Testing

Several parameters within the request were modified, including the target username and role values.

Attempts such as changing:

```text
username=carlos
```

to

```text
username=wiener
```

resulted in **403 Forbidden** or **Unauthorized** responses.

---

## Exploitation

The HTTP method used by the request was then modified.

Changing the request from **POST** to **GET** produced a different response, indicating that the application handled authorization differently depending on the request method.

Next, the administrator account was logged out and the normal user account (**wiener:peter**) was used to log in.

The session cookie belonging to **wiener** was copied into the previously captured administrator request.

Finally, the target username was changed:

```text
username=wiener
```

The modified request was sent using the vulnerable HTTP method.

The application incorrectly processed the request and promoted **wiener** to an administrator, successfully completing the lab.

---

## Key Takeaways

* Authorization should never depend on the HTTP method used.
* Every endpoint should enforce identical authorization checks regardless of whether the request is sent using GET, POST, PUT, or DELETE.
* Session validation and privilege checks must always be performed on the server before executing sensitive actions.

---

## Concept

### Method-Based Access Control

Some applications implement security rules that only apply to specific HTTP methods. For example, a developer may secure a **POST** request while forgetting to apply the same authorization checks to a **GET** request that performs the same action.

Attackers can exploit these inconsistencies by changing the HTTP method until they find one that bypasses the intended security controls.

A secure application should treat authorization independently of the request method and verify that the authenticated user has permission to perform the requested action before processing it.

---

## Tools Used

* Burp Suite
* Burp Repeater
