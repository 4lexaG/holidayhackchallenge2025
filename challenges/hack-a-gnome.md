# 🤖 Hack-a-Gnome

**Difficulty:** 3  
**Challenge Goal:**  
Davis in the Data Center is fighting a gnome army — join the hack‑a‑gnome fun and help turn one of these rebellious bots against its own kind.

---

## 🧠 Scenario Summary

This challenge chains **Prototype Pollution**, **EJS Server‑Side Template Injection**, and **Remote Code Execution** to gain root command execution on a smart gnome controller.  
Once shell access is obtained, the task shifts to **CAN bus reverse engineering** to identify undocumented movement commands and physically control the robot.

---

## 📖 Challenge Description

Hey, I could really use another set of eyes on this gnome takeover situation.

Their systems now have **multiple layers of protection**:
- Database authentication
- Web application security controls
- Additional defensive mechanisms

But every system has weaknesses if you know where to look.

Are you ready to help me **hack one of these rebellious bots** and turn it against the rest of the gnome army?

---

## 🎯 Objectives

- Analyze the gnome control systems
- Identify weaknesses across different protection layers
- Exploit at least one vulnerable component
- Gain control over a gnome bot

---

## 1. Vulnerability Summary

| Stage | Vulnerability | Impact |
|-----|--------------|--------|
| Web API | Prototype Pollution | Arbitrary object property overwrite |
| Templating | EJS SSTI | Arbitrary JS execution |
| OS | Command Execution | Root shell |
| Embedded | CAN Bus Logic Flaw | Physical device control |

---

## 2. Prototype Pollution via `/ctrlsignals`

The `/ctrlsignals` endpoint accepts a JSON object via URL encoding.

By abusing unsafe object merging, `Object.prototype` was polluted, allowing control of internal EJS options.

---

## 3. EJS Server‑Side Template Injection (SSTI)

The polluted `outputFunctionName` property is used by EJS during rendering.  
Injected JavaScript executes when `/stats` is accessed.

This results in **root‑level command execution**.

---

## 4. File System Enumeration

Command execution was used to exfiltrate files via Burp Collaborator using base64 encoding.

Key files:
- `canbus_client.py`
- `README.md`

---

## 5. CAN Bus Documentation Analysis

The leaked README documented CAN IDs on `gcan0` but explicitly stated movement commands were unstable.

---

## 6. Reverse Engineering Movement Commands

Brute‑forcing CAN IDs revealed valid movement signals.

```python
for cmd_id in range(0x1FF, 0x205):
    send_command(bus, cmd_id)
    time.sleep(1.3)
```

---

## 7. Final Command Mapping

```python
COMMAND_MAP = {
    "up":    0x201,
    "down":  0x202,
    "left":  0x203,
    "right": 0x204,
}
```

Using these, the gnome could move crates and reach the control console.

<video width="640" height="360" controls>
  <source src="../images/hag.mp4" type="video/mp4">
 error <code>video</code>.
</video>

---

## 8. Impact

- Full server compromise
- Root RCE
- Physical device manipulation

---

## 9. Mitigations

- Block prototype keys
- Disable dynamic EJS options
- Authenticate CAN frames

---

## 10. References

- https://portswigger.net/web-security/nosql-injection
- https://eslam.io/posts/ejs-server-side-template-injection-rce/
- https://www.kayssel.com/newsletter/issue-24/
- https://learn.microsoft.com/en-us/cosmos-db/query/functions
- https://www.kvaser.com/about-can/

---

🎅 Hack responsibly.

