# 💾 Retro Recovery – FAT12 Forensics Write-Up

**Difficulty:** 2  
**Challenge Goal:**  
Recover deleted or hidden data from a **FAT12 floppy disk image** and uncover a secret message hidden inside a BASIC program.

---

## 🧠 Scenario Summary

We step into **Mark’s retro shop** to perform digital archaeology on an old IBM PC–era floppy disk.  
Although files may appear deleted, FAT filesystems often leave data behind.

Our mission:
- Analyze the floppy disk image
- Recover BASIC source code from filesystem artifacts
- Identify and decode a hidden message

> *“Deleted does not always mean gone.”*

---

## 📦 Provided Artifact

- **floppy.img** — FAT12 floppy disk image from an IBM PC

---

## 🛠️ Tools Used

- **The Sleuth Kit (TSK)**
  - `fls` — list files (including deleted entries)
  - `icat` — extract file contents by inode
- **file** — identify file types
- **base64** — decode encoded data

---

## 🔍 Investigation Walkthrough

### Step 1: Enumerate Files on the Floppy

List all files recursively from the root directory:

```bash
fls -o 0 -r floppy.img
```

**Why this works:**
- `-o 0` specifies the filesystem offset (standard for raw floppy images)
- `-r` enables recursive listing

This reveals active, hidden, and deleted file entries.

---

### Step 2: Extract the BASIC Program

A notable file appears at **inode 6**. Extract it:

```bash
icat -o 0 floppy.img 6 > all.bas
```

Verify the file type:

```bash
file all.bas
```

**Result:**
```
ASCII text
```

The BASIC source code was successfully recovered.

---

### Step 3: Analyze Residual Data

Another inode (**10**) contains leftover data from editing activity:

```bash
icat -o 0 floppy.img 10 > hidden_fragment
```

Check the file type:

```bash
file hidden_fragment
```

**Result:**
```
Nano swap file, pid 4209, user mark, host arcade, file all_i-want_for_christmas.bas, modified
```

This confirms remnants of a BASIC editing session.

---

### Step 4: Recover the Full Source File

Re-extract the program using a meaningful filename:

```bash
icat -o 0 floppy.img 6 > all_i_want_for_christmas.bas
```

This ensures we are working with the intended source code.

---

### Step 5: Locate the Hidden Message

While reviewing the BASIC file, a suspicious comment appears:

```basic
REM bWVycnkgY2hyaXN0bWFzIHRvIGFsbCBhbmQgdG8gYWxsIGEgZ29vZCBuaWdodAo=
```

The format strongly suggests **Base64 encoding**.

---

### Step 6: Decode the Message

Decode the string:

```bash
echo "bWVycnkgY2hyaXN0bWFzIHRvIGFsbCBhbmQgdG8gYWxsIGEgZ29vZCBuaWdodAo=" | base64 -d
```

**Decoded Output:**
```
merry christmas to all and to all a good night
```

---

## 🧩 Final Findings

### ✅ Hidden Message (Answer)
```
merry christmas to all and to all a good night
```

---

## 🏁 Conclusion

This challenge highlights classic filesystem forensics concepts:

- FAT12 retains data after deletion
- Swap files and fragments reveal editing history
- BASIC comments are perfect places to hide encoded messages

💾 A nostalgic dive into retro computing — mystery recovered!
