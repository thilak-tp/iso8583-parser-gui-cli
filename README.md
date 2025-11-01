<!-- ISO8583 Parser README -->
<h1 align="center">💳 ISO8583 Parser — CLI & Win32 GUI</h1>
<p align="center">
  <b>A C++ Utility for Packing and Unpacking ISO8583 Financial Messages</b><br>
  <sub>By <b>Thilak</b> — Systems & Network Programming Enthusiast</sub>
</p>

<p align="center">
  <img src="https://img.shields.io/badge/language-C++17-blue.svg?style=for-the-badge">
  <img src="https://img.shields.io/badge/platform-Windows-lightgrey.svg?style=for-the-badge">
  <img src="https://img.shields.io/badge/build-CMake-orange.svg?style=for-the-badge">
  <img src="https://img.shields.io/badge/license-MIT-green.svg?style=for-the-badge">
</p>

---

## 🧭 Overview  

**ISO 8583** is the **international standard** for systems that exchange electronic transactions made by cardholders using payment cards.  
It defines message structure, field specifications, and flow for financial systems like **ATMs**, **POS terminals**, and **payment gateways**.

This project implements a **C++ ISO8583 Message Parser** that can:  
- 🧩 **Pack** data elements into a valid ISO8583 message  
- 🔍 **Unpack** ISO8583 messages into readable data elements  
- 💻 Run as a **Command-Line Tool** or a **Win32 GUI Application**

---

## ⚙️ Project Phases  

### 🧠 Phase 1 — Research  
> Understanding the ISO8583 standard, message structure, and field mappings.

**ISO8583 Message Format Example:**  
Each ISO8583 message typically includes:
1. **Message Type Indicator (MTI)** – Defines the type of transaction (e.g., authorization, financial, reversal, etc.)  
2. **Bitmap** – Indicates which data elements are present in the message  
3. **Data Elements** – Contain actual transaction information such as card number, amount, date, etc.  

<p align="center">
  <img src="resources/iso8583format.png" alt="ISO8583 Message Format" width="600"/>
</p>

## 🧾 Message Type Indicator (MTI)

The **MTI (Message Type Indicator)** in ISO 8583 is a 4-digit numeric field where **each digit has a specific meaning**.

Let’s represent the MTI as **ABCD**:

| Position | Meaning |
|-----------|----------|
| **A** | ISO 8583 version |
| **B** | Message class (Purpose) |
| **C** | Message function (Usage type) |
| **D** | Message origin |

---

### 🔹 1. A — ISO 8583 Version

| Value | Meaning |
|--------|----------|
| **0** | 1987 Version |
| **1** | 1993 Version |
| **2** | 2003 Version |

---

### 🔹 2. B — Message Class (Purpose)

| Value | Meaning |
|--------|----------|
| **1** | Authorization Message |
| **2** | Financial Message / Withdrawal |
| **3** | File Actions (Batch Transfers, etc.) |
| **4** | Reversal or Chargeback |
| **5** | Reconciliation |
| **6** | Administrative |
| **7** | Fee Collection / Network Management |
| **8** | Reserved for Private Use |

---

### 🔹 3. C — Message Function (Usage Type)

| Value | Meaning |
|--------|----------|
| **0** | Request |
| **1** | Response |
| **2** | Advice |
| **3** | Advice Response |
| **4** | Notification |

---

### 🔹 4. D — Message Origin

| Value | Meaning |
|--------|----------|
| **0** | Acquirer |
| **1** | Acquirer Repeat |
| **2** | Issuer |
| **3** | Issuer Repeat |
| **4** | Other (Switch, Network, etc.) |

---

### 🧠 Example

If **MTI = 1200**:
- **1** → 1993 Version  
- **2** → Financial Message  
- **0** → Request  
- **0** → Originated from Acquirer  

So, **MTI 1200** = *Financial Request message from Acquirer (1993 version)*.

## 🧩 Bitmap in ISO 8583

The **Bitmap** in ISO 8583 indicates which **Data Elements (Fields)** are present in the message.  
Each bit (0 or 1) represents whether a corresponding **Data Element (DE)** exists in the message.

---

### 🧠 Concept Overview

- The Bitmap is a **64-bit field** (or sometimes **128 bits**).
- Each bit corresponds to a **Data Element (DE)** — for example:
  - **Bit 1** → Secondary bitmap indicator  
  - **Bit 2** → DE 2 (Primary Account Number)  
  - **Bit 3** → DE 3 (Processing Code)  
  - …and so on up to **Bit 64 / 128**
- A bit value of:
  - **1 → Field present in the message**
  - **0 → Field absent**

---

### 🧮 Types of Bitmaps

| Bitmap Type | Description | Length | Data Elements Covered |
|--------------|--------------|---------|------------------------|
| **Primary Bitmap** | Always present | 64 bits (8 bytes) | Fields **1–64** |
| **Secondary Bitmap** | Present if bit 1 of Primary = 1 | 64 bits (8 bytes) | Fields **65–128** |
| **Tertiary Bitmap** *(optional)* | Rare, for extended specs | 64 bits (8 bytes) | Fields **129–192** |

---

### 🧰 Representation Formats

| Format | Description | Example |
|---------|--------------|----------|
| **Binary** | 64 bits of 0s and 1s | `1110001100000001000000000000000000000000000000000000000000000000` |
| **Hexadecimal** | 16 hex characters (each hex = 4 bits) | `E301000000000000` |

> 💡 **Tip:**  
> In most ISO 8583 implementations, the Bitmap is transmitted as a **16-character hexadecimal string**.

---

### 🧩 Example Breakdown

**Example Bitmap (Hex):**
( 7230001000000000 ) Decimal = (0111001000110000000000000001000000000000000000000000000000000000) Binary


**Interpretation:**

| Bit | Value | Meaning |
|-----|--------|----------|
| 1 | 0 | No secondary bitmap |
| 2 | 1 | DE 2 (Primary Account Number) present |
| 3 | 1 | DE 3 (Processing Code) present |
| 4 | 1 | DE 4 (Transaction Amount) present |
| 11 | 1 | DE 11 (System Trace Audit Number) present |
| ... | ... | Remaining bits = 0 → Fields absent |

---

### 🧾 Summary

- The Bitmap acts as a **map of active fields** in an ISO 8583 message.
- It ensures **variable-length message flexibility** — only necessary fields are transmitted.
- Parsing the Bitmap is the **first step** before decoding the data fields.

---

### 🧩 Phase 2 — Backend Development  
- Implement ISO8583 message structures (**MTI**, **Bitmaps**, **Data Elements**)  
- Create flexible **packer/unpacker** logic  
- Support **multiple ISO8583 versions** (1987 / 1993 / 2003)  
- Handle **variable length** fields (LLVAR, LLLVAR)  
- Ensure robust **error handling and validation**

---

### 💬 Phase 3 — CLI Application  
A command-line interface for developers and testers to easily pack/unpack ISO8583 messages.  

**Example Commands:**  
```bash
# Pack JSON input into ISO8583 message
iso8583cli --pack input.json --version 1987

# Unpack raw ISO8583 hex message into readable format
iso8583cli --unpack message.hex
