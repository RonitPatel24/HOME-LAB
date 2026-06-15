# Cybersecurity Home Lab

A hands-on network attack and defense lab using virtualization.

## Overview

This project demonstrates offensive and defensive cybersecurity techniques in an isolated virtual environment using Kali Linux (attacker) and Ubuntu (defender).

| Machine | Role | IP Address |
|---------|------|------------|
| Kali Linux | Attacker | 192.168.115.128 |
| Ubuntu | Defender | 192.168.115.129 |

## Tools Used

| Tool | Purpose |
|------|---------|
| **VMware** | Virtualization platform |
| **Nmap** | Network scanning and enumeration |
| **Wireshark** | Packet capture and analysis |
| **Hydra** | SSH brute force attack |
| **UFW** | Firewall configuration |
| **Fail2Ban** | Automatic intrusion prevention |
| **SSH** | Remote access testing |

## Lab Architecture

```
┌────────────────────────────────────────────────────┐
│              VMware Virtual Network                │
│                192.168.115.0/24                    │
│                                                    │
│  ┌──────────────┐          ┌──────────────┐       │
│  │ KALI LINUX   │          │   UBUNTU     │       │
│  │  (Attacker)  │◄────────►│  (Defender)  │       │
│  │              │          │              │       │
│  │ 192.168.115  │          │ 192.168.115  │       │
│  │    .128      │          │    .129      │       │
│  │              │          │              │       │
│  │ • Nmap       │          │ • SSH (22)   │       │
│  │ • Hydra      │          │ • Apache (80)│       │
│  │ • Wireshark  │          │ • UFW        │       │
│  │ • SSH client │          │ • Fail2Ban   │       │
│  └──────────────┘          └──────────────┘       │
│                                                    │
└────────────────────────────────────────────────────┘
```

## Exercises Completed

### Exercise 1: Network Scanning

Discovered open services using Nmap.

**Basic scan:**
```bash
nmap 192.168.115.129
```
```
PORT   STATE SERVICE
22/tcp open  ssh
80/tcp open  http
```

**Aggressive scan:**
```bash
nmap -A 192.168.115.129
```
```
22/tcp open  ssh     OpenSSH 9.6p1 Ubuntu
80/tcp open  http    Apache httpd 2.4.58 ((Ubuntu))
Running: Linux 5.X
```

**Wireshark Analysis:**
- Captured TCP SYN packets from attacker
- Observed SYN-ACK responses on open ports (22, 80)
- Observed RST responses on closed ports

---

### Exercise 2: Firewall Defense

Blocked attacker using UFW firewall.

```bash
sudo ufw reset
sudo ufw enable
sudo ufw default deny incoming
sudo ufw default allow outgoing
sudo ufw deny from 192.168.115.128
sudo ufw allow 22
sudo ufw allow 80
```

**Result:**
- Before: Ports 22 and 80 visible to attacker
- After: All 1000 ports show as "filtered" — attacker sees nothing

```
All 1000 scanned ports are in ignored states.
Not shown: 1000 filtered tcp ports (no-response)
```

---

### Exercise 3: Brute Force Attack & Defense

**Attack (from Kali):**
```bash
hydra -l ronit -P passwords.txt 192.168.115.129 ssh
```

**Detection (on Ubuntu):**
```bash
sudo cat /var/log/auth.log | grep -i failed
```
```
Failed password for ronit from 192.168.115.128 port 45748 ssh2
Failed password for ronit from 192.168.115.128 port 45750 ssh2
...
```

**Defense:**
```bash
sudo apt install fail2ban -y
sudo systemctl enable fail2ban
sudo systemctl start fail2ban
```

**Result:** After installing Fail2Ban, attacker was automatically blocked:
```
[ERROR] could not connect to ssh://192.168.115.129:22 - Connection refused
```

**Verify ban:**
```bash
sudo fail2ban-client status sshd
```
```
Banned IP list:   192.168.115.128
```

---

### Exercise 4: SSH Remote Access

Tested SSH connectivity from attacker to defender machine.

```bash
ssh ronit@192.168.115.129
```

- Confirmed host key fingerprint on first connection
- Authenticated with password
- Demonstrated how SSH provides remote command-line access (port 22)

---

## Key Findings

| Attack | Detection Method | Defense |
|--------|------------------|---------|
| Port Scan | Wireshark SYN packets | UFW firewall |
| Service Enumeration | Unusual traffic patterns | Minimize exposed services |
| SSH Brute Force | auth.log failed attempts | Fail2Ban auto-block |

## Commands Reference

| Task | Command |
|------|---------|
| Check IP address | `ip a` |
| Test connectivity | `ping <ip>` |
| Basic scan | `nmap <target>` |
| Aggressive scan | `nmap -A <target>` |
| Block IP | `sudo ufw deny from <ip>` |
| Allow port | `sudo ufw allow <port>` |
| Check firewall | `sudo ufw status numbered` |
| Delete firewall rule | `sudo ufw delete <number>` |
| Brute force SSH | `hydra -l <user> -P passwords.txt <target> ssh` |
| Check failed logins | `sudo cat /var/log/auth.log \| grep -i failed` |
| Check Fail2Ban status | `sudo fail2ban-client status sshd` |
| Unban IP | `sudo fail2ban-client set sshd unbanip <ip>` |
| SSH into a machine | `ssh <user>@<ip>` |

## Skills Demonstrated

- Network reconnaissance
- Packet analysis
- Firewall configuration
- Brute force attack simulation
- Intrusion detection and prevention
- Log analysis
- Remote access (SSH)
- Linux administration
- Virtualization

## License

MIT License

