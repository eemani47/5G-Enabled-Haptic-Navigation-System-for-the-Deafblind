# Alternative Implementations

This directory contains additional communication architectures, latency benchmarking modules, and experimental implementations developed during the design and optimization of the 5G-Enabled Haptic Navigation System for the Blind-Deaf.

These modules were created to evaluate:
- alternative networking protocols
- communication reliability
- latency behavior
- lightweight telemetry architectures
- ESP32 synchronization performance
- real-time responsiveness under different communication stacks

---

# Purpose of this Folder

The primary system implementation in this repository uses:
- WebSockets
- persistent full-duplex communication
- edge-based navigation processing

However, during development multiple alternative implementations were explored to:
- compare communication efficiency
- measure latency
- validate reliability
- test synchronization under different architectures
- benchmark transmission overhead

This folder stores those experimental and auxiliary implementations.

---

# Folder Contents

## `server_mqtt_code.py`

Python-based MQTT server implementation.

Purpose:
- test MQTT communication instead of WebSockets
- compare broker-based communication performance
- analyze lightweight publish/subscribe messaging

Features:
- MQTT message handling
- telemetry reception
- ESP32 synchronization
- navigation packet dispatching

---

## `user_mqtt_code.cpp`

ESP32 firmware using MQTT communication.

Purpose:
- compare MQTT against persistent WebSocket architecture
- evaluate lightweight IoT-oriented networking
- benchmark telemetry transmission timing

Features:
- ESP32 MQTT client
- GPS synchronization
- telemetry publishing
- command subscription
- navigation message handling

---

## `websockets_latency.py`

Python latency benchmarking server.

Purpose:
- measure communication latency
- benchmark packet turnaround timing
- analyze WebSocket responsiveness

Used for:
- round-trip timing tests
- latency profiling
- synchronization validation

---

## `websockets_latency_test.cpp`

ESP32 latency benchmarking firmware.

Purpose:
- send repeated timing packets
- measure server response latency
- validate real-time communication behavior

Features:
- timestamp generation
- repeated ping transmission
- latency logging
- synchronization benchmarking

---

# Why Multiple Implementations Were Developed

During the project lifecycle, multiple communication approaches were evaluated to determine the most suitable architecture for a real-time assistive navigation system.

The key engineering goals included:
- deterministic low latency
- persistent synchronization
- reliable telemetry delivery
- low computational overhead
- minimal packet delay
- stable communication under mobility

The project therefore explored:
- WebSockets
- MQTT
- latency benchmarking systems
- alternate synchronization architectures

before finalizing the primary WebSocket-based implementation.

---

# MQTT vs WebSockets

## MQTT

Advantages:
- lightweight protocol
- low bandwidth usage
- broker-based architecture
- efficient for IoT telemetry

Limitations observed:
- increased dependency on broker reliability
- additional publish/subscribe overhead
- less direct persistent interaction

---

## WebSockets

Advantages:
- persistent full-duplex communication
- lower interaction overhead
- faster bidirectional synchronization
- better suited for continuous navigation feedback

Final system architecture selected:
```text
WebSockets + Edge Processing
```

because it provided:
- better real-time responsiveness
- smoother telemetry synchronization
- more deterministic navigation behavior

---

# Latency Benchmarking

The latency benchmarking modules were developed to experimentally validate:
- communication delay
- synchronization consistency
- packet transmission speed
- ESP32 responsiveness
- server turnaround timing

These tests were critical because:
- tactile navigation requires near real-time feedback
- delayed turn instructions can reduce navigation safety
- network jitter directly affects user guidance quality

---

# Engineering Concepts Explored

This directory demonstrates experimentation with:

- MQTT Communication
- WebSocket Synchronization
- Real-Time Telemetry
- ESP32 Networking
- Packet Latency Measurement
- Full-Duplex Communication
- IoT Communication Architectures
- Low-Latency Edge Networking
- Navigation Synchronization Systems

---

# Research and Development Significance

These implementations represent the experimental phase of the project where different networking and synchronization strategies were explored before converging on the final architecture.

This folder therefore documents:
- protocol comparisons
- engineering experimentation
- latency optimization efforts
- networking evaluations
- architecture validation workflows

---

# Notes

These files are supplementary implementations and experimental modules.

The primary production-oriented implementation of the project remains:
- `server_end_code.py`
- `user_module_code.cpp`

located in the root repository directory.

---
