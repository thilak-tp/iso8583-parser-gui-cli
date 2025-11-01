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

**Message Type Indicator (MTI)** 
- It is a 4 digit where each digit place had it's own meaning
- Consider an MTI with digits as ABCD
1. A - Indications the ISO 8583 version being used
- 0 - Stands for 1987 Version
- 1 - Stands for 1993 Version
- 2 - Stands for 2003 Version
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
