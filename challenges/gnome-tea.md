# ☕ Gnome Tea – Technical Write-Up

**Difficulty:** 3  
**Challenge Goal:** Enter the apartment building near 24-7 and help Thomas infiltrate the GnomeTea social network and discover the secret agent passphrase.

---

## 🧠 Scenario Summary

GnomeTea is a social network used by gnomes to “spill the tea” on one another. The application is built on Firebase, exposing client-side configuration and relying heavily on frontend logic for authorization checks.

Thomas (CraHan) suspects the gnomes are using a secret passphrase and asks for help infiltrating the platform to retrieve it.

---

## 🔍 Reconnaissance

### Firebase Client Configuration

By inspecting the frontend JavaScript source code, the following Firebase components were identified:

- Firestore
- Firebase Storage
- Client-side authorization logic

A critical finding was the presence of **hard-coded admin validation** in the frontend:

```js
window.ADMIN_UID
EXPECTED_ADMIN_UID = "3loaihgxP0VwCTKmkHHFLe6FZ4m2"
```

This indicates that admin privileges are decided entirely client-side, which is insecure by design.

### Firestore Enumeration

Using the exposed Firebase project ID, Firestore collections were queried directly:

```bash
curl "https://firestore.googleapis.com/v1/projects/holidayhack2025/databases/(default)/documents/gnomes" > email.txt
```

Email extraction revealed a single backend-only email address:

```bash
grep -E -o '[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\\.[a-zA-Z]{2,}' email.txt
```

Result: barnabybriefcase@gnomemail.dosis


### Firebase Storage Abuse

Analysis of the gnome profile data revealed a reference to a driver’s license image stored in Firebase Storage.

### Incorrect Access Attempt

Direct Google Cloud Storage access failed due to IAM enforcement:

```
curl https://storage.googleapis.com/holidayhack2025.firebasestorage.app/gnome-documents/<file>.jpeg
```

This endpoint does not honor Firebase Storage rules and requires IAM permissions.

### Correct Firebase Storage Endpoint

Firebase Storage must be accessed using:

```bash
curl "https://firebasestorage.googleapis.com/v0/b/holidayhack2025.firebasestorage.app/o/gnome-documents%2Fl7VS01K9GKV5ir5S8suDcwOFEpp2_drivers_license.jpeg"
```

This returned object metadata, including a downloadTokens field.

### File Download via Token

Using the token allows intentional access to the object:

```bash
curl -o license.jpeg \
"https://firebasestorage.googleapis.com/v0/b/holidayhack2025.firebasestorage.app/o/gnome-documents%2Fl7VS01K9GKV5ir5S8suDcwOFEpp2_drivers_license.jpeg?alt=media&token=9292c2a3-ef8d-49f3-9b5b-02a076e63fee"
```

### EXIF Metadata Analysis

The downloaded image contained EXIF metadata with geographic coordinates. The location resolved to Gnomesville on Google Maps.

This location name was used as the password for the previously identified email account.

### Admin Access Bypass

After authenticating, admin access was granted by manipulating client-side variables in the browser console:

window.ADMIN_UID = "3loaihgxP0VwCTKmkHHFLe6FZ4m2"

Reloading the /admin route granted access to the Operations Dashboard.

### Secret Passphrase Disclosure

With admin access, the application loaded the Firestore document:

admins/secret_operations

This document revealed the secret agent passphrase, completing the challenge.

## 🏁 Conclusion

- Firebase client-side configuration should always be considered public
- Authorization decisions must never be enforced in frontend logic
- Firebase Storage download tokens can unintentionally bypass access controls
- EXIF metadata remains a powerful source of sensitive information leakage

