# 🧠 MOn the Wire Write‑Up

**Difficulty:** Medium  
**Protocols:** 1‑Wire → SPI → I2C  
**Goal:** Recover the final sensor value transmitted over I2C  

---

## 📖 Overview

This challenge involved progressively decoding data transmitted over multiple embedded communication protocols. Each stage revealed a new hint and encryption key required to advance to the next protocol.

The workflow was:

1. Identify repeated signal streams
2. Decode **SPI** traffic and XOR‑decrypt it
3. Use the SPI output to obtain the XOR key for **I2C**
4. Decode **I2C** traffic and extract the final sensor value

---

## 🔁 Repeated Data Observation

The captured signal data repeats continuously. This can be observed by inspecting the timestamp (`t`) field, which resets to `0` after reaching `32871 ns`:

```json
{"line":"dq","t":32871,"v":1}
```

This confirms that the same payload is transmitted in a loop, allowing us to safely analyze a single iteration.

---

## 🔐 XOR Key (SPI)

From the previous stage, the XOR key for SPI traffic was recovered as:

```
icy
```

---

## 🧩 Step 2 – SPI Protocol Analysis

### 🔍 SPI Background

SPI (Serial Peripheral Interface) is a synchronous serial protocol commonly used between microcontrollers and peripherals.

Relevant SPI lines:

- **SCK** – Clock
- **MOSI** – Master Out, Slave In
- **MISO** – Master In, Slave Out (not provided)
- **CS/SS** – Chip Select (not provided)

In this challenge, only **MOSI** and **SCK** were available.

The presence of `"marker": "idle-low"` and sampling on rising edges indicates **SPI Mode 0 (CPOL=0, CPHA=0)**.

---

### 🛠️ SPI Decoding Strategy

To reconstruct SPI data:

1. Iterate over all `sck` entries with `"marker": "sample"`
2. For each clock sample time `t`, retrieve the most recent MOSI value at or before `t`
3. Collect bits in order
4. Group bits into bytes (8 bits, MSB first)
5. XOR‑decode the resulting byte stream using the key `icy`

---

### 🧪 SPI Decoder Script

```bash
python3 spi-decoder.py sck.json mosi.json
```

*(Script omitted here for brevity — identical to the one used during solving)*

---

### 📤 SPI Decoder Output

```
Plaintext:
read and decrypt the I2C bus data using the XOR key: bananza.
the temperature sensor address is 0x3C
```

---

## 🔑 XOR Key (I2C)

From the decoded SPI message:

```
bananza
```

---

## 🧩 Step 3 – I2C Protocol Analysis

### 🔍 I2C Background

I2C (Inter‑Integrated Circuit) is a two‑wire serial bus:

- **SDA** – Data
- **SCL** – Clock

I2C communication consists of:

1. **Start condition**
2. **Address byte** (7‑bit address + R/W bit)
3. **ACK/NACK**
4. **Data bytes**
5. **Stop condition**

---

### 🛠️ I2C Decoding Strategy

1. Merge `sda.json` and `scl.json`
2. Split data into transactions using `start` / `stop`
3. Reconstruct bytes using bit indices
4. Extract the 7‑bit address (`first_byte >> 1`)
5. Ignore ACK bits
6. XOR‑decode data bytes using key `bananza`
7. Print decoded plaintext per transaction

---

### 🧪 I2C Decoder Script

```bash
python3 i2c-decoder.py sda.json scl.json
```

*(Script omitted here for brevity — identical to the one used during solving)*

---

### 📤 I2C Decoder Output

```
Transaction 2:
  Address (hex): ['0x3C']
  Raw Data: [81, 83, 64, 89, 90]
  Decoded Data: [51, 50, 46, 56, 52]
  Plaintext: 32.84
```

---

This value corresponds to the plaintext data sent by the I2C device at address **0x3C**, which was explicitly identified in the SPI stage.

---

🎄 *Holiday Hack Challenge – Signal Analysis Complete* 🎄
