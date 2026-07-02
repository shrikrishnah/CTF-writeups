# CTF Write-Up: Boogeyman 2

## Challenge Overview

* Platform: TryHackMe
* Challenge: Boogeyman 2
* Category: Malware Analysis, Memory Forensics, Threat Hunting
* Objective: Investigate a phishing attack, reconstruct the malware execution chain, identify persistence mechanisms, and determine the attacker's infrastructure.

---

## Scenario

A Human Resources employee, Maxine Beck, received a malicious résumé disguised as a job application. After opening the document, suspicious processes were detected on the workstation, prompting a forensic investigation.

---

## Initial Email Analysis

The phishing email was sent from:

```
westaylor23@outlook.com
```

Victim:

```
maxine.beck@quicklogisticsorg.onmicrosoft.com
```

The malicious attachment was:

```
Resume_WesleyTaylor.doc
```

MD5 Hash:

```
52c4384a0b9e248b95804352ebec6c5b
```

---

## Stage 1 Infection

The document contained a malicious macro that downloaded the next stage of the malware from:

```
https://files.boogeymanisback.lol/aa2a9c53cbb80416d3b47d85538d9971/update.png
```

Although the payload used a `.png` extension, it actually delivered malicious code.

---

## Stage 2 Execution

The downloaded JavaScript payload was executed by:

```
wscript.exe
```

File location:

```
C:\ProgramData\update.js
```

Process Information:

* PID: **4260**
* Parent PID: **1124**

The script then downloaded and executed another payload:

```
https://files.boogeymanisback.lol/aa2a9c53cbb80416d3b47d85538d9971/update.exe
```

---

## Command and Control

The malicious executable was stored as:

```
C:\Windows\Tasks\updater.exe
```

The process responsible for establishing the Command and Control (C2) connection had:

* PID: **6216**

The malware communicated with:

```
128.199.95.189:8080
```

---

## Memory Analysis

Memory analysis revealed the original phishing attachment location:

```
C:\Users\maxine.beck\AppData\Local\Microsoft\Windows\INetCache\Content.Outlook\WQHGZCFI\Resume_WesleyTaylor (002).doc
```

This confirmed the initial infection vector.

---

## Persistence Mechanism

Immediately after establishing communication with the C2 server, the attacker created a scheduled task for persistence.

The command used was:

```text
schtasks /Create /F /SC DAILY /ST 09:00 /TN Updater /TR 'C:\Windows\System32\WindowsPowerShell\v1.0\powershell.exe -NonI -W hidden -c "IEX ([Text.Encoding]::UNICODE.GetString([Convert]::FromBase64String((gp HKCU:\Software\Microsoft\Windows\CurrentVersion debug).debug)))"'
```

This scheduled task ensured that the malware would execute automatically every day, allowing the attacker to maintain long-term access to the compromised system.

---

## Key Takeaways

* Microsoft Office macros remain a common initial access technique.
* Attackers frequently disguise malware using misleading file extensions.
* `wscript.exe` is commonly abused to execute malicious JavaScript payloads.
* Scheduled tasks are a popular persistence mechanism in Windows environments.
* Memory forensics can reveal the original infection source, payload locations, and attacker activity even after files have been removed.
