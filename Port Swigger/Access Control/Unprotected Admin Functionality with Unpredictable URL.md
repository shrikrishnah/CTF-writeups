# CTF Write-Up: Access Control Lab 2 – Unprotected Admin Functionality with Unpredictable URL

## Challenge Overview

* Platform: PortSwigger Web Security Academy
* Category: Access Control
* Difficulty: Apprentice
* Objective: Discover a hidden administrator URL and delete the user `carlos`.

---

## Initial Analysis

The application did not expose any obvious administrator endpoints.

Developer Tools were opened to inspect the website's JavaScript files.

---

## Enumeration

While reviewing `index.js`, an administrator path was discovered.

The endpoint changed each time the lab was restarted, making it unpredictable.

After refreshing and checking the JavaScript again, the correct administrator endpoint was identified.

Example:

```text
/admin-n7fleo
```

---

## Exploitation

Accessing the administrator interface provided the option to delete users.

The following request was executed:

```text
/admin-n7fleo/delete?username=carlos
```

The user was successfully deleted, completing the lab.

---

## Key Takeaways

* Hidden URLs are not a replacement for proper authorization.
* JavaScript files often reveal sensitive application routes.
* Administrative functionality must always be protected by server-side access controls.

---

## Tools Used

* Browser Developer Tools
* Burp Suite
