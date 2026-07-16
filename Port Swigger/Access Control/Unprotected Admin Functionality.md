# CTF Write-Up: Access Control Lab 1 – Unprotected Admin Functionality

## Challenge Overview

* Platform: PortSwigger Web Security Academy
* Category: Access Control
* Difficulty: Apprentice
* Objective: Discover an exposed administrator panel and delete the user `carlos`.

---

## Initial Analysis

After browsing the application, requests were intercepted using Burp Suite and sent to Repeater.

Directly requesting `/admin` returned an error, suggesting that the administrator interface was located elsewhere.

---

## Enumeration

The file:

```text
/robots.txt
```

was accessed and contained the following disallowed path:

```text
/administrator-panel
```

Although hidden from search engines, the endpoint remained publicly accessible.

---

## Exploitation

Visiting:

```text
/administrator-panel
```

revealed the administrator interface.

The page contained functionality to delete users.

The following request was used:

```text
/administrator-panel/delete?username=carlos
```

Executing the request deleted the target user and solved the lab.

---

## Key Takeaways

* Sensitive endpoints should never rely on obscurity for protection.
* `robots.txt` should not expose administrative paths.
* Proper authentication and authorization must be enforced on all administrative interfaces.

---

## Tools Used

* Burp Suite
* Burp Repeater
