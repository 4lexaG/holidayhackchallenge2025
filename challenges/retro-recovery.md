# KringleCon 2025 – Act 2, Challenge 1  
## Retro Recovery  

**Difficulty:** 2  

---

## Overview

In this challenge, we step into Mark’s retro shop to perform digital archaeology on a classic **FAT12 floppy disk image**.  
The objective is to recover deleted or hidden data from an IBM PC–era floppy and uncover a secret message embedded inside a BASIC program.

This challenge highlights a timeless truth of filesystems: **deleted does not always mean gone**.

> *“Sometimes it is the people no one can imagine anything of who do the things no one can imagine.”*  
> — Alan Turing

---

## Challenge Objectives

Analyze a floppy disk image to:

- Enumerate existing, hidden, and deleted files
- Recover BASIC source code from filesystem artifacts
- Identify and decode an encoded message hidden in comments

---

## Provided Artifact

[![Floppy Disk](../images/floppy.png)](../images/floppy.png)
You got yourself a floppy disk image from an old IBM PC! Retro!!!!

---

## Tools & Techniques

### Forensic Utilities

- **The Sleuth Kit (TSK)**
  - `fls` — list files and directories (including deleted entries)
  - `icat` — extract file contents by inode
- **file** — determine file type
- **base64** — decode encoded strings

---

## Technical Walkthrough

### Step 1: Enumerate Files on the Floppy Image

We begin by listing all files recursively from the root of the filesystem:

```bash
fls -o 0 -r floppy.img
```

**Explanation:**

- `-o 0` specifies the filesystem offset (typical for raw floppy images)
- `-r` enables recursive directory traversal

This output reveals multiple file entries, including deleted or unlinked files that are no longer visible to the operating system.

---

### Step 2: Extract the Primary BASIC Program

One notable file entry appears at **inode 6**. We extract it using `icat`:

```bash
icat -o 0 floppy.img 6 > all.bas
```

Verify the extracted file type:

```bash
file all.bas
```

**Result:**

```
ASCII text
```

This confirms we have recovered readable source code.

---

### Step 3: Identify Hidden or Fragmented Data

Another interesting inode (**10**) appears to contain residual data:

```bash
icat -o 0 floppy.img 10 > hidden_fragment
```

Check its file type:

```bash
file hidden_fragment
```

**Result:**

```
Nano swap file, pid 4209, user mark, host arcade, file all_i-want_for_christmas.bas, modified
```

This suggests remnants of editing activity tied to the BASIC program.

---

### Step 4: Recover the Full BASIC Source

Re-extract the complete program with a meaningful filename:

```bash
icat -o 0 floppy.img 6 > all_i_want_for_christmas.bas
```

This ensures we are working with the full and intended source file.

---

### Step 5: Locate the Encoded Message

While reviewing the BASIC source, a suspicious comment appears at **line 211**:

```basic
REM bWVycnkgY2hyaXN0bWFzIHRvIGFsbCBhbmQgdG8gYWxsIGEgZ29vZCBuaWdodAo=
```

The character set strongly suggests **Base64 encoding**.

---

### Step 6: Decode the Hidden Message

Decode the string using `base64`:

```bash
echo "bWVycnkgY2hyaXN0bWFzIHRvIGFsbCBhbmQgdG8gYWxsIGEgZ29vZCBuaWdodAo=" | base64 -d
```

**Decoded Output:**

```
merry christmas to all and to all a good night
```

---

## Conclusion

This challenge demonstrates classic filesystem forensics:

- FAT filesystems preserve data even after deletion
- BASIC programs can hide secrets in plain sight using comments
- Simple encodings like Base64 remain effective for obfuscation

A nostalgic reminder that early computing—and its artifacts—still have stories to tell.

---

🎄 **Challenge Complete** 🎄
