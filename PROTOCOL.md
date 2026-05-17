# BlueLink — Protocol Design Document

| Field | Details |
|---|---|
| Course | CNDC — Computer Networks & Data Communications |
| Section | BSCS-6B, SZABIST Islamabad |
| Assignment | #3 — Bluetooth Messaging App |
| Technology | Bluetooth Low Energy (BLE) via Web Bluetooth API |
| Version | 1.0 |

---

## 1. Overview

BlueLink uses **BLE GATT** (Generic Attribute Profile) for all communication. Two devices connect as GATT Client (initiator) and GATT Server (responder). All data is split across three dedicated GATT Characteristics — one for messages, one for file chunks, and one for delivery receipts.

---

## 2. Service & Characteristic Layout

```
Primary Service
UUID: 12345678-1234-1234-1234-123456789abc
│
├── Message Characteristic
│   UUID: 12345678-1234-1234-1234-123456789abd
│   Properties: WRITE | NOTIFY
│   Max length: 512 bytes
│   Carries: TEXT frames, FILE_META frames
│
├── File Characteristic
│   UUID: 12345678-1234-1234-1234-123456789abe
│   Properties: WRITE WITHOUT RESPONSE
│   Max length: 523 bytes
│   Carries: FILE_CHUNK frames
│
└── ACK Characteristic
    UUID: 12345678-1234-1234-1234-123456789abf
    Properties: NOTIFY
    Max length: 128 bytes
    Carries: ACK frames
```

---

## 3. Frame Structure

Every frame starts with a **1-byte type tag** followed by a **UTF-8 JSON payload**. This allows fast frame-type identification without parsing the full JSON.

```
+----------+------------------------------------------+
| Byte  0  |  Bytes 1 to N                            |
| Type Tag |  UTF-8 JSON Payload                      |
+----------+------------------------------------------+
```

### Type Tag Table

| Tag (hex) | Decimal | Frame Name | Characteristic |
|-----------|---------|------------|----------------|
| `0x01`    | 1       | TEXT       | Message        |
| `0x02`    | 2       | FILE_META  | Message        |
| `0x03`    | 3       | FILE_CHUNK | File           |
| `0x04`    | 4       | ACK        | ACK            |

---

## 4. Frame Definitions

### 4.1 TEXT Frame (Tag: 0x01)

Sent when user sends a text message. Written to the **Message Characteristic**.

```
Byte 0:   0x01

Bytes 1+: JSON {
  "id":   uint32   — message ID, increments per sender (1, 2, 3...)
  "ts":   uint64   — Unix timestamp in milliseconds
  "text": string   — message body, UTF-8, max ~480 chars
}
```

**Example:**
```
Raw hex:  01 7B 22 69 64 22 3A 31 2C 22 74 73 22 3A ...
Decoded:  [0x01] {"id":1,"ts":1748000000000,"text":"Hello!"}
```

---

### 4.2 FILE_META Frame (Tag: 0x02)

Sent before any file chunks, to tell the receiver what file is coming. Written to the **Message Characteristic**.

```
Byte 0:   0x02

Bytes 1+: JSON {
  "fid":    uint32   — file session ID (unique per transfer)
  "name":   string   — original filename e.g. "photo.jpg"
  "size":   uint32   — total file size in bytes
  "chunks": uint16   — total number of chunks (ceil(size / 512))
  "mime":   string   — MIME type e.g. "image/jpeg"
  "ts":     uint64   — timestamp
}
```

**Example:**
```
[0x02] {"fid":1,"name":"photo.jpg","size":76800,"chunks":150,"mime":"image/jpeg","ts":1748000000000}
```

---

### 4.3 FILE_CHUNK Frame (Tag: 0x03)

Carries one 512-byte block of raw file data. Written to the **File Characteristic** (Write Without Response for speed). Uses a **binary header** instead of JSON for efficiency.

```
Byte 0:      0x03            — type tag
Bytes 1-4:   fid             — uint32, big-endian — matches FILE_META fid
Bytes 5-8:   idx             — uint32, big-endian — zero-based chunk index
Bytes 9-10:  len             — uint16, big-endian — actual data bytes in this chunk
Bytes 11+:   data            — raw binary file content (max 512 bytes)
```

**Total max frame size:**
```
1 (tag) + 4 (fid) + 4 (idx) + 2 (len) + 512 (data) = 523 bytes
```

This fits within the BLE ATT payload limit of ~512 bytes after ATT headers on most platforms.

**Reassembly logic (receiver side):**
```
1. Allocate buffer: new Uint8Array(file.size)
2. For each chunk: buffer.set(chunkData, idx * 512)
3. When received == total chunks: file is complete
4. Send ACK frame with fid
```

---

### 4.4 ACK Frame (Tag: 0x04)

Sent by the receiver after successfully receiving a TEXT message or a complete file. Notified via the **ACK Characteristic**.

```
Byte 0:   0x04

Bytes 1+: JSON {
  "id":   uint32         — echoes TEXT frame id, or fid for files
  "type": "msg" | "file" — what is being acknowledged
  "ts":   uint64         — receipt timestamp
}
```

**Example (message ack):**
```
[0x04] {"id":5,"type":"msg","ts":1748000001234}
```

---

## 5. Message Flow Diagrams

### 5.1 Text Message with Delivery Receipt

```
Device A (Sender)                        Device B (Receiver)
       |                                          |
       |--- Write TEXT(id=5, text="Hi") -------->|
       |                                          | display message
       |                                          | send ACK
       |<-- Notify ACK(id=5, type="msg") --------|
       |                                          |
  mark ✓✓ green
```

### 5.2 File Transfer

```
Device A (Sender)                        Device B (Receiver)
       |                                          |
       |--- Write FILE_META(fid=1) ------------->| allocate buffer
       |--- Write CHUNK(fid=1, idx=0) ---------->| store @ offset 0
       |--- Write CHUNK(fid=1, idx=1) ---------->| store @ offset 512
       |--- Write CHUNK(fid=1, idx=2) ---------->| store @ offset 1024
       |          ... N chunks ...               |
       |--- Write CHUNK(fid=1, idx=N) ---------->| reassemble complete
       |                                          | show file bubble
       |<-- Notify ACK(fid=1, type="file") ------|
       |                                          |
  show ✓✓ green
```

---

## 6. Chunk Size Rationale

| Platform | Typical BLE MTU | ATT Payload | Our Chunk Size |
|---|---|---|---|
| Android | 517 bytes | 512 bytes | 512 bytes ✅ |
| Windows | 247 bytes | 244 bytes | 512 bytes (split by stack) |
| iOS | 185 bytes | 182 bytes | Not supported |

We set `CHUNK_SIZE = 512` because:
- It fits within Android's common 517-byte MTU
- The OS BLE stack automatically fragments larger writes on Windows
- It balances throughput vs. overhead (fewer chunks = fewer round trips)

---

## 7. Error Handling

| Error Condition | Handling Strategy |
|---|---|
| Missing chunk | Future: NACK frame (`0x05`) with missing `idx`; sender retransmits |
| Connection drop mid-transfer | Transfer discarded; UI shows "Transfer interrupted" |
| Duplicate chunk | Receiver checks `idx` in received set; silently ignores duplicates |
| Text too long (>480 chars) | Sender splits into multiple TEXT frames with sequence numbers |
| Write failure | Retry up to 3 times with 200ms backoff, then surface error to UI |

---

## 8. Security Considerations

| Layer | Mechanism |
|---|---|
| BLE Pairing | BLE Secure Connections (LESC) with bonding for eavesdrop protection |
| App Layer | AES-GCM encryption with shared key derived from QR code scan (future) |
| Data Integrity | BLE CRC-24 on every packet (hardware level, automatic) |

> For this assignment, standard BLE pairing is used. Application-layer encryption can be added in a future version by deriving a shared AES key via a QR code exchange before the Bluetooth connection is made.

---

## 9. UUID Assignment Rationale

We use a custom 128-bit UUID base (`12345678-1234-1234-1234-123456789ab_`) with the last byte differentiating each characteristic. This avoids conflicts with standard Bluetooth SIG services and makes the app discoverable only by other BlueLink instances during scanning.
