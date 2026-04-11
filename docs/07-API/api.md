---

# API — HMI Subsystem

**Name:** Christo Jomon Joseph
**Team:** 305
**Board ID:** `0x43` ('C')

---

## Overview

My HMI subsystem is the user-facing node on Team 305's daisy-chain bus. The board is an ESP32-S3-WROOM-1-N4 running MicroPython v1.23.0, paired with a OLED over I2C. This subsystem pulls in sensor and system data from teammates, pushes motor control commands to Liam's motor board and forwards everything else. The OLED displays live readings organized across named pages.

---

## Protocol

| Field | Value |
|---|---|
| Baud rate | 9600, 8N1, no parity, no flow control |
| Packet size | 64 bytes fixed |
| Start bits | `0xA5 0x5A` (bytes 0-1) |
| Stop bits | `0x59 0x42` (bytes 62-63) |
| Byte 2 | Sender ID |
| Byte 3 | Receiver ID |
| Bytes 4-61 | Payload (payload byte 1 = packet byte 4) |
| Broadcast | `0x58` ('X') |

**Clarification:** Tables below use payload-relative byte numbering. Payload Byte 1 maps to packet byte 4, Payload Byte 2 maps to packet byte 5, and so on through Payload Byte 58 at packet byte 61. The prefix, suffix, sender ID and receiver ID fields are the same on every node and are left out of the tables.

Payload bytes must not reproduce the start bit pattern (`0xA5 0x5A`) or stop bit pattern (`0x59 0x42`). Byte stuffing in `build_packet()` handles this automatically. See the Byte Stuffing section for the mechanism.

---

## Teammate IDs

Each node ID is the ASCII hex encoding of the person's first name initial. This keeps addresses human-readable in a serial monitor without any lookup table.

| Name | Subsystem | Board ID | Initial |
|---|---|---|---|
| **Christo** | **HMI** | **`0x43`** | **C** |
| Liam | Motor | `0x4C` | L |
| Isaiah | Environmental Sensor | `0x49` | I |
| Arianna | Camera | `0x41` | A |
| Myles | Distance Sensor | `0x4D` | M |
| Ragul | Gyroscope | `0x52` | R |
| Damian | MQTT / Wireless | `0x44` | D |
| — | Broadcast / Everyone | `0x58` | X |

---

## Receive State Machine

Every byte off the UART runs through three states before any routing decision happens. The receiver does not reset on bad bytes : it waits. Junk before the start pattern is silently ignored. A false start (0xA5 not followed by 0x5A) drops back to WAIT_P1 without discarding the second byte if it's another 0xA5.

```
WAIT_P1  ->  byte == 0xA5?  yes -> WAIT_P2,  no -> ignore junk
WAIT_P2  ->  byte == 0x5A?  yes -> COLLECT,  no -> back to WAIT_P1
COLLECT  ->  accumulate 64 bytes, then VALIDATE
VALIDATE ->  stop bits present? sender in loop? receiver in loop? loopback?
             all pass -> ROUTE,  any fail -> discard, back to WAIT_P1
ROUTE    ->  receiver == me:    handle + ACK, do NOT forward
             receiver == 0x58:  handle AND forward downstream
             receiver == other: forward only, do NOT handle
```

RX drains completely before any outbound packet goes out. Packets with bad start/stop bits or unknown sender IDs are dropped immediately.

---

## Messages I Send

### Type 1 — Motor Control State Report

**To:** Liam (`0x4C`)

| Payload Byte | 1-2 | 3-4 | 5-6 | 7 | 8 |
|---|---|---|---|---|---|
| **Variable Name** | `message_type` | `left_motor_speed` | `right_motor_speed` | `left_motor_dir` | `right_motor_dir` |
| **C Type** | `uint16_t` | `int16_t` | `int16_t` | `uint8_t` | `uint8_t` |
| **Min** | `1` | `-100` | `-100` | `0` | `0` |
| **Max** | `1` | `100` | `100` | `1` | `1` |
| **Example** | `1` | `-30` | `-30` | `0` | `0` |

`left_motor_dir` / `right_motor_dir`: 0 = Reverse, 1 = Forward. Speed is PWM percent from -100 to 100. Zero is not transmitted : the motor subsystem holds its last commanded state. A non-blocking timer enforces the 500 ms rate limit.

---

## Messages I Receive

All multi-byte fields are little-endian. The message type occupies payload bytes 1-2 as a `uint16_t` on every incoming message type.

### Type 8 — Temperature Sensor Data Report

**From:** Isaiah (`0x49`)

| Payload Byte | 1-2 | 3-6 |
|---|---|---|
| **Variable Name** | `message_type` | `temperature` |
| **C Type** | `uint16_t` | `float` |
| **Min** | `8` | `-40.0` |
| **Max** | `8` | `85.0` |
| **Example** | `8` | `24.0` |

IEEE 754 little-endian float, degrees Celsius. Shown on the Values page of the OLED.

---

### Type 10 — Barometric Pressure Sensor Data Report

**From:** Isaiah (`0x49`)

| Payload Byte | 1-2 | 3-6 | 7-10 |
|---|---|---|---|
| **Variable Name** | `message_type` | `pressure` | `altitude` |
| **C Type** | `uint16_t` | `float` | `float` |
| **Min** | `10` | `300.0` | `-500.0` |
| **Max** | `10` | `1100.0` | `9000.0` |
| **Example** | `10` | `1013.25` | `350.0` |

Pressure in hPa, altitude in meters, both as floats. All three environmental readings (temperature, pressure, altitude) render on the OLED.

---

### Type 11 — Humidity Sensor Data Report

**From:** Isaiah (`0x49`)

| Payload Byte | 1-2 | 3-6 |
|---|---|---|
| **Variable Name** | `message_type` | `humidity` |
| **C Type** | `uint16_t` | `float` |
| **Min** | `11` | `0.0` |
| **Max** | `11` | `100.0` |
| **Example** | `11` | `55.0` |

Relative humidity as a float percentage. Shown on the Values page alongside temperature.

---

### Type 4 — Camera Status Report

**From:** Arianna (`0x41`)

| Payload Byte | 1-2 | 3 | 4-5 | 6-7 | 8 |
|---|---|---|---|---|---|
| **Variable Name** | `message_type` | `camera_state` | `frame_width` | `frame_height` | `error_code` |
| **C Type** | `uint16_t` | `uint8_t` | `uint16_t` | `uint16_t` | `uint8_t` |
| **Min** | `4` | `0` | `0` | `0` | `0` |
| **Max** | `4` | `2` | `1920` | `1080` | `2` |
| **Example** | `4` | `2` | `640` | `480` | `0` |

`camera_state`: 0 = Off, 1 = Idle, 2 = Capturing. `error_code`: 0 = no error, 1 = camera not detected, 2 = unknown error. State and error code both render on the Values page.

---

### Type 3 — Camera Frame Data Packet (forward only)

**From:** Arianna (`0x41`)

| Payload Byte | 1-2 | 3-4 | 5-6 | 7-8 | 9-58 |
|---|---|---|---|---|---|
| **Variable Name** | `message_type` | `frame_id` | `packet_index` | `total_packets` | `image_data_chunk` |
| **C Type** | `uint16_t` | `uint16_t` | `uint16_t` | `uint16_t` | `uint8_t[50]` |
| **Min** | `3` | `0` | `0` | `1` | `0` |
| **Max** | `3` | `65535` | `65535` | `65535` | `255` |
| **Example** | `3` | `25` | `1` | `10` | `—` |

Arianna chunks each frame across multiple 64-byte packets. This subsystem forwards them immediately without buffering or attempting to reconstruct the image.

---

### Type 5 — Gyroscope Data Report

**From:** Ragul (`0x52`)

| Payload Byte | 1-2 | 3-6 | 7-10 | 11-14 |
|---|---|---|---|---|
| **Variable Name** | `message_type` | `angular_vel_x` | `angular_vel_y` | `angular_vel_z` |
| **C Type** | `uint16_t` | `float` | `float` | `float` |
| **Min** | `5` | `-34.9` | `-34.9` | `-34.9` |
| **Max** | `5` | `34.9` | `34.9` | `34.9` |
| **Example** | `5` | `0.12` | `-0.05` | `0.03` |

Three-axis angular velocity from Ragul's LSM9DS1, in rad/s. Each axis is a 4-byte IEEE 754 little-endian float. All three values render on the Gyro Data page of the OLED.

---

### Type 12 — Distance Sensor Data Report

**From:** Myles (`0x4D`)

| Payload Byte | 1-2 | 3-6 |
|---|---|---|
| **Variable Name** | `message_type` | `distance_m` |
| **C Type** | `uint16_t` | `float` |
| **Min** | `12` | `0.0` |
| **Max** | `12` | `6.0` |
| **Example** | `12` | `0.87` |

Distance in meters as an IEEE 754 little-endian float. Myles only transmits this message when the reading is at or below 0.50 m, so silence means nothing is close. The OLED converts to centimeters (multiply by 100.0) for display.

---

## Broadcast Messages I Handle

Broadcast packets go to address `0x58`. This subsystem processes them and then passes them downstream : both actions are required. Dropping a broadcast breaks the ring for every node that hasn't seen it yet.

### Type 13 — System Status Report

**From:** Damian (`0x44`) or any subsystem

| Payload Byte | 1-2 | 3 | 4-7 | 8-9 |
|---|---|---|---|---|
| **Variable Name** | `message_type` | `system_state` | `uptime_ms` | `battery_mv` |
| **C Type** | `uint16_t` | `uint8_t` | `uint32_t` | `uint16_t` |
| **Min** | `13` | `0` | `0` | `0` |
| **Max** | `13` | `255` | `4294967295` | `65535` |
| **Example** | `13` | `1` | `120000` | `7400` |

System state and battery voltage render on the status page of the OLED. The uptime field is useful for spotting restarts mid-session. Forwarded downstream after processing.

---

### Type 14 — System Error Code Report

**From:** Any subsystem (broadcast)

| Payload Byte | 1-2 | 3 | 4 |
|---|---|---|---|
| **Variable Name** | `message_type` | `subsystem_id` | `error_code` |
| **C Type** | `uint16_t` | `uint8_t` | `uint8_t` |
| **Min** | `14` | `0` | `0` |
| **Max** | `14` | `255` | `255` |
| **Example** | `14` | `3` | `2` |

The subsystem ID field identifies which board threw the error. An alert renders on the OLED and the packet passes downstream.

---

### Type 16 — Heartbeat / Alive Signal

**From:** Any subsystem (broadcast)

| Payload Byte | 1-2 | 3 | 4 |
|---|---|---|---|
| **Variable Name** | `message_type` | `system_state` | `error_flag` |
| **C Type** | `uint16_t` | `uint8_t` | `uint8_t` |
| **Min** | `16` | `0` | `0` |
| **Max** | `16` | `255` | `1` |
| **Example** | `16` | `1` | `0` |

`error_flag`: 0 = nominal, 1 = fault present. The HMI uses these to show a live online/offline indicator per node on the OLED. A board that stops sending heartbeats within the expected window gets flagged as unresponsive. Forwarded downstream.

---

## ACK

Every valid packet addressed directly to me (`0x43`) gets an immediate ACK back to the sender. Broadcast packets (`0x58`) do not : sending ACKs to the broadcast address would flood the bus with a response from every node simultaneously.

| Payload Byte | 1 | 2 |
|---|---|---|
| **Variable Name** | `message_type` | `received_type` |
| **C Type** | `uint8_t` | `uint8_t` |
| **Value** | `0xAA` (170) | echoes back the msg type that triggered it |
| **Example** | `170` | `8` |

`0xAA` is reserved for ACK only. No sensor or system message type uses that value.

---

## Routing Logic

| Condition | Action |
|---|---|
| Receiver == `0x43` (me) | Handle, send ACK, do not forward |
| Receiver == `0x58` (broadcast) | Handle and forward downstream |
| Sender == `0x43` (looped back) | E09 discard silently |
| Receiver == other known node | Forward immediately, do not handle |
| Malformed packet / unknown ID | Discard, print error code to REPL |

RX drains before any outbound packet goes out. A packet that loops back (sender == `0x43`) means either a broadcast completed the ring or something went wrong upstream : either way it gets dropped silently with an E09 log to the REPL.

---

## Byte Stuffing

`build_packet()` subtracts 1 from any payload byte at positions 4-61 that matches a reserved value. The reserved set is `0xA5`, `0x5A`, `0x59`, `0x42` : the four bytes that make up the start and stop bit pairs. Without this, a data value that happens to match those bytes would be misread as a packet boundary mid-stream. Any subsystem parsing messages from this board must add 1 back to recover stuffed bytes before interpreting the data.

---

## Source Code

Full source code is available on the [Resources](../08-Resources/resources.md) page.

[Download HMI Subsystem Code (ZIP)](./Christo_HMI_Final.zip)

MicroPython v1.23.0 on ESP32-S3-WROOM-1-N4.
