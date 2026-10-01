# Password Cracking Lab

This repository documents a series of hands-on exercises demonstrating password cracking techniques against different hash types and services. The lab utilizes standard cybersecurity tools like John the Ripper, Hashcat, and Hydra to perform dictionary and brute-force attacks.

Every exercise below is followed by a look at how to detect and respond to it from the defender's side.

All exercises were performed in a secure, sandboxed virtual environment.

---

## 1. Cracking a Linux Password Hash (John the Ripper)

### Objective
To extract a user's password hash from a Linux system and crack it using a dictionary attack with John the Ripper.

### Method
1.  A new user (`test`) was created on a Linux machine with a simple password (`aaa`).
2.  The user's password hash (a `sha512crypt` hash) was extracted from the `/etc/shadow` file.
3.  John the Ripper was run with the `rockyou.txt` wordlist against the extracted hash.
4.  The `--show` flag was used to display the cracked password, successfully revealing the original password "aaa".

### Evidence
A screenshot showing the hash extraction and the successful crack with John the Ripper is included in this repository.
*   *(See `linux-john-the-ripper-crack.png`)*

---

## 2. Cracking an MD5 Hash (Hashcat)

### Objective
To identify the hash type of an unknown hash and crack it using Hashcat.

### Method
1.  The provided hash (`fbb1b3d8ca94bac2fb046742c957b61c`) was first analyzed. Tools like `hashid` can identify this as a standard MD5 hash.
2.  Hashcat was used with mode `-m 0` (for MD5) and attack mode `-a 0` (dictionary attack) against the hash, using the `rockyou.txt` wordlist.
3.  The `--show` flag was used to display the cracked password from Hashcat's potfile.

### Evidence
A screenshot showing the Hashcat command and the successful result is included in this repository.
*   *(See `md5-hashcat-crack.png`)*

---

## 3. Cracking a Windows NTLM Hash (Online Tools)

### Objective
To extract an NTLM password hash from a Windows XP machine and crack it using an online hash-cracking service.

### Method
1.  A password was set for a user on a Windows XP virtual machine.
2.  `pwdump7` was used to dump the password hashes from the Windows SAM database. The NTLM hash for the user was identified.
3.  The NTLM hash was submitted to an online cracking service (like onlinehashcrack.com).
4.  Because NTLM is an unsalted hashing algorithm, the online service's pre-computed tables (rainbow tables) were able to quickly find the corresponding password.

### Evidence
A screenshot showing the dumped NTLM hash and the result from the online cracking tool is included in this repository.
*   *(See `windows-ntlm-online-crack.png`)*

---

## 4. Brute-Forcing a Live Service (Hydra)

### Objective
To perform a dictionary attack against a live FTP service to find valid login credentials.

### Method
1.  Two files were created: `userss.txt` (a list of potential usernames) and `passwords.txt` (a list of potential passwords).
2.  Hydra was used to target the FTP service on the Metasploitable machine (`ftp://192.168.174.131`).
3.  Hydra systematically tried every combination of username and password from the provided lists until it found a valid pair.

### Evidence
A screenshot showing the Hydra command, the brute-force process, and the discovered valid credentials (`service:service`) is included in this repository.
*   *(See `ftp-hydra-bruteforce.png`)*

---

## From the Defender's Side

Running these attacks myself made it clear what evidence each one leaves behind, and where the realistic chance to catch them is.

### Linux Password Hash Cracking (John the Ripper)

This attack requires the attacker to already have read access to `/etc/shadow`, so the real detection point is earlier than the cracking itself.

**What to look for:**
- File integrity monitoring (FIM) alerting on any read or access to `/etc/shadow` outside of expected system processes
- Auditd rules watching for access to shadow files, which is a standard hardening baseline on Linux systems
- Any new or modified user accounts around the same time, since extracting a hash is often paired with creating a backdoor account

**A rule to write:**
> Alert on any process other than the standard authentication stack reading `/etc/shadow`.

**How to respond:** Treat this as a sign the attacker already has local access or a foothold. The password crack itself happens offline and leaves no network trace, so the access that got them the file is what I'd be investigating, not the cracking.

### MD5 Hash Cracking (Hashcat)

Hashcat ran locally against an already-obtained hash, so there's no network signature here either. The defensive angle is upstream: how did an MD5 hash end up crackable at all?

**What to look for:**
- Any system or application still using MD5 or other weak hashing algorithms for passwords, which is a finding worth flagging regardless of whether an attack happened
- Password policy and hashing algorithm audits as part of regular vulnerability management

**How to respond:** This one's really a hardening recommendation rather than an incident: migrate anything still using MD5/SHA1 for password storage to a modern algorithm like bcrypt or Argon2, which are deliberately slow and resist exactly this kind of offline cracking.

### NTLM Hash Cracking (Online Rainbow Tables)

This is the step I'd be most worried about seeing in a real environment, because NTLM being unsalted means a dumped hash is often cracked in seconds.

**What to look for:**
- Any use of tools like `pwdump` or similar credential-dumping utilities on an endpoint, which EDR products flag by default
- Access to the SAM database outside normal system processes
- The resulting cracked password being reused somewhere, which I'd check via credential-reuse detection if available

**How to respond:** Treat SAM database access as a high-severity alert on its own, since it almost always means local admin access has already been achieved. I wouldn't wait to see if the hash gets cracked; I'd respond to the dump itself.

### Brute-Forcing a Live Service (Hydra)

This is the one exercise in this repo that's actually visible on the network as it happens, which makes it the most realistically detectable.

**What to look for:**
- A spike in failed login attempts to the FTP service from a single source IP in a short time window
- Multiple different usernames being tried in sequence from the same source, which distinguishes a credential-stuffing or brute-force attempt from a user mistyping their password
- The eventual successful login following a long run of failures from the same source

**A rule to write:**
> If a single source IP generates more than N failed FTP logins within T seconds, raise a "possible brute-force" alert; raise it to high severity if a successful login follows.

**How to respond:** Block or rate-limit the source IP, and treat the account that succeeded as compromised: force a password reset and review what that account accessed afterward.

### What to take from this

Three of these four attacks happen entirely offline after the real compromise has already occurred, so cracking the hash isn't usually where the defender gets their chance. Getting better at catching this class of attack means focusing on the access that produces the hash in the first place: file access to shadow/SAM databases, credential-dumping tool usage, and privilege escalation. Only the live brute-force attempt is something a SIEM can realistically catch in the moment.
