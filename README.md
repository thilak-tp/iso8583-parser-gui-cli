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

## 🧾 ISO 8583 Data Elements (Fields)

ISO 8583 messages are composed of a **Message Type Indicator (MTI)**, a **Bitmap**, and a series of **Data Elements (DEs)**.  
Each **Data Element** carries a specific piece of transaction-related information.

---

### 🧩 Overview

- Each Data Element (DE) is identified by a number (e.g., DE 2 = Primary Account Number).
- The **Bitmap** indicates which fields are present in the message.
- ISO 8583 defines up to **128 standard fields** (some variants extend to 192).

---

### 🧱 Primary Data Elements (Fields 1–64)

| Field No. | Field Name | Description | Example |
|------------|-------------|--------------|----------|
| **1** | Secondary Bitmap | Indicates presence of fields 65–128 | 1 = Present |
| **2** | Primary Account Number (PAN) | Card number | `4567890123456789` |
| **3** | Processing Code | Identifies transaction type | `000000` |
| **4** | Transaction Amount | Amount of transaction | `000000010000` |
| **5** | Settlement Amount | Amount to be settled | `000000010000` |
| **6** | Cardholder Billing Amount | Amount in cardholder’s currency | `000000010000` |
| **7** | Transmission Date & Time | MMDDhhmmss | `1024163045` |
| **8** | Cardholder Billing Fee | Fee charged | `000000001000` |
| **9** | Settlement Conversion Rate | Exchange rate | `000000001234` |
| **10** | Cardholder Billing Conversion Rate | Conversion rate | `000000001234` |
| **11** | System Trace Audit Number (STAN) | Unique transaction trace ID | `123456` |
| **12** | Local Transaction Time | hhmmss | `163045` |
| **13** | Local Transaction Date | MMDD | `1024` |
| **14** | Card Expiration Date | YYMM | `2612` |
| **15** | Settlement Date | MMDD | `1024` |
| **16** | Conversion Date | MMDD | `1024` |
| **17** | Capture Date | MMDD | `1024` |
| **18** | Merchant Category Code | MCC code for business type | `6011` |
| **19** | Acquiring Institution Country Code | Numeric country code | `356` |
| **20** | PAN Country Code | Issuer country code | `356` |
| **21** | Forwarding Institution Country Code | Network country code | `356` |
| **22** | POS Entry Mode | Card input type | `021` |
| **23** | Card Sequence Number | Card index | `001` |
| **24** | Network International ID | Network code | `001` |
| **25** | POS Condition Code | POS transaction condition | `00` |
| **26** | POS PIN Capture Code | Max PIN length | `04` |
| **27** | Authorization ID Response Length | Length of auth ID | `06` |
| **28** | Amount, Transaction Fee | Merchant fee | `D00000100` |
| **29** | Amount, Settlement Fee | Settlement fee | `C00000100` |
| **30** | Amount, Transaction Processing Fee | Processor fee | `C00000100` |
| **31** | Amount, Settlement Processing Fee | Settlement processor fee | `C00000100` |
| **32** | Acquiring Institution ID Code | Acquirer bank code | `123456` |
| **33** | Forwarding Institution ID Code | Forwarder ID | `789012` |
| **34** | PAN Extended | Secondary PAN info | `5678901234567890` |
| **35** | Track 2 Data | Card track data | `4567890123456789D2612201123456789` |
| **36** | Track 3 Data | Optional card data | — |
| **37** | Retrieval Reference Number | Transaction reference | `654321123456` |
| **38** | Authorization Identification Response | Auth code | `ABC123` |
| **39** | Response Code | Result of transaction | `00` = Approved |
| **40** | Service Restriction Code | Service limits | `201` |
| **41** | Card Acceptor Terminal ID | Terminal ID | `ATM12345` |
| **42** | Card Acceptor ID Code | Merchant ID | `M12345678901234` |
| **43** | Card Acceptor Name/Location | Merchant name/location | `ABC STORE / MUMBAI` |
| **44** | Additional Response Data | Optional info | `Approved` |
| **45** | Track 1 Data | Magnetic track 1 info | `B4567890123456789^DOE/JOHN` |
| **46** | Additional Data – ISO | Additional field | — |
| **47** | Additional Data – National | Country-specific field | — |
| **48** | Additional Data – Private | Private field | — |
| **49** | Currency Code, Transaction | ISO 4217 numeric code | `356` |
| **50** | Currency Code, Settlement | Settlement currency | `356` |
| **51** | Currency Code, Cardholder Billing | Billing currency | `840` |
| **52** | Personal Identification Number (PIN) Data | Encrypted PIN block | — |
| **53** | Security Related Control Info | Security data | — |
| **54** | Additional Amounts | Extra amount fields | — |
| **55** | ICC (EMV) Data | Chip card data | — |
| **56** | Reserved (ISO) | Reserved | — |
| **57** | Reserved (National) | Reserved | — |
| **58** | Reserved (Private) | Reserved | — |
| **59** | Reserved (National Use) | Reserved | — |
| **60** | Advice/Batch Number | Message advice code | `000001` |
| **61** | Additional Data – National | National data | — |
| **62** | Additional Data – Private | Private data | — |
| **63** | Network Data | Network-specific info | — |
| **64** | Message Authentication Code (MAC) | Security MAC | `A1B2C3D4` |

---

### 🧱 Secondary Data Elements (Fields 65–128)

| Field No. | Field Name | Description |
|------------|-------------|--------------|
| **65** | MAC Extension | Extended MAC data |
| **66** | Settlement Code | Settlement type indicator |
| **67** | Extended Payment Code | Payment extension info |
| **68** | Receiving Institution Country Code | Receiver’s country |
| **69** | Settlement Institution Country Code | Settlement country |
| **70** | Network Management Information Code | Network command type (e.g., logon, echo) |
| **71** | Message Number | Message counter |
| **72** | Data Record | Transaction data record |
| **73** | Date Action | Date of action |
| **74** | Credits, Number | No. of credit transactions |
| **75** | Credits, Reversal Number | No. of reversed credits |
| **76** | Debits, Number | No. of debit transactions |
| **77** | Debits, Reversal Number | No. of reversed debits |
| **78** | Transfer Number | No. of transfers |
| **79** | Transfer Reversal Number | No. of reversed transfers |
| **80** | Inquiries Number | No. of inquiries |
| **81** | Authorizations Number | No. of authorizations |
| **82** | Credits, Processing Fee Amount | Total credit fees |
| **83** | Credits, Transaction Fee Amount | Total credit transaction fees |
| **84** | Debits, Processing Fee Amount | Total debit fees |
| **85** | Debits, Transaction Fee Amount | Total debit transaction fees |
| **86** | Credits, Amount | Total credit amount |
| **87** | Credits, Reversal Amount | Total reversed credits |
| **88** | Debits, Amount | Total debit amount |
| **89** | Debits, Reversal Amount | Total reversed debits |
| **90** | Original Data Elements | Original MTI/fields for reversal |
| **91** | File Update Code | Batch update code |
| **92** | File Security Code | File authentication |
| **93** | Response Indicator | Response status |
| **94** | Service Indicator | Network service indicator |
| **95** | Replacement Amounts | Replacement transaction amounts |
| **96** | Message Security Code | Security value |
| **97** | Net Settlement Amount | Net settlement |
| **98** | Payee | Payee name/account |
| **99** | Settlement Institution ID Code | Settlement institution |
| **100** | Receiving Institution ID Code | Receiving institution |
| **101** | File Name | File transfer name |
| **102** | Account Identification 1 | From account | `1234567890` |
| **103** | Account Identification 2 | To account | `9876543210` |
| **104** | Transaction Description | Transaction purpose |
| **105–112** | Reserved for ISO Use | Reserved |
| **113–120** | Reserved for National Use | Reserved |
| **121–128** | Reserved for Private Use | Reserved |

---

### 🧾 Example Summary

**Example Message (simplified)**
MTI: 0200
Bitmap: F23C44C128E08010
Data Elements Present: 2, 3, 4, 7, 11, 12, 37, 41, 49

| Field | Description | Example Value |
|--------|--------------|----------------|
| DE 2 | Primary Account Number | `4567890123456789` |
| DE 3 | Processing Code | `000000` |
| DE 4 | Transaction Amount | `000000010000` |
| DE 7 | Transmission Date & Time | `1024163045` |
| DE 11 | STAN | `123456` |
| DE 12 | Local Time | `163045` |
| DE 37 | Retrieval Reference Number | `654321123456` |
| DE 41 | Terminal ID | `ATM12345` |
| DE 49 | Currency Code | `356` |

---

### 🧠 Key Takeaways

- The **Bitmap** determines which **Data Elements** are included.
- **Fields 1–64** → Primary Bitmap, **65–128** → Secondary Bitmap.  
- Each field provides a **specific transaction detail** (amount, time, ID, etc.).
- Parsing correctly requires understanding **field formats and lengths**.

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
