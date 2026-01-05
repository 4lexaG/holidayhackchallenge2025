# 🎄 Holiday Hack Orientation & Act I 

## 🧭 Challenge Overview

The **Holiday Hack Orientation** and **Act I** challenges are designed to introduce core cybersecurity concepts through practical, hands-on scenarios. These labs progressively cover:

- Threat intelligence extraction and defanging
- Linux privilege escalation via misconfigured sudo
- Network service discovery
- Protocol-level networking fundamentals
- Firewall rule design
- Port scanning and service fingerprinting
- Azure Storage, IAM, NSG, and SAS token misconfigurations

---

## Holiday Hack Orientation

**Objective:**  
Ensure familiarity with the terminal interface and challenge workflow.

**Solution:**  

```
answer
```

---

## It's All About Defang

**Objective:**  
Extract **Indicators of Compromise (IOCs)** from a phishing email and safely prepare them for sharing.

### Regex Used

```
Domains:        [a-zA-Z0-9-]+(\.[a-zA-Z0-9-]+)+
IP addresses:  \d{1,3}(\.\d{1,3}){3}
URLs:          https://[a-zA-Z0-9-]+(\.[a-zA-Z0-9-]+)+(:[0-9]+)?(/[^\s]*)?
Emails:        \b[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}\b
```

### Defanging with sed

```
sed 's/\./[.]/g; s/@/[@]/g; s/http/hxxp/g; s/:\//[://]/g'
```

---

## Neighborhood Watch Bypass

Privilege escalation via PATH hijacking in a sudo-executable script.

**Objective:**  
Regain admin privileges by exploiting a misconfigured sudo rule.

### Process

```
sudo -l
```
Reveals permission to run as root.:
/usr/local/bin/system_status.sh

Vulnerability: PATH Hijacking

The script calls binaries without absolute paths. Since /home/chiuser/bin appears early in $PATH, we can hijack command execution.

```
echo $PATH
```

Exploit Steps

Identify a binary called by the script (e.g., free)

Create a malicious replacement:

```
echo -e '#!/bin/bash\n/home/chiuser/bin/runtoanswer' > ~/bin/free
chmod 777 ~/bin/free
```

Execute the script:

sudo /usr/local/bin/system_status.sh


Result:
/etc/firealarm/restore_fire_alarm is executed as root.


---

## Santa's Gift-Tracking Service Port Mystery

Objective:
Identify the listening port of santa_tracker.

```
ss -tnlp
curl -k 0.0.0.0:12321
```

---

## Visual Networking Thinger

Objective:
Understand packet flow and protocol layering.


Step-by-Step 
1. DNS Lookup — Resolve hostname to IPv4
2. TCP 3-Way Handshake — SYN → SYN/ACK → ACK
3. HTTP GET — Plaintext request
4. TLS Handshake — Key exchange, certificate validation
5. HTTPS GET — Encrypted application data

---

## Visual Firewall Thinger

Objective:
Configure firewall rules according to the principle of least privilege.

Source → Destination	Allowed Traffic
Internet → DMZ	HTTP, HTTPS
DMZ → Internal	HTTP, HTTPS, SSH
Internal → DMZ	HTTP, HTTPS, SSH
Internal → Cloud	HTTP, HTTPS, SSH, SMTP
Internal → Workstations	All

Why this matters:
Improper firewall zoning is a common root cause of lateral movement.

---

## Intro to Nmap

Objective:
Intro to Nmap. reconnaissance, service discovery, and attack surface mapping.

```
nmap 127.0.12.25
nmap 127.0.12.25 -p-
nmap 127.0.12.20-28
nmap -sV 127.0.12.25 -p8080
ncat 127.0.12.25 24601
```

---

## Blob Storage Challenge in the Neighborhood

Blob Storage exposure, leaked SAS tokens, NSG misconfigurations, and IAM/PIM analysis.

---


## The Open Door
Critical Finding:

```
Allow-RDP-From-Internet
```

Allows unrestricted RDP access.

```
az network nsg rule show \
  --nsg-name nsg-production-eastus \
  -n Allow-RDP-From-Internet \
  --resource-group theneighborhood-rg1
 ```
 
 ## Owner
 
Identified PIM-enabled Owner group:

```
the-neighborhood-sub3

```

