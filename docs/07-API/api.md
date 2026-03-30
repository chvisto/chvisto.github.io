---

# API — HMI Subsystem

**Name:** Christo Jomon Joseph
**Team:** 305
**Board ID:** `0x43` ('C')

> **Note:** Teammate board IDs below are my own assignments. They have not been confirmed by all teammates yet and will be updated before integration.

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

## Teammate IDs (Unconfirmed)

| Name | Subsystem | Board ID |
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

**To:** Liam (`0x4C`)

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

### Type 8 / 10 / 11 — Environmental Data

**From:** Isaiah (`0x49`)

Isaiah's subsystem sends three separate message types from a BME680 sensor. I display temperature and humidity on the OLED. Pressure is forwarded but not displayed.

#### Type 8 — Temperature

| | Byte 1-2 | Byte 3-4 |
|---|---|---|
| **Variable Name** | `message_type` | `temperature` |
| **C Type** | `uint16_t` | `int16_t` |
| **Min** | `8` | `-4000` |
| **Max** | `8` | `8500` |
| **Example** | `8` | `2400` |

Scaled × 100 (°C × 100). Divide by 100.0 to display. Range: -40.00 to 85.00 °C.

#### Type 10 — Barometric Pressure

| | Byte 1-2 | Byte 3-6 | Byte 7-10 |
|---|---|---|---|
| **Variable Name** | `message_type` | `pressure` | `altitude` |
| **C Type** | `uint16_t` | `uint32_t` | `int32_t` |
| **Min** | `10` | `30000` | `-50000` |
| **Max** | `10` | `110000` | `100000` |
| **Example** | `10` | `101325` | `150` |

Pressure in Pa. Altitude in cm. Forwarded but not rendered on OLED.

#### Type 11 — Humidity

| | Byte 1-2 | Byte 3-4 |
|---|---|---|
| **Variable Name** | `message_type` | `humidity` |
| **C Type** | `uint16_t` | `uint16_t` |
| **Min** | `11` | `0` |
| **Max** | `11` | `10000` |
| **Example** | `11` | `5500` |

Scaled × 100 (% × 100). Divide by 100.0 to display. Shown on the Values page of the OLED.

---

### Type 3 / 4 — Camera Data

**From:** Arianna (`0x41`)

Arianna's subsystem sends two message types. Type 4 (camera status) is displayed on the OLED. Type 3 (image frame chunks) is received and forwarded but not rendered.

#### Type 4 — Camera Status Report

| | Byte 1-2 | Byte 3 | Byte 4-5 | Byte 6-7 | Byte 8 |
|---|---|---|---|---|---|
| **Variable Name** | `message_type` | `camera_state` | `frame_width` | `frame_height` | `error_code` |
| **C Type** | `uint16_t` | `uint8_t` | `uint16_t` | `uint16_t` | `uint8_t` |
| **Min** | `4` | `0` | `0` | `0` | `0` |
| **Max** | `4` | `2` | `1920` | `1080` | `2` |
| **Example** | `4` | `2` | `640` | `480` | `0` |

`camera_state`: 0 = Off, 1 = Idle, 2 = Capturing. `error_code`: 0 = No error, 1 = Camera not detected, 2 = Unknown error. State and error code display on the Values page.

#### Type 3 — Camera Frame Data Packet (forward only)

| | Byte 1-2 | Byte 3-4 | Byte 5-6 | Byte 7-8 | Byte 9-58 |
|---|---|---|---|---|---|
| **Variable Name** | `message_type` | `frame_id` | `packet_index` | `total_packets` | `image_data_chunk` |
| **C Type** | `uint16_t` | `uint16_t` | `uint16_t` | `uint16_t` | `uint8_t[50]` |
| **Min** | `3` | `0` | `0` | `1` | `0` |
| **Max** | `3` | `65535` | `65535` | `65535` | `255` |
| **Example** | `3` | `25` | `1` | `10` | `45` |

Each frame is split across multiple 64-byte packets. My subsystem forwards these but does not reconstruct or render image data.

---

### Type 4 — Gyroscope Data

**From:** Ragul (`0x52`)

| | Byte 1 | Byte 2-3 | Byte 4-5 | Byte 6-7 |
|---|---|---|---|---|
| **Variable Name** | `message_type` | `gyro_x` | `gyro_y` | `gyro_z` |
| **C Type** | `uint8_t` | `int16_t` | `int16_t` | `int16_t` |
| **Min** | `4` | `-100` | `-100` | `-100` |
| **Max** | `4` | `100` | `100` | `100` |
| **Example** | `4` | `12` | `-5` | `3` |

Values are pre-scaled from raw LSM9DS1 counts to -100 to 100 by Ragul's firmware. Each axis is a 2-byte little-endian `int16_t`. All three axes display on the Gyro Data page of the OLED.

---

### Type 12 — Distance Reading

**From:** Myles (`0x4D`)

| | Byte 1-2 | Byte 3-6 |
|---|---|---|
| **Variable Name** | `message_type` | `distance_m` |
| **C Type** | `uint16_t` | `float` |
| **Min** | `12` | `0.00` |
| **Max** | `12` | `6.00` |
| **Example** | `12` | `0.87` |

Distance in meters as a 4-byte IEEE 754 float. Myles sends this message only when distance is at or below 0.50 m. Multiply by 100.0 to display centimeters on the OLED Values page.

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
