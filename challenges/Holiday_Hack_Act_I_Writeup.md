# 🎄 Holiday Hack Orientation & Act I — Technical Write-Up

**Author:** Cybersecurity Analyst  
**Difficulty:** Introductory → Intermediate  
**Focus Areas:** SOC fundamentals, Linux privilege escalation, networking, cloud security (Azure), threat intelligence handling

---

## 🧭 Challenge Overview

The **Holiday Hack Orientation** and **Act I** challenges are designed to introduce core cybersecurity concepts through practical, hands-on scenarios. These labs progressively cover:

- Threat intelligence extraction and defanging
- Linux privilege escalation via misconfigured sudo
- Network service discovery
- Protocol-level networking fundamentals
- Firewall rule design
- Port scanning and service fingerprinting
- Azure Storage, IAM, NSG, and SAS token misconfigurations

The platform uses **Cranberry Pi terminals**, simulating real-world analyst and engineer workflows.

---

## 🎯 Orientation Challenge

### Challenge: Holiday Hack Orientation

**Objective:**  
Ensure familiarity with the terminal interface and challenge workflow.

**Solution:**  

```
answer
```

---

## 🏛️ Act I — City Hall

### Challenge: It's All About Defang

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

## 🏢 Data Center — Neighborhood Watch Bypass

Privilege escalation via PATH hijacking in a sudo-executable script.

---

## 🏘️ The Neighborhood

### Santa's Gift-Tracking Service Port Mystery

```
ss -tnlp
curl -k 0.0.0.0:12321
```

---

### Visual Networking Thinger

DNS → TCP → HTTP → TLS → HTTPS

---

## 🏨 Grand Hotel — Visual Firewall Thinger

Firewall rules designed with least privilege principles.

---

## 🚗 Parking Lot — Intro to Nmap

```
nmap 127.0.12.25
nmap 127.0.12.25 -p-
nmap 127.0.12.20-28
nmap -sV 127.0.12.25 -p8080
ncat 127.0.12.25 24601
```

---

## ☁️ Azure Security Challenges

Blob Storage exposure, leaked SAS tokens, NSG misconfigurations, and IAM/PIM analysis.

---

## 🧠 Final Thoughts

Most real-world breaches stem from misconfigurations, not zero-days.

---

## 📚 References

- MITRE ATT&CK
- NIST SP 800-41
- RFC 793 / RFC 8446
- Microsoft Azure Security Documentation
- GTFOBins
- Nmap Official Guide
