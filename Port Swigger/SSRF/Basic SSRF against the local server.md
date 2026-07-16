# CTF Write-Up: SSRF Lab 1 – Basic SSRF Against the Local Server

## Challenge Overview

* Platform: PortSwigger Web Security Academy
* Category: Server-Side Request Forgery (SSRF)
* Difficulty: Apprentice
* Objective: Exploit an SSRF vulnerability to access the application's local admin panel and delete the user `carlos`.

---

## Initial Analysis

After accessing the lab, I browsed through several products and used the **Check Stock** functionality. Since stock information is retrieved from another endpoint, this suggested that the server was making backend requests on behalf of the client, indicating a potential SSRF vulnerability.

Intercepting the request in Burp Suite revealed a parameter named `stockApi`, which contained the URL used by the server to retrieve stock information.

---

## Exploitation

The intercepted POST request was sent to **Repeater**.

The value of the `stockApi` parameter was replaced with:

```text
http://localhost/admin
```

Since the parameter was URL-encoded, the modified URL was encoded before sending the request.

The response successfully returned the internal administrator panel, confirming that the application could access resources hosted on the local server.

Next, the HTML response revealed the following endpoint:

```text
http://localhost/admin/delete?username=carlos
```

After URL encoding this endpoint and replacing the value of the `stockApi` parameter, the request was sent again through Repeater.

The server executed the request and deleted the user **carlos**, successfully completing the lab.

---

## Key Takeaways

* SSRF vulnerabilities allow attackers to force the server to make requests to unintended locations.
* Internal services such as `localhost` are often inaccessible externally but reachable through SSRF.
* Always validate and restrict user-controlled URLs before making server-side requests.

---

## Tools Used

* Burp Suite
* Repeater
* URL Encoding
