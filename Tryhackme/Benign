# CTF Write-Up: Benign

## Challenge Overview

* Platform: TryHackMe
* Challenge: Benign
* Category: SOC Investigation / Threat Hunting
* Objective: Analyze Windows process creation logs in Splunk to identify the compromised user, trace the attack, and recover the malicious artifact and flag.

---

## Scenario

An IDS alert indicated suspicious process execution on one of the hosts belonging to the HR department. Due to limited resources, only Windows Process Creation logs (Event ID **4688**) were collected and ingested into the `win_eventlogs` index in Splunk.

The objective was to investigate the logs, identify the compromised user, determine the attack chain, and recover the flag.

---

## Question 1: How many logs were ingested for March 2022?

The following search was executed to view the available logs:

```spl
index="win_eventlogs"
```

The search returned:

```text
13,959
```

---

## Question 2: Identify the imposter account

To identify unusual usernames, the usernames were visualized and compared against the known employee list.

One suspicious account was observed:

```text
Amel1a
```

The account closely resembled the legitimate user **Amelia**, indicating a possible impersonation attempt.

---

## Question 3: Which HR user executed scheduled tasks?

To identify scheduled task creation, the following search was performed:

```spl
index="win_eventlogs" *schtasks*
```

The logs showed that the following HR user executed scheduled tasks:

```text
Chris.fort
```

---

## Question 4: Which HR user used a LOLBIN to download a payload?

Living-off-the-Land Binaries (LOLBins) such as `certutil.exe` are commonly abused to download malware.

The following search was used:

```spl
index="win_eventlogs" (haroon OR chris OR diana) ProcessName="C:\\Windows\\System32\\certutil.exe"
```

The logs revealed that the user responsible was:

```text
Haroon
```

---

## Question 5: Which LOLBIN was used?

The process responsible for downloading the payload was:

```text
certutil.exe
```

`certutil.exe` is a legitimate Windows utility that attackers frequently abuse to download malicious files while avoiding detection.

---

## Question 6: When was the binary executed?

Reviewing the process execution timestamp showed the binary was executed on:

```text
2022-03-04
```

---

## Question 7: Which third-party website hosted the payload?

Examining the command-line arguments and URL within the logs revealed that the payload was downloaded from:

```text
controlc.com
```

Although ControlC is a legitimate text-sharing service, it can also be abused by attackers to host malicious scripts or payloads.

---

## Question 8: What file was downloaded?

The downloaded payload was saved on the victim machine as:

```text
benign.exe
```

---

## Question 9: Recover the flag

The URL discovered in the previous step was visited to inspect the hosted content.

The malicious content contained the following flag:

```text
THM{KJ&*H^B0}
```

---

## Question 10: What URL did the infected host connect to?

The complete URL identified during the investigation was:

```text
https://controlc.com/e4d11035
```

---

## Attack Summary

1. Windows Event ID 4688 logs were analyzed in Splunk.
2. An impersonated user account (`Amel1a`) was identified.
3. Scheduled task activity pointed to suspicious execution by an HR user.
4. The attacker abused the legitimate Windows binary `certutil.exe` (a LOLBIN) to download a malicious payload.
5. The payload was hosted on `controlc.com`.
6. The downloaded file (`benign.exe`) contained the malicious content and the challenge flag.

---

## Key Takeaways

* Windows Event ID 4688 is valuable for investigating process execution.
* LOLBins such as `certutil.exe` are commonly abused to evade detection.
* Monitoring command-line arguments helps identify malicious downloads.
* Scheduled tasks are frequently used by attackers to establish persistence.
* Splunk is an effective tool for correlating Windows logs and reconstructing attack timelines.

---

## Tools Used

* Splunk
* Windows Event Logs (Event ID 4688)
* Threat Hunting Techniques
* Process and Command-Line Analysis
