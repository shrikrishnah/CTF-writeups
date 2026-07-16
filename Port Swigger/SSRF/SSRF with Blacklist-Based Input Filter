# CTF Write-Up: SSRF Lab 3 – SSRF with Blacklist-Based Input Filter

## Challenge Overview

* Platform: PortSwigger Web Security Academy
* Category: Server-Side Request Forgery (SSRF)
* Difficulty: Practitioner
* Objective: Bypass blacklist-based input validation to access the internal administrator interface and delete the user `carlos`.

---

## Initial Analysis

After identifying the vulnerable stock checking functionality, the intercepted request was sent to **Repeater**.

A direct request to the localhost address was attempted:

```text
http://localhost/
```

The application returned **400 Bad Request**.

The same response occurred when using:

```text
http://127.0.0.1/
```

This indicated that the application was filtering requests to localhost using a blacklist.

---

## Bypassing the Blacklist

Different IP representations were tested.

Using the shortened loopback notation:

```text
http://127.1/
```

successfully bypassed the filter and returned **HTTP 200 OK**.

This confirmed that the blacklist only blocked common localhost representations.

---

## Accessing the Admin Panel

Attempting to access:

```text
http://127.1/admin
```

was still blocked because the string **admin** was also blacklisted.

To bypass this restriction, URL encoding was attempted. A single URL encoding did not work, so **double URL encoding** was used.

The payload became:

```text
http://127.1/%2561dmin
```

where `%2561` is the double URL-encoded representation of the character **a**.

This successfully bypassed the filter and returned the administrator interface.

---

## Exploitation

The administrator page contained the endpoint used to delete the target user.

The final payload used was:

```text
http://127.1/%2561dmin/delete?username=carlos
```

Sending the request through Burp Repeater successfully deleted the user **carlos**, completing the lab.

---

## Key Takeaways

* Blacklist-based filtering is ineffective against alternate IP representations.
* Loopback addresses have multiple valid formats, such as `127.1`.
* Double URL encoding can bypass poorly implemented input validation.
* Whitelisting is significantly more secure than relying on blacklist filtering.

---

## Tools Used

* Burp Suite
* Repeater
* URL Encoding
* Double URL Encoding
