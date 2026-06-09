# 📡 IOT-BASED-SMART-ATTENDANCE-SYSTEM

> An IoT-enabled embedded system for automated classroom attendance tracking using RFID, ESP32, LoRa SX1278, GSM SIM800L, and Google Sheets — with real-time parental SMS alerts for attendance shortages.

---

## 📋 Table of Contents

- [Overview](#overview)
- [Features](#features)
- [System Architecture](#system-architecture)
- [Hardware Components](#hardware-components)
- [Software Tools](#software-tools)
- [Circuit & Pin Connections](#circuit--pin-connections)
- [How It Works](#how-it-works)
- [Communication Protocols](#communication-protocols)
- [Results](#results)
- [Future Scope](#future-scope)

---

## Overview

Manual attendance systems in educational institutions are time-consuming, error-prone, and offer no real-time visibility to parents or administrators. This project addresses that problem with a **fully automated, IoT-based classroom attendance system** built on embedded hardware.

Each classroom is equipped with a **Transmitter Unit** — an RFID-RC522 reader interfaced with an ESP32 microcontroller. Students tap their RFID tags once per class period. The ESP32 tracks attendance for up to **8 class periods per day** and transmits the records wirelessly via **LoRa SX1278** at the end of each period. A single **Receiver Unit** installed at the institution's admin block receives the LoRa packets, pushes hourly attendance data to a **Google Sheets spreadsheet** via a deployed Google Apps Script Web App, and dispatches **SMS alerts via GSM SIM800L** to parents of students whose cumulative attendance falls below 75% — all **without any manual intervention**.

Both units are powered independently through **LM2596 buck converters**, making them suitable for mains-fed installation across any classroom or block.

---

## Features

- ✅ Per-hour RFID attendance across 8 class periods per day
- ✅ Long-range wireless data transfer via LoRa SX1278 (no Wi-Fi dependency at TX end)
- ✅ Centralized Google Sheets logging with subject-wise attendance breakdown
- ✅ Automated SMS alerts to parents when attendance drops below 75%
- ✅ Receiver-initiated START handshake ensures synchronised daily operation
- ✅ Duplicate-scan prevention within the same class period
- ✅ Visual + buzzer feedback on card scan and period transitions
- ✅ Scalable — one TX unit per classroom, one common RX unit for the institution
- ✅ Regulated DC power via LM2596 buck converters for both units

---

## System Architecture

The system is divided into four layers:

```
┌──────────────────────────────────────────────────────────────────┐
│  SENSING LAYER        RFID-RC522 Reader (SPI)                    │
│                       Student RFID Tag Scanning (8 periods/day)  │
├──────────────────────────────────────────────────────────────────┤
│  PROCESSING LAYER     ESP32 TX — Attendance marking & LoRa TX    │
│                       ESP32 RX — LoRa RX, Wi-Fi, GSM control     │
├──────────────────────────────────────────────────────────────────┤
│  COMMUNICATION LAYER  LoRa SX1278 (UART @ 9600 bps)              │
│                       Long-range wireless packet transfer        │
├──────────────────────────────────────────────────────────────────┤
│  CLOUD & ALERT LAYER  Google Sheets via Apps Script Web App      │
│                       GSM SIM800L — Parental SMS alerts          │
└──────────────────────────────────────────────────────────────────┘
```

### Block Diagram

```
[CLASSROOM — TX UNIT]                    [ADMIN BLOCK — RX UNIT]
                                                             
  RFID-RC522 ──(SPI)──► ESP32 TX                            
                              │                              
                         SX1278 TX ═══(LoRa RF)═══► SX1278 RX ──(UART)──► ESP32 RX
                              ▲                                                  │
                         START signal ◄──────────────────────────────────────────┘
                                                                                 │
                                                         ┌───────────────────────┤
                                                         │                       │
                                                    Google Sheets          GSM SIM800L
                                                  (ATTENDANCE /          (SMS to parents
                                                  REPORT /              if < 75% attendance)
                                                  SUBJECT_REPORT)
                                                         
  LM2596 Buck Converter                         LM2596 Buck Converter
  (TX Unit Power Supply)                        (RX Unit Power Supply)
```

---

## Hardware Components

| Component | Model | Interface | Unit | Purpose |
|---|---|---|---|---|
| Microcontroller (TX) | ESP32 DevKit | — | TX | RFID reading, attendance logic, LoRa TX |
| Microcontroller (RX) | ESP32 DevKit | — | RX | LoRa RX, Wi-Fi HTTP, GSM control |
| RFID Reader | MFRC522 (RC522) | SPI | TX | Student card scanning |
| LoRa Module (TX) | SX1278 / Ra-02 | SPI | TX | Wireless attendance packet transmission |
| LoRa Module (RX) | SX1278 / Ra-02 | SPI | RX | Wireless attendance packet reception |
| GSM Module | SIM800L | UART1 (IO26/IO27) | RX | Parental SMS alerts |
| Buck Converter (TX) | LM2596 | — | TX | Regulated DC supply for TX unit |
| Buck Converter (RX) | LM2596 | — | RX | Regulated DC supply for RX unit |
| LED Indicator | 5 mm LED | GPIO | Both | Visual feedback |
| Buzzer | Active Buzzer | GPIO | TX | Audible card-scan and period-change alert |

---

## Software Tools

| Tool / Platform | Purpose |
|---|---|
| **Arduino IDE** | Firmware development and flashing for ESP32 TX and RX |
| **Arduino ESP32 Core** | ESP32 board support package |
| **MFRC522 Library** | RFID-RC522 driver for card UID reading |
| **Google Apps Script** | Web App deployed as HTTP endpoint for attendance logging |
| **Google Sheets** | Cloud spreadsheet for attendance storage and reporting |

---

## Circuit & Pin Connections

### RFID-RC522 → ESP32 TX (SPI)

| RC522 Pin | ESP32 TX Pin | Description |
|---|---|---|
| VCC | 3.3V | Power |
| GND | GND | Ground |
| SDA (SS) | IO5 | SPI Chip Select |
| SCK | IO18 | SPI Clock |
| MOSI | IO23 | SPI MOSI |
| MISO | IO19 | SPI MISO |
| RST | IO22 | Module Reset |

### LoRa SX1278 → ESP32 TX (UART2)

| SX1278 Pin | ESP32 TX Pin | Description |
|---|---|---|
| VCC | 3.3V | Power |
| GND | GND | Ground |
| TXD | IO16 (RX2) | LoRa data to ESP32 |
| RXD | IO17 (TX2) | Commands to LoRa |

### LED & Buzzer → ESP32 TX

| Component | ESP32 TX Pin | Description |
|---|---|---|
| LED (+) | IO2 | Card scan / period indicator |
| Buzzer (+) | IO4 | Audible alert output |
| Both (−) | GND | Ground |

### LoRa SX1278 → ESP32 RX (UART2)

| SX1278 Pin | ESP32 RX Pin | Description |
|---|---|---|
| VCC | 3.3V | Power |
| GND | GND | Ground |
| TXD | IO16 (RX2) | LoRa data to ESP32 |
| RXD | IO17 (TX2) | Commands to LoRa |

### GSM SIM800L → ESP32 RX (UART1)

| SIM800L Pin | ESP32 RX Pin | Description |
|---|---|---|
| VCC | Li-ion 3.7–4.2V | Power (via LM2596 output) |
| GND | GND | Ground |
| TXD | IO26 (RX1) | GSM data to ESP32 |
| RXD | IO27 (TX1) | AT commands to GSM |

### LED → ESP32 RX

| Component | ESP32 RX Pin | Description |
|---|---|---|
| LED (+) | IO2 | Wi-Fi status / data received indicator |
| LED (−) | GND | Ground |

### LM2596 Buck Converters — Power Supply

| LM2596 Unit | Input | Output | Supplies |
|---|---|---|---|
| TX Unit | Mains adapter / 12V | 5V regulated | ESP32 TX, RC522, SX1278, LED, Buzzer |
| RX Unit | Mains adapter / 12V | 5V / 4V split | ESP32 RX, SX1278 (3.3V), SIM800L (4V) |

---

## How It Works

1. **Power-On** — Both units power up. RX ESP32 connects to Wi-Fi and begins broadcasting `START` over LoRa every 2 seconds.
2. **Handshake** — TX ESP32 receives the `START` signal from RX via LoRa, confirms system start with a triple buzzer beep, and begins **Period 1**.
3. **RFID Scanning** — During each class period, students tap their RFID tags on the RC522 reader. The ESP32 reads the card UID, marks the student **Present**, and prevents duplicate scans within the same period using a `lastUID` guard.
4. **Period End & Data Transmission** — When the period timer expires, the TX ESP32 sends a LoRa packet for each registered student in the format:
   ```
   <UID>,<Present|Absent>,<ClassNumber>
   ```
   A triple beep signals the period transition.
5. **LoRa Reception** — RX ESP32 receives each packet, parses the UID, status, and class number.
6. **Google Sheets Logging** — RX ESP32 performs an HTTP GET to the Apps Script Web App URL:
   ```
   ?uid=<uid>&status=<status>&class=<classNum>
   ```
   The script maps the UID to roll number and name, appends a row to the `ATTENDANCE` sheet, and auto-updates the `REPORT` and `SUBJECT_REPORT` sheets.
7. **Subject Mapping** — Class numbers map to subjects as follows:

   | Period | Subject |
   |---|---|
   | 1 | PRP |
   | 2 | SS |
   | 3 | LIC |
   | 4 | TLW |
   | 5 | ACS |
   | 6 | DT |
   | 7 | PRP |
   | 8 | SS |

8. **End-of-Day SMS Alert** — After all 8 periods are transmitted, RX ESP32 waits 15 seconds, then queries the Apps Script (`?check=1`). The script returns a list of students with attendance below 75%. For each such student, the RX ESP32 sends an SMS via GSM SIM800L:
   ```
   Dear Parent, <Name>'s attendance is <X.XX>%. Please maintain above 75%.
   ```
9. **Daily Completion** — After period 8, the TX unit halts; a fresh power cycle begins the next day's session.

### TX Attendance Logic (simplified)

```cpp
// At end of each period:
for (int i = 0; i < 3; i++) {
    String status = present[i] ? "Present" : "Absent";
    sendData(studentUID[i], status);   // Send via LoRa
    present[i] = false;                // Reset for next period
}
currentClass++;
```

### RX Alert Logic (simplified)

```cpp
// After class 8, query Apps Script for defaulters:
String url = server + "?check=1";
// Response: "Name,Phone,Percent;Name,Phone,Percent;..."
// For each entry with percent < 75:
sendSMS(phone, "Dear Parent, " + name + "'s attendance is " + percent + "%...");
```

### Performance Metrics

| Parameter | Value |
|---|---|
| RFID Card Read Time | < 150 ms |
| LoRa Packet Transmission | < 1 second per student record |
| Google Sheets Update Latency | 1–3 seconds per HTTP request |
| SMS Dispatch Time | ~5 seconds per message |
| End-of-Day Report Generation | Automatic (Apps Script trigger) |

---

## Communication Protocols

### SPI (RFID-RC522)
- Full-duplex synchronous protocol used between ESP32 TX and RC522
- ESP32 acts as SPI master; RC522 is the slave
- Card UID read using the MFRC522 Arduino library via `rfid.PICC_ReadCardSerial()`

### UART (LoRa SX1278)
- Asynchronous serial at **9600 bps** on `HardwareSerial lora(2)` (UART2)
- TX ESP32 transmits CSV attendance records: `uid,status,class\n`
- RX ESP32 reads using `lora.readStringUntil('\n')` and parses with `indexOf(',')`
- RX ESP32 sends `START\n` to TX on startup until acknowledged

**LoRa Packet Format:**
```
1ed21407,Present,3\n
71bc1407,Absent,3\n
4e6b1407,Present,3\n
```

### UART (GSM SIM800L)
- Asynchronous serial at **9600 bps** on `HardwareSerial gsm(1)` (UART1)
- Controlled using AT commands from RX ESP32

**AT Command Sequence for SMS:**
```
AT                              // Module check
AT+CMGF=1                       // Set SMS text mode
AT+CMGS="+91XXXXXXXXXX"         // Set recipient number
> Dear Parent, <Name>'s...      // Message body
<CTRL+Z (0x1A)>                 // Send terminator
```

### HTTP (Google Sheets via Apps Script)
- RX ESP32 uses `WiFiClientSecure` + `HTTPClient` with `HTTPC_STRICT_FOLLOW_REDIRECTS`
- Attendance write endpoint: `GET /exec?uid=<>&status=<>&class=<>`
- Attendance check endpoint: `GET /exec?check=1`
- Apps Script maintains three sheets: `ATTENDANCE`, `REPORT`, `SUBJECT_REPORT`

**Google Apps Script Sheet Structure:**

| Sheet | Columns |
|---|---|
| `student` | UID, Roll, Name, Phone |
| `ATTENDANCE` | UID, Roll, Name, Subject, Status, Class, Time |
| `REPORT` | Name, Roll, Present, Total, Percentage |
| `SUBJECT_REPORT` | Roll, Name, PRP%, SS%, LIC%, TLW%, ACS%, DT% |

---

## Results

The system was successfully tested on hardware. Key outcomes:

- The RFID-RC522 correctly read all registered student tags with a response time under 150 ms, with LED and buzzer providing immediate feedback.
- LoRa SX1278 reliably transmitted attendance packets between the TX classroom unit and the centralized RX admin unit across the test range.
- The RX ESP32 successfully pushed all per-period attendance records to Google Sheets via the Apps Script Web App, with real-time updates visible in the `REPORT` and `SUBJECT_REPORT` sheets.
- The end-of-day SMS alert mechanism correctly identified students below 75% attendance and dispatched personalised messages to parent phone numbers via GSM SIM800L.
- Duplicate-scan prevention ensured a student could only be marked present once per period regardless of repeated tag taps.
- Both TX and RX units operated stably on regulated 5V supplied by the LM2596 buck converters.

---

## Future Scope

- **Multi-classroom scaling** — Add unique classroom IDs to LoRa packets to support multiple TX units on a single RX node
- **OLED / LCD display** — Add a local display to the TX unit to show present/absent count per period in real time
- **LoRa mesh networking** — Enable multi-hop relay for larger campus deployments beyond direct LoRa range
- **Mobile dashboard** — Real-time attendance dashboard app for faculty and administration
- **Biometric integration** — Combine RFID with fingerprint sensor to prevent proxy attendance
- **Cloud backup** — Mirror Google Sheets data to Firebase or AWS DynamoDB for redundancy
- **Battery backup** — UPS module for uninterrupted operation during power cuts
- **OTA firmware update** — Enable remote firmware updates for ESP32 TX units across all classrooms

---

> *Built to automate student attendance and keep parents informed — powered by LoRa, ESP32, and Google Sheets.*
