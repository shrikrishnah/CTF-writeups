# CTF Write-Up: Access Control Lab 5 – URL-Based Access Control Can Be Circumvented

## Challenge Overview

* Platform: PortSwigger Web Security Academy
* Category: Access Control
* Difficulty: Practitioner
* Objective: Bypass URL-based access controls to access the administrator panel and delete the user `carlos`.

---

## Initial Analysis

After logging into the application as a normal user, attempts to directly access the `/admin` endpoint resulted in an **Unauthorized** response.

The intercepted request was sent to **Burp Repeater** for further testing.

---

## Exploitation

A custom HTTP header was added to the request:

```http
X-Original-URL: /admin
```

Initially, the request failed because only the header was modified while the request path still pointed to another endpoint.

The request was then changed to:

```http
GET / HTTP/2
Host: <target>

X-Original-URL: /admin
```

The application processed the value supplied in the `X-Original-URL` header instead of the original request path and returned the administrator panel.

The response contained the endpoint used to delete users.

The request was modified again:

```http
GET /delete?username=carlos HTTP/2
Host: <target>

X-Original-URL: /admin
```

After following the HTTP redirect, the request was processed successfully and the user **carlos** was deleted.

---

## Key Takeaways

* Some reverse proxies or web servers trust headers such as `X-Original-URL` or `X-Rewrite-URL` to determine the requested resource.
* If backend applications rely on these headers without proper validation, attackers may bypass URL-based access restrictions.
* Authorization checks must always be performed on the final resource being accessed rather than trusting client-supplied headers.

---

## Concept

### URL-Based Access Control

Some web applications sit behind reverse proxies that rewrite incoming URLs before forwarding them to the backend server. Headers like **X-Original-URL** and **X-Rewrite-URL** are intended for internal use to preserve the original request path.

If the backend trusts these headers without validating the user's permissions, an attacker can manipulate them to access protected endpoints such as `/admin`, effectively bypassing access controls.

The correct implementation is to ignore user-supplied rewrite headers or validate authorization after URL rewriting has occurred.

---

## Tools Used

* Burp Suite
* Burp Repeater
