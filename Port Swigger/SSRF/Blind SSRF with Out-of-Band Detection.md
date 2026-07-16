# CTF Write-Up: SSRF Lab 4 – Blind SSRF with Out-of-Band Detection

## Challenge Overview

* Platform: PortSwigger Web Security Academy
* Category: Server-Side Request Forgery (SSRF)
* Difficulty: Practitioner
* Objective: Confirm a Blind SSRF vulnerability using Burp Collaborator.

---

## Initial Analysis

After accessing the lab, Burp Suite was configured to intercept traffic. Unlike previous SSRF labs, the stock API parameter was not directly exposed in the request.

Inspecting the request revealed a **Referer** header that was reflected back to the server.

Since no visible response was expected, this indicated that the vulnerability might be a **Blind SSRF**, requiring out-of-band detection.

---

## Exploitation

A Burp Collaborator payload was generated and initially placed inside the `Referer` header.

Example:

```text
58b8h7v794ul3oe669m9vio5iwoncd02.oastify.com
```

The request returned **HTTP 200 OK**, but no DNS or HTTP interactions appeared in Burp Collaborator.

After reviewing the payload, it was updated to include the required protocol:

```text
http://58b8h7v794ul3oe669m9vio5iwoncd02.oastify.com
```

The modified request was sent again.

This time, Burp Collaborator recorded outbound interactions from the vulnerable server, confirming that the application performed a server-side request to the supplied URL.

The interaction successfully completed the lab.

---

## Key Takeaways

* Blind SSRF vulnerabilities often do not produce visible responses.
* Burp Collaborator is an effective tool for detecting out-of-band interactions.
* Always include the correct protocol (`http://` or `https://`) when testing SSRF payloads.

---

## Tools Used

* Burp Suite
* Burp Repeater
* Burp Collaborator
