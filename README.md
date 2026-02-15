# Password Cracking Lab

This repository documents a series of hands-on exercises demonstrating password cracking techniques against different hash types and services. The lab utilizes standard cybersecurity tools like John the Ripper, Hashcat, and Hydra to perform dictionary and brute-force attacks.

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
