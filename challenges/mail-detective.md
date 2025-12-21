# 🕵️ Mail Detective – IMAP Investigation Write‑Up

**Difficulty:** 2  
**Challenge Goal:**  
Identify the **pastebin service URL** used by the gnomes to exfiltrate data via malicious JavaScript emails.

---

## 🧠 Scenario Summary

Mo from City Hall needs help investigating suspicious emails sent by mischievous gnomes.  
Due to unsafe email clients, the **only allowed access** to the mail server is via **`curl` over IMAP**.

Our task:
- Access the IMAP mailbox using `curl`
- Review all emails
- Identify how and where data is being exfiltrated

<img width="597" height="458" alt="image" src="https://github.com/user-attachments/assets/7e891c4b-20e2-4c11-96b5-f870e672d3d7" />

---

## 🔐 Accessing the IMAP Server

We authenticate directly to the IMAP service using known credentials.

```bash
curl -u dosismail:holidaymagic imap://localhost:143/INBOX -X "SEARCH ALL"
```

This returns all message IDs stored in the inbox.

---

## 📬 Reading Emails

Each email can be fetched by index:

```bash
curl -u dosismail:holidaymagic imap://localhost:143/INBOX -X "FETCH 1 BODY[]"
```

Or directly viewed using:

```bash
curl -u dosismail:holidaymagic "imap://localhost:143/INBOX;MAILINDEX=1"
```

After reviewing **7 inbox emails**, all messages pointed to strange neighborhood activity involving:
- Tiny tools
- Miniature footprints
- Gnome‑like behavior

⚠️ **None of the inbox emails contained the exfiltration URL.**

---

## 🚫 Checking the Spam Folder

Next, we checked the Spam mailbox.

### Count spam messages
```bash
curl -u dosismail:holidaymagic "imap://localhost:143" -X 'STATUS Spam (MESSAGES)'
```

Result:
```
MESSAGES 3
```

### Search for suspicious keywords
```bash
curl -u dosismail:holidaymagic "imap://localhost:143/Spam" -X 'SEARCH TEXT "pastebin"'
```

Result:
```
SEARCH 2
```

---

## 🧾 Inspecting the Malicious Email

We opened the suspicious spam message:

```bash
curl -u dosismail:holidaymagic "imap://localhost:143/Spam;MAILINDEX=2"
```

Inside, a [JavaScript](../files/mail-detective-spam.txt) payload referenced a **pastebin‑style service** used for exfiltration.

---

## 🧩 Final Findings

### ✅ Pastebin Service URL (Answer)
```
https://frostbin.atnas.mail/api/paste
```

### 🔁 Exfiltration Endpoint Pattern
```
https://frostbin.atnas.mail/raw/<RANDOM_ID>
```

The gnomes send stolen data to this endpoint using randomly generated IDs.

---

## 🏁 Conclusion

By safely interacting with the IMAP server using `curl`, we:
- Enumerated inbox and spam emails
- Identified malicious content in Spam
- Discovered the pastebin service used for data exfiltration

🎄 Case closed — the gnomes’ secret pastebin is exposed!  
