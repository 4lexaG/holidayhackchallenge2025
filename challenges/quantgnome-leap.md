# 🧬 Quantgnome Leap

**Difficulty:** 2\
**Challenge Goal:** Charlie in the hotel has quantum gnome mysteries
waiting to be solved. What is the flag that you find?

------------------------------------------------------------------------

## 🧩 Scenario Summary

I'm not JJ. I like music, quantum pancakes, and social engineering. I
accept AI tokens.

I just spotted a mysterious gnome -- he winked and vanished, or maybe he's still here?
Things are getting strange, and I think we've wandered into a quantum conundrum!

If you help me unravel these riddles, we might just outsmart future
quantum computers.

Cryptic puzzles, quirky gnomes, and post-quantum secrets---will you leap
with me?

------------------------------------------------------------------------

## 🎯 Challenge Overview

This challenge introduces **Post-Quantum Cryptography (PQC)** concepts
through a sequence of SSH authentications.\
Each "gnome" account can only be accessed using a specific **SSH key**,
progressing from classical cryptography to **hybrid post‑quantum
algorithms** based on the **Open Quantum Safe (OQS)** project.

The goal is to: 1. Identify SSH keys stored on the system 2. Inspect
their *comments* to determine the next user 3. Authenticate locally
using SSH with the correct private key 4. Progress through all gnome
users until reaching `admin` 5. Locate the final flag

------------------------------------------------------------------------

## 🔐 Step-by-Step Solution

### 🟢 Step 1: Initial Access (`qgnome → gnome1`)

The initial hint pointed toward **SSH keys and their comments**.\
Inspecting `~/.ssh/id_rsa.pub` revealed a comment:

    gnome1

Login:

``` bash
ssh gnome1@localhost
```

This authentication used **RSA**, which is vulnerable in a post‑quantum
context (Shor's algorithm).

------------------------------------------------------------------------

### 🟢 Step 2: ED25519 (`gnome1 → gnome2`)

Inside `/home/gnome1/.ssh/`:

-   `id_ed25519`
-   `id_ed25519.pub` (comment: `gnome2`)

Login:

``` bash
ssh -i ~/.ssh/id_ed25519 gnome2@localhost
```

ED25519 is smaller and faster than RSA, but still **not quantum‑safe**.

------------------------------------------------------------------------

### 🟢 Step 3: MAYO (`gnome2 → gnome3`)

Inside `/home/gnome2/.ssh/`:

-   `id_mayo2`
-   `id_mayo2.pub`

Login:

``` bash
ssh -i ~/.ssh/id_mayo2 gnome3@localhost
```

This introduces **MAYO**, a post‑quantum signature scheme.

------------------------------------------------------------------------

### 🟢 Step 4: Hybrid ECDSA + SPHINCS+ (`gnome3 → gnome4`)

Inside `/home/gnome3/.ssh/`:

-   `id_ecdsa_nistp256_sphincssha2128fsimple`
-   Hybrid key: **ECDSA P‑256 + SPHINCS+**

Login:

``` bash
ssh -i ~/.ssh/id_ecdsa_nistp256_sphincssha2128fsimple gnome4@localhost
```

This is a **hybrid post‑quantum key**, validating both classical and PQC
signatures.

------------------------------------------------------------------------

### 🟢 Step 5: Security Level 5 (`gnome4 → admin`)

Inside `/home/gnome4/.ssh/`:

-   `id_ecdsa_nistp521_mldsa87`
-   Hybrid key: **ECDSA P‑521 + ML‑DSA‑87 (Dilithium)**

Login:

``` bash
ssh -i ~/.ssh/id_ecdsa_nistp521_mldsa87 admin@localhost
```

This is a **NIST Security Level 5** configuration, equivalent to
**AES‑256** strength.

------------------------------------------------------------------------

## 🏁 Final Step: Flag Retrieval

The final instructions indicated access near the SSH daemon.

Flag location:

``` bash
cat /opt/oqs-ssh/flag/flag
```

### 🏆 Flag

    HHC{L3aping_0v3r_Quantum_Crypt0}

------------------------------------------------------------------------

## 📚 Additional Resources

-   **Open Quantum Safe (OQS):** https://openquantumsafe.org/
-   **PQC in SSH:** https://github.com/open-quantum-safe/openssh
-   **NIST PQC Standards:**
    -   FIPS 204 --- ML‑DSA (Dilithium)
    -   FIPS 205 --- SLH‑DSA (SPHINCS+)
-   **Quantum-safe TLS test:** https://isitquantumsafe.info/
-   **PQC ecosystem overview:** https://pkic.org/pqccm/

------------------------------------------------------------------------

## 🧠 Key Takeaways

-   SSH key *comments* are critical metadata
-   Hybrid cryptography enables **quantum agility**
-   Post‑quantum adoption is already practical today
-   OQS provides a real-world implementation path

This challenge effectively demonstrates how cryptography is evolving to
remain secure in a **post‑quantum future**.
