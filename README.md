# 🔵 BlueLink — Bluetooth Messenger

![License](https://img.shields.io/badge/license-MIT-blue.svg)
![Platform](https://img.shields.io/badge/platform-Web%20%7C%20Android-brightgreen)
![Bluetooth](https://img.shields.io/badge/Bluetooth-BLE-blue)
![Course](https://img.shields.io/badge/SZABIST-CNDC%20BSCS--6B-orange)

A **WhatsApp-style** real-time chat application that connects two devices over **Bluetooth Low Energy (BLE)** using the Web Bluetooth API. Supports text messaging, file transfer with progress, delivery receipts, and timestamps — all in a single HTML file with zero dependencies.

---

## 📱 Live Demo

> **[https://dushantraja0.github.io/bluetooth-chat/](https://dushantraja0.github.io/bluetooth-chat/)**
> *(Replace YOUR-USERNAME with your GitHub username after deployment)*

---

## ✨ Features

| Feature | Status |
|---|---|
| 📡 Device Discovery & Scanning | ✅ |
| 💬 Real-time Text Messaging | ✅ |
| 📁 File Transfer with Progress Bar | ✅ |
| 🟢 Connection Status Indicators | ✅ |
| 🕐 Message Timestamps | ✅ |
| ✓✓ Delivery Receipts (Bonus) | ✅ |

---

## 🛠 Technology

**Option A — Web (Browser)**

| Stack | Details |
|---|---|
| Language | Vanilla HTML, CSS, JavaScript |
| Bluetooth API | Web Bluetooth API (BLE / GATT) |
| Browser Support | Chrome 56+, Edge 79+ |
| Platform | Desktop (Windows/Mac/Linux), Android |
| Dependencies | None — single file app |

---

## 📋 Requirements

- **Chrome** or **Edge** browser (latest version)
- **Windows / Mac / Linux / Android** device
- Bluetooth must be enabled on both devices
- Both devices must be within ~10 meters of each other
- ⚠️ **iOS (Safari) is NOT supported** — Web Bluetooth is blocked by Apple

---

## 🚀 Running the App

### Method 1 — GitHub Pages (Recommended for real BLE)

1. Fork or clone this repository
2. Go to **Settings → Pages → Branch: main → Save**
3. Open the link on both devices in Chrome:
```
https://YOUR-USERNAME.github.io/bluetooth-chat
```

### Method 2 — Local Server

**Python (recommended):**
```bash
git clone https://github.com/YOUR-USERNAME/bluetooth-chat.git
cd bluetooth-chat
python -m http.server 8080
```
Then open `http://localhost:8080` in Chrome.

**Node.js:**
```bash
npx serve .
```

### Method 3 — Two Devices on same Wi-Fi

```bash
# On your laptop, run the server
python -m http.server 8080

# Find your laptop's IP
# Windows: ipconfig -> look for IPv4 Address
# Mac/Linux: ifconfig

# On second device (Android phone), open Chrome:
http://192.168.1.YOUR_IP:8080
```

---

## 📖 How to Use

### Connecting Two Devices

1. Open the app on **Device A** and **Device B** (both in Chrome)
2. On **Device A** tap **"Scan for Devices"**
3. Chrome shows a Bluetooth picker — select **Device B**
4. Both devices enter the Chat screen automatically

### Sending Messages

- Type in the bottom input box
- Press **Enter** or tap the send button
- Your messages appear on the **right (blue)**
- Received messages appear on the **left (gray)**
- **✓** = sent, **✓✓ green** = delivered to peer

### Sending Files

1. Tap the **📎** attachment button
2. Pick any file (image, PDF, document, etc.)
3. A progress bar shows real-time transfer progress
4. File appears as a bubble with name and size when complete

### Demo Mode

If running on `localhost` (no HTTPS) or an unsupported browser, the app automatically enters **Demo Mode** with simulated devices and replies — perfect for UI testing and screenshots.

---

## 🏗 Project Structure

```
bluetooth-chat/
├── index.html        # Complete app (single file — HTML + CSS + JS)
├── README.md         # This documentation
├── PROTOCOL.md       # Byte-level protocol design document
├── .gitignore        # Git ignore rules
└── LICENSE           # MIT License
```

---

## 🔧 BLE Service Architecture

```
Primary Service UUID: 12345678-1234-1234-1234-123456789abc
│
├── Message Characteristic  UUID: ...abd  [Write | Notify]
│     Text messages, FILE_META frames
│
├── File Characteristic     UUID: ...abe  [Write Without Response]
│     FILE_CHUNK frames (512 bytes each)
│
└── ACK Characteristic      UUID: ...abf  [Notify]
      Delivery receipt frames
```

---

## 📡 Protocol Overview

All frames are UTF-8 JSON with a 1-byte type tag prefix:

| Frame | Tag | Description |
|---|---|---|
| TEXT | `0x01` | Text message with ID, timestamp, body |
| FILE_META | `0x02` | File name, size, chunk count |
| FILE_CHUNK | `0x03` | 512-byte binary data block |
| ACK | `0x04` | Delivery receipt |

See [PROTOCOL.md](PROTOCOL.md) for full byte-level specification.

---

## ⚠️ Known Limitations

- Web Bluetooth requires a **user gesture** to initiate scanning (browser security)
- Maximum BLE MTU: **512 bytes** — larger files are automatically chunked
- One active connection per browser tab
- iOS Safari does not support Web Bluetooth — use React Native for iPhones

---

## 🧪 Testing

| Scenario | How to Test |
|---|---|
| UI and Demo | Open `localhost:8080`, use Demo Mode |
| Real BLE | Deploy to GitHub Pages, open on 2 Android/PC devices |
| File Transfer | Attach any image or PDF via 📎 button |
| Delivery Receipts | Send a message, watch tick turn to double tick |

---

## 📄 Assignment Details

| Field | Details |
|---|---|
| Course | CNDC — Computer Networks & Data Communications |
| Section | BSCS-6B |
| Institute | SZABIST Islamabad |
| Assignment | #3 — Bluetooth Messaging App |
| Option Chosen | A — Web Browser |

---

## 📝 License

This project is licensed under the **MIT License** — see [LICENSE](LICENSE) for details.
