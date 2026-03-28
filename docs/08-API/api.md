---

# API — HMI Subsystem

**Name:** Christo Jomon Joseph
**Team:** 305
**Board ID:** `0x43` ('C')

> **Note:** Teammate board IDs below are temporary placeholders. My team has not finalized individual IDs yet. These will be updated once confirmed with each subsystem owner.

---

## Overview

This documents the UART message interface for my HMI subsystem on the EGR 314 daisy-chain network. My board is an ESP32-S3-WROOM-1-N4 running MicroPython v1.23.0, driving a 128x64 SSD1306 OLED over I2C. The HMI sits at one node in a six-board ring. It receives sensor data from four subsystems and sends motor speed commands to one.

---

## Protocol

| Field | Value |
|---|---|
| Baud rate | 9600, 8N1 |
| Packet size | 64 bytes fixed |
| Prefix | `0x41 0x5A` |
| Suffix | `0x59 0x42` |
| Byte 2 | Sender ID |
| Byte 3 | Receiver ID |
| Bytes 4-61 | Payload (first payload byte = message type) |
| Broadcast | `0x58` |

The message tables below cover only the inner payload bytes (4-61). Prefix, suffix, sender ID, and receiver ID are not listed since they are the same across every subsystem.

---

## Teammate IDs (Placeholders)

These IDs are assigned by me for now. Actual IDs will be coordinated with teammates before integration.

| Name | Subsystem | Board ID (placeholder) |
|---|---|---|
| Christo | HMI (me) | `0x43` |
| Liam | Motor | `0x4C` |
| Isaiah | Environmental Sensor | `0x49` |
| Arianna | Camera | `0x41` |
| Myles | Distance Sensor | `0x4D` |
| Ragul | Gyroscope | `0x52` |
| — | Broadcast | `0x58` |

---

## Messages I Send

### `0x01` — Motor Speed Command

**To:** Liam (`0x4C`, placeholder)

| | Byte 1 | Byte 2 |
|---|---|---|
| **Variable Name** | `message_type` | `motor_speed` |
| **C Type** | `uint8_t` | `int8_t` |
| **Min** | `1` | `-100` |
| **Max** | `1` | `100` |
| **Example** | `1` | `-30` |

Positive values mean forward, negative means reverse. Zero is not transmitted. The sender rate-limits this message to once per 500 ms using a non-blocking timer check.

---

## Messages I Receive

### `0x02` — Environmental Data

**From:** Isaiah (`0x49`, placeholder)

| | Byte 1 | Byte 2 | Byte 3 |
|---|---|---|---|
| **Variable Name** | `message_type` | `temp` | `humidity` |
| **C Type** | `uint8_t` | `uint8_t` | `uint8_t` |
| **Min** | `2` | `0` | `0` |
| **Max** | `2` | `100` | `100` |
| **Example** | `2` | `24` | `55` |

Temperature in degrees Celsius, humidity as percent relative humidity. Both values display on the Values page of the OLED.

---

### `0x03` — Camera Status

**From:** Arianna (`0x41`, placeholder)

| | Byte 1 | Byte 2 | Byte 3 |
|---|---|---|---|
| **Variable Name** | `message_type` | `motion` | `object_code` |
| **C Type** | `uint8_t` | `uint8_t` | `uint8_t` |
| **Min** | `3` | `0` | `0` |
| **Max** | `3` | `1` | `255` |
| **Example** | `3` | `1` | `4` |

`motion` is a boolean: 0 means nothing detected, 1 means motion present. `object_code` is a classification value from the camera subsystem. Both display on the Values page.

---

### `0x04` — Gyroscope Data

**From:** Ragul (`0x52`, placeholder)

| | Byte 1 | Byte 2 | Byte 3 | Byte 4 |
|---|---|---|---|---|
| **Variable Name** | `message_type` | `gyro_x` | `gyro_y` | `gyro_z` |
| **C Type** | `uint8_t` | `int8_t` | `int8_t` | `int8_t` |
| **Min** | `4` | `-100` | `-100` | `-100` |
| **Max** | `4` | `100` | `100` | `100` |
| **Example** | `4` | `12` | `-5` | `3` |

Signed bytes sent as two's complement. Decode with: `x if x < 128 else x - 256`. All three axes display on the Gyro Data page of the OLED.

---

### `0x05` — Distance Reading

**From:** Myles (`0x4D`, placeholder)

| | Byte 1 | Byte 2 |
|---|---|---|
| **Variable Name** | `message_type` | `distance_cm` |
| **C Type** | `uint8_t` | `uint8_t` |
| **Min** | `5` | `0` |
| **Max** | `5` | `200` |
| **Example** | `5` | `87` |

Distance in centimeters from the ultrasonic sensor. Displays on the Values page.

---

## ACK

Every valid packet addressed to me gets an immediate acknowledgement back to the sender:

| | Byte 1 | Byte 2 |
|---|---|---|
| **Variable Name** | `message_type` | `received_type` |
| **C Type** | `uint8_t` | `uint8_t` |
| **Min** | `170` | `1` |
| **Max** | `170` | `5` |
| **Example** | `170` | `2` |

`0xAA` (170) is reserved for ACK only. Byte 2 echoes back whichever message type triggered it. Broadcast packets do not get an ACK.

---

## Routing Logic

| Condition | Action |
|---|---|
| Receiver == `0x43` (me) | Process, send ACK, do not forward |
| Receiver == `0x58` (broadcast) | Process and forward |
| Sender == `0x43` (looped back) | Discard |
| Receiver == someone else | Forward immediately |
| Malformed packet | Discard |

---

## Byte Stuffing

`build_packet()` subtracts 1 from any payload byte that matches a reserved byte (`0x41`, `0x5A`, `0x59`, `0x42`). This prevents false prefix or suffix matches inside the payload. Any subsystem decoding my messages needs to account for this.

---

## Source Code

[Download HMI Subsystem Code (ZIP)](./HMI_subsystem_code.zip)

MicroPython v1.23.0 on ESP32-S3-WROOM-1-N4.
