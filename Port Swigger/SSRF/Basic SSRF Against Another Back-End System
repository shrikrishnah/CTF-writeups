# CTF Write-Up: SSRF Lab 2 – Basic SSRF Against Another Back-End System

## Challenge Overview

* Platform: PortSwigger Web Security Academy
* Category: Server-Side Request Forgery (SSRF)
* Difficulty: Apprentice
* Objective: Discover an internal backend server, access its administrator interface, and delete the user `carlos`.

---

## Initial Analysis

After exploring the application, the **Check Stock** functionality was identified as a potential SSRF entry point.

The corresponding request was intercepted using Burp Suite and sent to **Intruder** for further testing.

---

## Exploitation

The stock API URL contained an internal IP address.

The final octet of the IP address was selected as the Intruder payload position:

```text
192.168.0.§X§
```

An attack was launched against the subnet to identify active hosts.

After the scan completed, responses were filtered by **HTTP Status Code 200**, revealing an active backend server at:

```text
http://192.168.0.16
```

The successful request was sent to **Repeater**.

Appending `/admin` returned the internal administrator interface:

```text
http://192.168.0.16/admin
```

The response contained the endpoint used to delete the target user.

The request URL was then modified to:

```text
http://192.168.0.16/admin/delete?username=carlos
```

Sending the request successfully deleted the user and completed the lab.

---

## Key Takeaways

* SSRF can be used to discover hidden internal infrastructure.
* Burp Intruder is effective for enumerating internal IP ranges.
* Backend administrative interfaces are often only accessible from trusted internal networks.

---

## Tools Used

* Burp Suite
* Intruder
* Repeater
