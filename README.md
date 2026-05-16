# 5G-Enabled Haptic Navigation System for the Blind-Deaf

> Distributed Spatial Intelligence and Haptic Feedback Framework using ESP32, FreeRTOS, WebSockets, GPS Telemetry, and Edge-Based Navigation Processing

---

# Overview

This project is a real-time assistive navigation framework designed specifically for **blind and deaf users**. The system replaces traditional visual maps and voice instructions with a **tactile glove-based interaction system** powered by an ESP32 and a Python edge navigation server.

The project follows a **thin-client architecture**:

- The wearable device stays lightweight and power-efficient.
- Heavy navigation computation is offloaded to a Python edge server.
- The ESP32 handles:
  - tactile input
  - GPS synchronization
  - networking
  - state management
  - haptic command control
- The edge server performs:
  - route generation
  - 1-meter path densification
  - bearing calculations
  - drift detection
  - turn extraction
  - instruction dispatching

The communication layer uses persistent WebSocket-based networking to maintain low-latency bidirectional synchronization between the wearable and the server.

---

# Core Idea of the Project

Modern navigation systems assume users can:
- see maps
- hear voice prompts
- read screens

This becomes unusable for deafblind individuals.

This project introduces a new interaction paradigm:
- navigation through touch
- tactile command entry
- haptic directional guidance

The wearable glove becomes the user's:
- keyboard
- command interface
- navigation controller

while the server acts as the:
- spatial intelligence engine
- route processor
- path tracking system

---

# System Architecture

```text
+--------------------------------------------------+
|                USER INTERFACE LAYER              |
|--------------------------------------------------|
|   Tactile Glove Buttons (Dot / Dash / Actions)  |
+------------------------+-------------------------+
                         |
                         v
+--------------------------------------------------+
|                  ESP32 THIN CLIENT               |
|--------------------------------------------------|
| Core 0 -> Input Decoder + State Machine          |
| Core 1 -> GPS + WebSocket + Telemetry Engine     |
|                                                  |
| FreeRTOS                                         |
| Queues                                           |
| Mutex Locks                                      |
| Persistent WebSocket Connection                  |
+------------------------+-------------------------+
                         |
                         v
+--------------------------------------------------+
|              PYTHON EDGE NAVIGATION SERVER       |
|--------------------------------------------------|
| Route Generation                                 |
| Polyline Decoding                                |
| 1 Meter Path Densification                       |
| Bearing Differential Analysis                    |
| Drift Detection                                  |
| Turn Extraction                                  |
| Live Instruction Dispatch                        |
+------------------------+-------------------------+
                         |
                         v
+--------------------------------------------------+
|                HAPTIC NAVIGATION OUTPUT          |
|--------------------------------------------------|
| TURN LEFT                                        |
| TURN RIGHT                                       |
| KEEP STRAIGHT                                    |
| ARRIVED                                          |
+--------------------------------------------------+
```

---

# Repository Structure

```text
.
├── tcp_version.cpp
├── websockets_version.py
└── README.md
```

---

# Hardware Components

## ESP32

The ESP32 is the main wearable controller.

It is responsible for:
- running FreeRTOS tasks
- decoding glove inputs
- GPS synchronization
- maintaining the WebSocket tunnel
- sending telemetry
- receiving navigation instructions

The project uses the ESP32 because:
- it supports dual-core execution
- it supports Wi-Fi networking
- it integrates well with FreeRTOS
- it is power efficient
- it supports real-time multitasking

---

## GPS Module

The GPS module continuously streams:
- latitude
- longitude
- NMEA sentences

The ESP32 parses this data using:

```cpp
TinyGPS++
```

The GPS data is:
- mutex protected
- synchronized between tasks
- transmitted periodically to the server

---

## Tactile Glove Interface

The glove is a tactile command system.

Instead of:
- keyboards
- touchscreens
- microphones

the user interacts using:
- dot input
- dash input
- tactile command buttons

The glove acts as:
- text entry interface
- navigation control system
- emergency interaction layer

---

# ESP32 Pin Connections

| Function | GPIO |
|---|---:|
| Dot Button | 33 |
| Dash Button | 32 |
| Delete Button | 27 |
| Clear Button | 14 |
| Action Button | 13 |
| GPS RX | 16 |
| GPS TX | 17 |

---

# Important Wiring Information

All buttons use:

```cpp
INPUT_PULLUP
```

This means:
- one side of each button connects to GPIO
- the other side connects to GND

When pressed:
- pin becomes LOW

---

# Required Libraries

# ESP32 Libraries

Install:
- TinyGPS++
- ArduinoWebsockets
- ESP32 Arduino Core

---

# Python Libraries

Install using:

```bash
pip install websockets googlemaps polyline folium
```

---

# Complete Workflow

## Step 1 — User Enters Destination

The user presses:
- dot
- dash

to generate encoded characters.

---

## Step 2 — ESP32 Decodes Input

The ESP32:
- stores the current frame
- converts dot/dash into binary
- maps binary into characters
- builds the destination string

---

## Step 3 — Destination Sent to Server

ESP32 sends:

```json
{
  "dest": "A",
  "lat": 26.18,
  "lon": 91.69
}
```

---

## Step 4 — Server Generates Route

The Python server:
- requests walking directions
- decodes route geometry
- creates a dense path

---

## Step 5 — Continuous GPS Tracking

ESP32 sends telemetry every few seconds.

Server:
- tracks user position
- compares against dense route

---

## Step 6 — Navigation Instructions Returned

Server sends:
- KEEP STRAIGHT
- TURN_LEFT
- TURN_RIGHT
- ARRIVED

---

# How the Glove Works

The glove is based on tactile encoding.

Each button press contributes to a binary frame.

The firmware uses:
- `.` → binary 0
- `-` → binary 1

Every completed 5-symbol frame is decoded into:
- letters
- numbers
- spaces

The ESP32 stores these decoded characters inside:

```cpp
destBuffer
```

---

# Glove Interaction Modes

The glove dynamically changes functionality based on system state.

---

# MODE_INPUT

Used for:
- typing destination

Buttons:
- Dot → add dot
- Dash → add dash
- Clear → delete current frame
- Delete(long press) → wipe memory
- Action → move to review mode

---

# MODE_REVIEW

Used for:
- confirmation before transmission

Buttons:
- Action → start navigation
- Delete → return to input

---

# MODE_NAVIGATION

Used during active navigation.

Buttons:
- Dot → repeat last instruction
- Dash → pause navigation
- Action → reroute
- Delete(long press) → terminate session

---

# MODE_PAUSED

Used during temporary stop state.

Buttons:
- Dash → resume navigation

---

# ESP32 Firmware Architecture

File:

```text
tcp_version.cpp
```

The firmware is divided into:
- Core 0
- Core 1

using FreeRTOS task pinning.

---

# FreeRTOS Design

The project uses:
- multitasking
- queues
- mutexes
- pinned tasks

This prevents:
- GPS corruption
- blocking operations
- networking delays affecting input

---

# Core 0 — Input Processing Engine

Task:

```cpp
Task_Core0_Processor
```

Responsible for:
- button polling
- debounce filtering
- character decoding
- state transitions
- queue message generation

---

# Debounce Filtering

Function:

```cpp
checkTap()
```

Purpose:
- remove switch noise
- detect stable presses
- measure press duration

Debounce delay:

```text
25 milliseconds
```

---

# Character Decoder

Function:

```cpp
decode5Bit()
```

Logic:
- dot = 0
- dash = 1

Maps:
- 0–25 → A-Z
- 26 → space
- 27–36 → numbers

---

# Buffer Management

## frameBuffer

Stores current:
- 5-symbol temporary frame

---

## destBuffer

Stores:
- final destination string

---

## recoveryBuffer

Stores:
- navigation session for auto-recovery after disconnect

---

# Queue-Based Communication

The ESP32 uses:

```cpp
msgQueue
```

Purpose:
- safely pass messages between tasks

Without queues:
- race conditions could occur
- networking could block input processing

---

# Mutex Protection

The project uses:

```cpp
gpsMutex
```

Purpose:
- protect shared GPS coordinates

Without mutexes:
- simultaneous access from two tasks could corrupt telemetry data

---

# Core 1 — Radio + GPS Synchronization Engine

Task:

```cpp
Task_Core1_Radio
```

Responsible for:
- Wi-Fi connection
- WebSocket maintenance
- GPS polling
- telemetry transmission
- reconnect handling

---

# WebSocket Communication

ESP32 Library:

```cpp
ArduinoWebsockets
```

Python Library:

```python
websockets
```

Why WebSockets?
- persistent full-duplex communication
- low-latency updates
- continuous telemetry streaming
- bidirectional control

---

# Persistent Connection Recovery

If the network disconnects:
- navigation pauses
- destination preserved
- reconnect initiated
- auto-resume triggered

Variables used:

```cpp
needsAutoResume
```

and:

```cpp
recoveryBuffer
```

---

# Heartbeat Kill-Switch

The ESP32 constantly monitors:

```cpp
lastServerResponse
```

If the server stops responding:
- socket forcibly closes
- stale navigation sessions prevented

This prevents:
- frozen sessions
- dead sockets
- incorrect navigation states

---

# GPS Telemetry System

Telemetry sent every:

```text
5 seconds
```

Packet format:

```json
{
  "lat": 26.18,
  "lon": 91.69
}
```

---

# Indoor GPS Override

If GPS lock fails:
- firmware injects simulated coordinates

Purpose:
- indoor demos
- hackathon testing
- satellite unavailable environments

---

# Python Edge Navigation Server

File:

```text
websockets_version.py
```

This acts as the:
- navigation engine
- geometry processor
- telemetry analyzer
- instruction dispatcher

---

# Python Libraries Used

```python
asyncio
websockets
googlemaps
polyline
folium
math
json
```

---

# What Each Library Does

## asyncio
Provides asynchronous execution.

## websockets
Maintains WebSocket server.

## googlemaps
Requests walking directions.

## polyline
Decodes route geometry.

## folium
Generates live HTML navigation map.

## math
Distance + bearing calculations.

---

# Navigation Engine

Class:

```python
MasterNavigator
```

Stores:
- dense route
- action points
- user position
- instruction history
- session state

---

# Haversine Formula

Function:

```python
haversine()
```

Used for:
- distance calculations
- arrival checks
- drift detection
- turn trigger checks

---

# Bearing Analysis

Function:

```python
get_bearing()
```

Used for:
- turn extraction

The engine compares:
- incoming path direction
- outgoing path direction

If angular change exceeds:

```text
22 degrees
```

a turn is detected.

---

# 1-Meter Path Densification

Google routes are sparse.

The server interpolates:

```text
1-meter spaced nodes
```

Why?
- precise turn timing
- accurate tracking
- smoother navigation
- better map matching

---

# Turn Trigger Logic

Instructions are sent:

```text
1 meter BEFORE actual turn
```

Purpose:
- compensate for human reaction delay

---

# 4-Meter Catch Radius

The server triggers instructions if:

```text
distance < 4m
```

Why?
- GPS jitter
- delayed telemetry
- walking motion uncertainty

---

# Drift Detection

If user deviates:

```text
> 12 meters
```

Server sends:

```text
DRIFT: CHECK PATH
```

---

# Arrival Detection

If distance to destination becomes:

```text
< 8 meters
```

Server sends:

```text
ARRIVED
```

---

# Live HTML Map Generation

Generated using:

```python
folium
```

Creates:

```text
live_nav_map.html
```

Shows:
- user marker
- dense route
- turn points
- trigger points

Useful for:
- debugging
- demonstrations
- visual verification

---

# Message Protocol

# ESP32 → Server

## Start Navigation

```json
{
  "dest": "A",
  "lat": 26.18,
  "lon": 91.69
}
```

---

## GPS Telemetry

```json
{
  "lat": 26.18,
  "lon": 91.69
}
```

---

## Commands

```json
{
  "cmd": "pause"
}
```

Commands:
- pause
- resume
- reroute
- cancel
- resay

---

# Server → ESP32

Possible responses:

```text
KEEP STRAIGHT
TURN_LEFT
TURN_RIGHT
ARRIVED
PAUSED
RESUMED
REROUTING COMPLETE
ERROR_NO_ROUTE
```

---

# How to Run the Project

# Step 1 — Install Python Libraries

```bash
pip install websockets googlemaps polyline folium
```

---

# Step 2 — Add Google API Key

Inside:

```python
websockets_version.py
```

replace:

```python
GOOGLE_KEY = "YOUR_KEY"
```

---

# Step 3 — Start Python Server

Run:

```bash
python websockets_version.py
```

Server prints:

```text
UPDATE ESP32 'ws_server_ip' TO: xxx.xxx.xxx.xxx
```

Copy that IP.

---

# Step 4 — Update ESP32 Server IP

Inside:

```cpp
tcp_version.cpp
```

set:

```cpp
const char* ws_server_ip
```

to the server IP.

---

# Step 5 — Flash ESP32

Upload firmware using:
- Arduino IDE
- PlatformIO

---

# Step 6 — Open Serial Monitor

Baud rate:

```text
115200
```

---

# Step 7 — Start Navigation

1. Power ESP32
2. Connect Wi-Fi
3. Open WebSocket tunnel
4. Enter destination
5. Confirm
6. Start navigation
7. Follow haptic instructions

---

# Example Navigation Session

```text
BOOT
WIFI CONNECTED
WEBSOCKET OPEN
GPS SYNCED
DESTINATION ENTERED
ROUTE GENERATED
KEEP STRAIGHT
TURN_LEFT
KEEP STRAIGHT
ARRIVED
```

---

# Engineering Concepts Demonstrated

## Distributed Intelligence
Heavy computation moved to edge server.

## Thin-Client Architecture
Wearable remains lightweight.

## Real-Time Systems
FreeRTOS enables deterministic behavior.

## Concurrent Processing
Dual-core separation prevents blocking.

## Persistent Networking
WebSockets maintain low-latency synchronization.

## Geometric Navigation Processing
Bearing analysis extracts turns.

## Path Densification
Improves spatial precision.

## GPS Map Matching
Improves navigation stability.

---

# Future Improvements

Possible future additions:
- BLE backup communication
- MQTT support
- obstacle detection
- computer vision
- offline routing
- encrypted sockets
- mobile app interface
- battery optimization
- AI obstacle prediction

---

# Troubleshooting

# ESP32 Not Connecting

Check:
- SSID
- password
- power supply

---

# GPS Always Zero

Check:
- UART wiring
- antenna visibility
- baud rate

---

# WebSocket Failure

Check:
- server IP
- firewall
- port 8765

---

# Route Generation Failure

Check:
- Google API key
- internet connectivity
- Directions API enabled

---

# Authors

Add:
- Emani Sri Ajay Karthik, G Manishankar
- IIT GUWAHATI
- acknowledgements

---

