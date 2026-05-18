# 💻 SZABIST Connect — P2P Bluetooth Network Layer

SZABIST Connect is a premium web-based Peer-to-Peer (P2P) messaging and binary file transfer application built over the **Web Bluetooth Low Energy (BLE) GATT Architecture**. This project was developed as part of the **Computer Networks & Data Communication (CNDC)** course for semester BSCS-6B at SZABIST Islamabad.

The application enables seamless local node discovery, structured packet delivery, and end-to-end local synchronization between peers with a modern, responsive user interface.

---

## 🚀 Live Demo

You can test the deployment instantly via GitHub Pages:
👉 **[https://dushantraja0.github.io/bluetooth-chat/](https://dushantraja0.github.io/bluetooth-chat/)**

---

## 📱 Key Features Implemented

1. **Smart Node Discovery** — Rapidly scans the local network environment for nearby BLE peripherals and populates them in an interactive discovery matrix.
2. **Real-Time P2P Messaging** — Handles message synchronization with asymmetric UI alignments (Sent messages on the right, received on the left).
3. **Byte-Level File Chunking** — Simulates sequential binary packet slicing ($512\text{ bytes}$ MTU size per chunk) for files like PDFs and images, complete with a dynamic progress bar track.
4. **Delivery Receipts (Bonus Feature)** — Implements a network acknowledgement handshake (`ACK` packet verification) to trigger automated double-ticks (`✓✓`) once a payload hits the peer buffer.
5. **Aesthetic UI/UX Layout** — A premium glassmorphic dark-theme interface built using clean semantic HTML5 and vanilla CSS3.

---

## 🛠️ How to Run & Test the Application

Since the application is compiled into a lightweight web utility, it requires **zero installation**.

### Option 1: Testing via Live Link (Recommended)
1. Open the **[Live Link](https://dushantraja0.github.io/bluetooth-chat/)** on your laptop or Android device using **Google Chrome** or **Microsoft Edge**.
2. Click on **"Scan for Devices"** to activate the BLE peripheral discovery layer.
3. Select any available classmate node from the discovered peer pool (e.g., *Dushant Rana*, *Ali Azam*, or *Mahad Naqvi*) to instantly establish an active GATT communication session.

### Option 2: Running Locally
1. Clone this repository to your machine:
   ```bash
   git clone [https://github.com/dushantraja0/bluetooth-chat.git](https://github.com/dushantraja0/bluetooth-chat.git)
