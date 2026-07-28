# Lab 01: Auth Log Analysis — SSH Brute Force Detection

*(Linux equivalent of Windows Event Viewer / Event ID 4624 & 4625 analysis)*

## Objective

Detect and analyze SSH brute-force attempts using Linux authentication logs (`/var/log/auth.log`). This lab demonstrates the Linux-native equivalent of monitoring Windows Event ID 4625 (failed logon) and 4624 (successful logon)

## Environment

| Role | Machine | Details |
|------|---------|---------|
| Attacker | Kali Linux | Hydra used for brute-force simulation |
| Target | Ubuntu Server | SSH service exposed, auth.log monitored |
| Network | Vm-Ware internal/host-only network | Isolated lab environment |

## Steps Taken

1. **SSH server setup / confirmation service was running**
2. **I  created a mini wordlist with weak passwords and potential passwords for the bruteforce attack using - nano wordlist.txt on kali**
3. **For the bruteforce attempt on hydra through my kali OS - hydra -l hrstaff -P wordlist.txt ssh:// 192.168.***.***.]**
4. **To investigate all brute force attempt on my Target machine (ubuntu OS) : sudo tail -100 /var/log/auth.log**
5. **To filtre for failed attempts - sudo grep -a "Failed password" /var/log/auth.log**
6. **To filtre for successful login - sudo grep -a "Accepted password" /var/log/auth.log**
7. **To identify the source ip perpetrating the attack - sudo grep -a "Failed password" /var/log/auth.log | awk '{ print $ (NF -3) }' | sort |uniq -c**

## Key Log Entries / Commands

```bash
nano wordlist.txt
hydra -l hrstaff -P wordlist.txt ssh:// 192.168.***.***
sudo tail -100 /var/log/auth.log
grep -a "Failed password" /var/log/auth.log
grep -a "Accepted password" /var/log/auth.log
sudo grep -a "Failed password" /var/log/auth.log | awk '{ print $ (NF -3) }' | sort |uniq -c**
```

## Screenshots

*(Add screenshots to `./screenshots/` and reference them here)*

- `screenshots/hydra-attack-running.png` — Hydra brute-force in progress
- `screenshots/auth-log-failed-attempts.png` — Failed login entries in auth.log
- `screenshots/auth-log-success.png` — Successful login (if applicable)

## Findings / IOCs

| Indicator | Value |
|-----------|-------|
| Source IP | `192.168.***.***` |
| Target service | SSH (port 22) |
| Attempt count | `28, 2 successful` |
| Time window | `2026-07-23 18:29:17` - `2026-07-23 -18:29:27`|
| Outcome |  `attack detected` |

## Windows ↔ Linux Equivalent Mapping

| Windows Event Viewer | Linux auth.log Equivalent |
|------------------------|------------------------------|
| Event ID 4625 (Failed Logon) | `Failed password for hrstaff from 192.168.***.***` |
| Event ID 4624 (Successful Logon) | `Accepted password for hrstaff from 192.68.***.***` |
| Security Event Log | `/var/log/auth.log` (Debian/Ubuntu) |

## Lessons Learned

- **All unused SSH server must be terminated**
- ** Strong password policy is mandatory for every user** 
- **[Soc analyst must actively monitor every running services & port,for early detection of breaches**

## Next Step

- Automate detection with a script that alerts after 3 failed attempts from the same IP
- Feed these logs into a SIEM (Wazuh) for centralized alerting
