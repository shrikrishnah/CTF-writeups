# CTF Write-Up: Boogeyman 1

## Challenge Overview

* Platform: TryHackMe
* Challenge: Boogeyman 1
* Category: Phishing Analysis, Malware Analysis, Network Forensics
* Objective: Investigate a phishing attack, identify the malware's behavior, trace the attack chain, and recover the compromised data.

---

## Scenario

Quick Logistics LLC reported a phishing campaign targeting employees in the Finance department. An employee, Julianne, opened a malicious invoice attachment that compromised her workstation. The objective was to analyze the phishing email, determine the attacker's infrastructure, investigate the malware, and identify the data exfiltrated from the victim.

---

## Initial Email Analysis

The phishing email originated from:

* **Sender:** `agriffin@bpakcaging.xyz`
* **Victim:** `julianne.westcott@hotmail.com`

Inspection of the email headers revealed that the attacker used **Elastic Email** as the third-party mail relay service.

The attachment was a password-protected archive containing:

```
Invoice_20230103.lnk
```

Password:

```
Invoice2023!
```

---

## Malicious Shortcut Analysis

Using **lnkparse**, the malicious shortcut was analyzed.

The command-line arguments contained a Base64-encoded PowerShell payload, which downloaded additional malware from the attacker's infrastructure.

Further analysis identified the attacker-controlled domains:

* `cdn.bpakcaging.xyz`
* `files.bpakcaging.xyz`

---

## Host Enumeration

The downloaded malware executed an enumeration utility:

```
Seatbelt
```

The attacker later accessed the following database:

```
C:\Users\j.westcott\AppData\Local\Packages\Microsoft.MicrosoftStickyNotes_8wekyb3d8bbwe\LocalState\plum.sqlite
```

This database belongs to **Microsoft Sticky Notes**, indicating that the attacker searched for locally stored sensitive information.

---

## Data Exfiltration

The attacker collected a KeePass password database:

```
protected_data.kdbx
```

The `.kdbx` extension confirmed it was a **KeePass** password vault.

Instead of using traditional HTTP exfiltration, the attacker encoded the data in **hexadecimal** and exfiltrated it using **DNS** requests through the `nslookup` utility.

---

## Command and Control

The investigation revealed:

* File server hosted using **Python**
* Command output sent using the **HTTP POST** method
* Data exfiltration performed over the **DNS** protocol

Recovered information included:

* KeePass password:

  ```
  %p9^3!lL^Mz47E2GaT^y
  ```

* Stored credit card number:

  ```
  4024007128269551
  ```

---

## Key Takeaways

* Password-protected attachments are frequently used to bypass email security filters.
* Windows shortcut (`.lnk`) files are commonly abused to launch PowerShell payloads.
* Legitimate tools such as **Seatbelt** and **nslookup** are often leveraged during post-exploitation.
* DNS tunneling remains an effective method for covert data exfiltration.
* KeePass databases are valuable targets because they may contain organizational credentials.
