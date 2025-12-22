# 💾 Going in Reverse  
**Difficulty:** 2  

---

## 🎯 Challenge Goal  
Reverse engineer a Commodore 64 BASIC program to uncover the hidden flag printed only when the correct password is provided.

---

## 🧠 Scenario Summary  
Kevin, a nostalgic retro-tech fan, finds an old Commodore 64 disk containing a “Security System” BASIC program. Your task is to analyze and reverse engineer this program to discover the password and extract the secret flag.

---

## 🧩 Provided Code  

```basic
10 REM *** COMMODORE 64 SECURITY SYSTEM ***
20 ENC_PASS$ = "D13URKBT"
30 ENC_FLAG$ = "DSA|auhts*wkfi=dhjwubtthut+dhhkfis+hnkz" ' old "DSA|qnisf`bX_huXariz"
40 INPUT "ENTER PASSWORD: "; PASS$
50 IF LEN(PASS$) <> LEN(ENC_PASS$) THEN GOTO 90
60 FOR I = 1 TO LEN(PASS$)
70 IF CHR$(ASC(MID$(PASS$,I,1)) XOR 7) <> MID$(ENC_PASS$,I,1) THEN GOTO 90
80 NEXT I
85 FLAG$ = "" : FOR I = 1 TO LEN(ENC_FLAG$) : FLAG$ = FLAG$ + CHR$(ASC(MID$(ENC_FLAG$,I,1)) XOR 7) : NEXT I : PRINT FLAG$
90 PRINT "ACCESS DENIED"
100 END
```

## 🔍 Step-by-Step Analysis

1. Password check logic
The program compares each character of the user-input password after XORing it with 7.
If it matches the encrypted string ENC_PASS$, access is granted.

```basic
IF CHR$(ASC(MID$(PASS$,I,1)) XOR 7) <> MID$(ENC_PASS$,I,1) THEN GOTO 90
```

2. Decryption formula
Since the program XORs the input with 7 and compares it to ENC_PASS$, we can reverse this logic to find the real password:

```basic
PASS_CHAR = CHR$(ASC(ENC_PASS_CHAR) XOR 7)
```

3. Decrypted password
Performing XOR 7 on "D13URKBT" gives:

C64RULES

4. Flag decryption
Once authenticated, the program decodes ENC_FLAG$ using the same XOR 7 technique.

```basic
FLAG_CHAR = CHR$(ASC(ENC_FLAG_CHAR) XOR 7)
```

Running this operation reveals the hidden flag.

```python
enc_pass = "D13URKBT"
enc_flag = "DSA|auhts*wkfi=dhjwubtthut+dhhkfis+hnkz"

password = ''.join(chr(ord(c) ^ 7) for c in enc_pass)
flag = ''.join(chr(ord(c) ^ 7) for c in enc_flag)

print("Password:", password)
print("Flag:", flag)
```

Password: C64RULES
Flag: CTF{frost-plan:compressors,coolant,oil}
