# 🍳 Overheard at Breakfast — CTF Writeup

> **Platform:** TryHackMe &nbsp;|&nbsp; **Category:** OSINT &nbsp;|&nbsp; **Difficulty:** Easy
>
> **Flag:** `THM{S3creT_Pr0fil3_H4s_b33n_Ident1fi3d}`

---

## Overview

Two strangers — Ponzi and Lambo — share a casual conversation on a resort chat platform. Lambo drops their email address and a vague reference to a social media tool "starting with G" they once used to link their accounts. That's all it takes.

The goal: pivot from those two breadcrumbs to a hidden online profile and extract the flag buried inside it.

---

## Key Concepts

| Concept | Description |
|---|---|
| **Gravatar** | A global profile service that identifies users by the MD5 hash of their email — making profiles discoverable from an email alone, no username needed |
| **MD5 as Identifier** | Gravatar derives profile URLs from `MD5(lowercase_email)` — no brute force required if you have the email |
| **OSINT via Email** | An email address is an identity anchor that links to Gravatar, GitHub, HaveIBeenPwned, and other platforms |
| **Base64 Obfuscation** | Encoding ≠ encryption. Base64 is trivially reversible and offers zero confidentiality |
| **Passive Reconnaissance** | No scanning or exploitation — pure intel gathering from public sources |

---

## Attack Chain

```
conversation.png  (task file)
      │
      ├──▶ email leaked: lambobytelotushotel@gmail.com
      └──▶ hint: "social media tool starting with G"
                        │
                        ▼
                   Gravatar (gravatar.com)
                        │
                        ▼
           MD5(email) = d4a5fc5d3128890778667e24617d7cc0
                        │
                        ▼
           gravatar.com/<hash>  →  Lambo's profile
                        │
                        ▼
           Base64 string in profile bio
                        │
                        ▼
           CyberChef decode  →  THM{S3creT_Pr0fil3_H4s_b33n_Ident1fi3d}
```

---

## Step-by-Step Solution

### Step 1 — Read the Conversation

Extract the task zip and open `conversation.png`. Two pieces of intel are visible:

- **Email:** `lambobytelotushotel@gmail.com`
- **Hint:** Lambo mentions a profile tool "started with a G" used to link all social accounts — and says the email is their best way to be contacted

> **Pivot point:** Email = identity anchor. "Starts with G" + "link social accounts" + "profile" = Gravatar. The service uses `MD5(email)` as a public URL key, meaning the profile is discoverable from the email alone.
<img width="1175" height="781" alt="conversation" src="https://github.com/user-attachments/assets/41fbbe8a-3658-4d52-9035-5f746d8379ef" />


---

### Step 2 — Generate the MD5 Hash

Gravatar profile URLs are derived from the MD5 hash of the lowercase, trimmed email. Run:

```bash
echo -n "lambobytelotushotel@gmail.com" | md5sum
# d4a5fc5d3128890778667e24617d7cc0
```
<img width="611" height="61" alt="image" src="https://github.com/user-attachments/assets/cb0ca8cc-12cb-45e3-b69c-0a59b00ea313" />

> **Why `-n`?** It strips the trailing newline from `echo`. Without it, the newline is included in the hash — the output changes and the profile won't resolve.

Navigate to:

```
https://gravatar.com/d4a5fc5d3128890778667e24617d7cc0
```
<img width="1310" height="528" alt="image" src="https://github.com/user-attachments/assets/1cc1ab8f-eed1-4cc9-a4d3-99c215f51b6e" />

---

### Step 3 — Find the Gravatar Profile

The profile resolves to **Lambo** (Lam-boh · Byte Lotus Hotel). The bio reads:

> *"Funny thing about email hashes, they follow you places you didn't expect. Glad you found the right corner of the internet! Here is your prize:*
> `VEhNe1MzY3JlVF9QcjBmaWwzX0g0c19iMzNuX0lkZW50MWZpM2R9`"

> Even after "wiping" social accounts, MD5-linked profiles persist and remain discoverable from the email alone.

---

### Step 4 — Decode Base64 in CyberChef

Paste the string into [CyberChef](https://gchq.github.io/CyberChef/) with the **From Base64** operation:

```
Input:  VEhNe1MzY3JlVF9QcjBmaWwzX0g0c19iMzNuX0lkZW50MWZpM2R9
Output: THM{S3creT_Pr0fil3_H4s_b33n_Ident1fi3d}
```
<img width="692" height="421" alt="image" src="https://github.com/user-attachments/assets/601f4f1a-a1fb-4cb8-a24b-0dbb7f48690d" />

---

## Tools Used

| Tool | Purpose |
|---|---|
| **md5sum** (Linux CLI) | Hash the email to derive the Gravatar profile ID |
| **Gravatar** | Public profile lookup via `MD5(email)` |
| **CyberChef** | Decode the Base64-encoded flag from the profile bio |

---

## Vulnerabilities

**Email-to-Profile Linkage via MD5**

Gravatar uses `MD5(email)` as a permanent, publicly accessible identifier. Anyone with your email can find your Gravatar profile without ever knowing the URL. Profiles you "deleted" may still be cached or indexed, and any sensitive info stored in the bio is fully discoverable from the email alone.

**Base64 Is Not Encryption**

Base64 is an encoding scheme, not a security mechanism — it's reversible without any key. Storing secrets in Base64 and treating them as "hidden" is a common mistake that any attacker will undo in seconds.

---

## Key Takeaways

- **An email address is an identity anchor.** It ties to Gravatar, GitHub, HaveIBeenPwned, and more — treat it as sensitive.
- **Deleting a profile doesn't erase discoverability.** MD5-linked profiles persist in caches long after removal.
- **MD5 hashes of emails are not private.** If an adversary has your email, they already have your Gravatar hash.
- **Base64 ≠ security.** Encoding and encrypting are completely different operations.
- **OSINT chains are short.** Email → hash → profile → flag: four steps, no exploitation needed.

---

## Further Reading

- [Gravatar API Docs — Hash Format](https://docs.gravatar.com/general/hash/)
- [OSINT Framework](https://osintframework.com/)
- [CyberChef](https://gchq.github.io/CyberChef/)
- [HaveIBeenPwned](https://haveibeenpwned.com/)

---

*Writeup by [@shrikrishnah](https://github.com/shrikrishnah)*
