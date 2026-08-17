# ESP32 802.11 Frame Capture & Visualizer

> **A complete toolset for capturing, parsing, and visualizing 802.11 management/control/data frames using ESP32 / ESP32-C3, with a dedicated Python GUI for real‑time analysis.**

[![PlatformIO](https://img.shields.io/badge/PlatformIO-ESP32-orange?style=flat&logo=platformio)](https://platformio.org/)
[![Arduino](https://img.shields.io/badge/Arduino-ESP32-00979D?style=flat&logo=arduino)](https://arduino-esp32.readthedocs.io/)
[![Python](https://img.shields.io/badge/Python-3.7+-3776AB?style=flat&logo=python)](https://www.python.org/)
[![Tkinter](https://img.shields.io/badge/GUI-Tkinter-0078D4?style=flat&logo=tkinter)](https://docs.python.org/3/library/tkinter.html)
[![License](https://img.shields.io/badge/License-MIT-yellow.svg)](LICENSE)

---

## 📖 Table of Contents

- [Project Overview](#project-overview)
- [Features](#features)
- [Firmware: ESP32 802.11 Frame Capture v2.3](#firmware-esp32-80211-frame-capture-v23)
  - [Hardware Requirements](#hardware-requirements)
  - [User Configuration](#user-configuration)
    - [Capture Filters](#capture-filters)
    - [Output Options](#output-options)
    - [Target MAC Filtering](#target-mac-filtering)
    - [Channel Mode](#channel-mode)
    - [LED Indicators](#led-indicators)
    - [Temperature Sensor](#temperature-sensor)
  - [PlatformIO Build & Upload](#platformio-build--upload)
  - [Serial Output Format](#serial-output-format)
  - [Frame Types & Subtypes](#frame-types--subtypes)
  - [Reason Codes](#reason-codes)
  - [Watchdog & Reliability](#watchdog--reliability)
- [Python GUI: Frame Capture Visualizer](#python-gui-frame-capture-visualizer)
  - [Installation & Dependencies](#installation--dependencies)
  - [Running the GUI](#running-the-gui)
  - [GUI Layout & Components](#gui-layout--components)
    - [Serial Control Bar & Marquee](#serial-control-bar--marquee)
    - [Real‑time Statistics Panel](#realtime-statistics-panel)
    - [Temperature Display](#temperature-display)
    - [Frame Type Distribution Chart](#frame-type-distribution-chart)
    - [Frame List Tabs](#frame-list-tabs)
    - [Raw Log Viewer](#raw-log-viewer)
    - [Author & Status Bar](#author--status-bar)
  - [Parsing Engine: Regex & Data Model](#parsing-engine-regex--data-model)
    - [Regular Expressions](#regular-expressions)
    - [Data Model](#data-model)
    - [Frame Parsing Logic](#frame-parsing-logic)
  - [GUI Interaction & Feedback](#gui-interaction--feedback)
- [Full Example Workflow](#full-example-workflow)
- [Performance & Optimization](#performance--optimization)
- [Troubleshooting & FAQ](#troubleshooting--faq)
- [Contributing](#contributing)
- [Author & Credits](#author--credits)
- [License](#license)

---
<img width="1016" height="549" alt="image" src="https://github.com/user-attachments/assets/a8d01968-2d43-404d-9aa8-3fce21de0b9b" />
---
<img width="1452" height="952" alt="image" src="https://github.com/user-attachments/assets/1dbcc5ed-b8cd-41b7-98cc-81f8d4180edc" />
---
<img width="1452" height="952" alt="image" src="https://github.com/user-attachments/assets/371f3755-580b-4c35-8e60-d89ce80bbeae" />
---
<img width="500" height="497" alt="image" src="https://github.com/user-attachments/assets/51b9d8e7-deed-4fde-8f93-ba5c92aab866" />

## 具体视频：
[![点击查看演示视频](demo_thumb.png)](https://github.com/t84RT/ESP32-802.11-Frame-Capture-Visualizer/blob/main/bandicam%202026-08-17%2017-50-49-491.mp4)


## Project Overview

**ESP32 802.11 Frame Capture & Visualizer** is a complete solution for capturing and analyzing WiFi frames in real time. It consists of:

1. **Firmware** (C++ for ESP32/ESP32-C3) that puts the Wi-Fi interface into promiscuous mode, captures management, control, and data frames, applies user‑defined filters, and streams the results over serial with optional detailed fields and hex dumps.

2. **Python GUI** (Tkinter) that reads the serial output, parses every line using pre‑compiled regular expressions, displays live statistics, frame lists (all frames, Deauth‑specific, management frames), a frame type distribution chart, and chip temperature – all while providing a professional look with a marquee, author links, and status bar.

This tool is ideal for:
- Wi‑Fi security testing and monitoring
- Debugging 802.11 protocol interactions
- Educational demonstrations of frame types
- Analysing deauthentication/disassociation attacks
- Monitoring client‑AP reconnection behaviour

The firmware is highly configurable: you can choose which frames to capture, enable detailed field output, set a fixed or hopping channel, filter by target MAC addresses, and even control two LEDs – one for activity and one for temperature‑based blinking.

The GUI is a full‑featured companion that transforms raw serial text into an interactive dashboard, making it easy to spot trends, count frame types, and examine individual frames with their source/destination addresses and detailed fields.

---

## Features

### Firmware (ESP32)
- **Promiscuous capture** of 802.11 management, control, and data frames.
- **Selective filtering** – choose to capture Deauth, Disassoc, CTS, Probe Request, Beacon, or all frames.
- **Target MAC filtering** – focus on specific AP or STA addresses.
- **Flexible channel operation** – fixed channel or hopping across channels 1‑13 with configurable dwell time.
- **Detailed output** – includes sequence number, fragment number, duration, FC flags, reason codes (with textual description), and CTS duration.
- **Hex dump** – complete raw frame dump for Wireshark import.
- **Reassociation detection** – alerts when a Deauth/Disassoc is followed by a quick Assoc/Reassoc.
- **Rate‑limiting** – prevents serial buffer overflow.
- **LED indicators**:
  - Main LED: idle heartbeat, flash on frame capture, double‑flash on statistics print.
  - Auxiliary LED: blinking rate varies with chip temperature (faster when hot).
- **On‑chip temperature sensor** (ESP32/ESP32‑C3) read periodically.
- **Watchdog** – restarts the device if the loop hangs.
- **Zero‑copy frame templates** and minimal overhead for high capture rates.

### Python GUI
- **Cross‑platform** (Windows, macOS, Linux) – requires only Python 3.7+ and `pyserial`.
- **Real‑time serial reading** with separate thread and queue to avoid UI freezing.
- **Automatic port detection** and baud rate selection (default 115200).
- **Full parsing** of every serial line:
  - Firmware version and MAC address.
  - Channel changes.
  - Temperature updates.
  - Frame lines (with or without detailed fields).
  - Statistics blocks.
  - Reassociation warnings.
- **Interactive data displays**:
  - **Statistics panel** – live counts for 8 frame types (Deauth, Disassoc, Assoc Req, Reassoc Req, Probe Req, Beacon, CTS, Other) plus total frames.
  - **Temperature** – large red digits, updated when a temperature line arrives.
  - **Frame type distribution chart** – colour‑coded bar chart that automatically scales.
  - **Frame list tabs**:
    - *All frames* – shows time, channel, type, subtype, DA, SA, BSSID, and detailed info (sequence, duration, reason, FC flags).
    - *Deauth frames* – dedicated list showing only Deauth/Disassoc frames with reason codes.
    - *Management frames* – displays non‑Deauth management frames (Beacon, Probe, Assoc, Reassoc, etc.).
- **Raw log viewer** – scrollable text box showing every line from the serial port.
- **Marquee** – scrolling project title and status at the top right.
- **Author information bar** – clickable links to GitHub, personal website, Bilibili, CSDN, and Blog Park.
- **Status bar** – shows connection state, frame rate (fps), elapsed time, and current channel.
- **Clear display** – one‑click reset of all data and charts.
- **Pre‑compiled regular expressions** for high‑performance parsing.

---

## Firmware: ESP32 802.11 Frame Capture v2.3

The firmware is written in Arduino‑style C++ using the ESP32 Arduino core. It uses the native ESP‑IDF Wi‑Fi promiscuous mode API for maximum efficiency.

### Hardware Requirements
- ESP32 or ESP32‑C3 (or any ESP32‑family with Wi‑Fi).
- USB‑to‑UART adapter (CH340/CP2102) or native USB‑CDC (for ESP32‑C3 boards with built‑in USB).
- Optional: two LEDs (with current‑limiting resistors) connected to GPIO pins (default: main LED on GPIO12, auxiliary LED on GPIO13).

### User Configuration

All user‑definable parameters are grouped at the top of `main.cpp`. You can modify them without touching the core logic.

#### Capture Filters
Uncomment the desired `#define` statements to capture specific frame types:
```cpp
#define CAPTURE_DEAUTH          // Deauth (broadcast + unicast)
#define CAPTURE_DISASSOC        // Disassociation
#define CAPTURE_CTS             // CTS
#define CAPTURE_PROBE_REQ       // Probe Request
#define CAPTURE_BEACON          // Beacon
//#define CAPTURE_ALL           // captures all frames, overrides individual filters
```
If none are defined, the firmware captures **all** frames (default behaviour).

#### Output Options
- `OUTPUT_HEX_DUMP` – prints a hex dump of each captured frame (useful for Wireshark).
- `OUTPUT_DETAILED` – prints additional fields: sequence number, duration, FC flags, reason/CTS duration.
- `OUTPUT_MINIMAL` – prints only frame type, MAC addresses, and length (default).

Only one of these can be active at a time (you can comment/uncomment as needed).

#### Target MAC Filtering
Define `TARGET_AP_MAC` and/or `TARGET_STA_MAC` to focus on specific devices. For example:
```cpp
#define TARGET_AP_MAC   {0x78,0x44,0xFD,0x3B,0x8F,0xD5}
#define TARGET_STA_MAC  {0x54,0x78,0xC9,0x07,0x33,0x0A}
```
If both are zeroed, no filtering is applied.

#### Channel Mode
- `#define FIXED_CHANNEL 11` – fixed on channel 11 (uncomment to enable).
- If not defined, the firmware hops between `CHANNEL_MIN` and `CHANNEL_MAX` (default 1‑13) with `CHANNEL_DWELL_MS` (default 2000 ms) dwell time.

#### LED Indicators
Configure LED pins and behaviour:
```cpp
#define LED_ENABLED         1
#define LED_PIN             12           // main LED
#define LED_PIN_AUX         13           // auxiliary (temperature)
#define LED_ACTIVE_HIGH     1            // 1 = high‑active, 0 = low‑active
#define LED_BLINK_INTERVAL  1000         // idle heartbeat interval (ms)
#define LED_CAPTURE_FLASH   100          // flash duration on capture (ms)
#define LED_STAT_FLASH      300          // double‑flash interval for statistics
```
- Main LED: idle = slow heartbeat; capture = short flash; statistics = double‑flash.
- Auxiliary LED: blinks with frequency proportional to chip temperature (faster = hotter). If temperature sensor fails, it stays off.

#### Temperature Sensor
```cpp
#define TEMP_READ_INTERVAL  500          // read temperature every 500 ms
#define TEMP_CRITICAL       75.0f        // upper limit for LED scaling
```
The auxiliary LED interval is mapped from 38°C (slowest ~1200 ms) to 75°C (fastest ~300 ms).

### PlatformIO Build & Upload

The project uses PlatformIO. The `platformio.ini` file provides two environments:
- `usb_cdc` – for ESP32‑C3 boards with native USB (requires `-DARDUINO_USB_MODE=1` and `-DARDUINO_USB_CDC_ON_BOOT=1`).
- `ch340` – for boards using an external USB‑to‑UART chip (CH340/CP2102). This is the default.

To build and upload:
1. Install PlatformIO (VS Code extension or CLI).
2. Connect your ESP32 board.
3. Select the appropriate environment in `platformio.ini` (change `default_envs`).
4. Run:
   ```bash
   pio run -t upload
   ```
5. Open the serial monitor (or use the GUI) at 115200 baud.

### Serial Output Format

The firmware outputs lines in the following format (depending on enabled options):

#### Frame line (minimal)
```
[12345 ms] [CH 6] ⭐ [Broadcast Deauth] len=26
  DA=FF:FF:FF:FF:FF:FF SA=AA:BB:CC:DD:EE:FF BSSID=AA:BB:CC:DD:EE:FF
```

#### With detailed fields
```
[12345 ms] [CH 6] ⭐ [Broadcast Deauth] len=26
  DA=FF:FF:FF:FF:FF:FF SA=AA:BB:CC:DD:EE:FF BSSID=AA:BB:CC:DD:EE:FF
  Seq=1234 (Frag=0) Duration=314 µs
  FC: ToDS=0 FromDS=0 MoreFrag=0 Retry=0 MoreData=0 Protected=0 Order=0
  Reason=3 (Deauthenticated because sending STA is leaving)
```

#### With hex dump (additional lines)
```
  [HEX DUMP]
    C0 00 00 00 FF FF FF FF FF FF ...
```

#### Statistics block (every 5 seconds)
```
========== STATISTICS ==========
Deauth (去认证)     : 1234
Disassoc (解关联)   : 567
Assoc Req (关联请求): 89
Reassoc Req (重关联): 45
Probe Req (探测请求): 234
Beacon (信标)       : 678
CTS                 : 901
Other (其他)         : 345
Temperature (温度)  : 45.2 °C
============================================
```

#### Reassociation warning
```
⚠️  [REASSOC DETECTED] STA trying to reconnect after kick!
```

#### Channel switch
```
--- Switching to CH 5 ---
```

#### Boot info
```
========================================
   ESP32 802.11 帧捕获工具 v2.3（带温度指示）
   Enhanced Frame Capture + Temp LED
========================================
MAC: 94:B9:7E:XX:XX:XX
Mode: Hopping 1-13 (dwell 2000ms)
Filter: Deauth
Filter: Disassociation
...
```

### Frame Types & Subtypes

The firmware supports the following management frame subtypes (fully named in Chinese and English):

| Subtype (hex) | Name (EN) | Name (CN) |
|---------------|-----------|-----------|
| 0x0 | Association Request | 关联请求 |
| 0x1 | Association Response | 关联响应 |
| 0x2 | Reassociation Request | 重关联请求 |
| 0x3 | Reassociation Response | 重关联响应 |
| 0x4 | Probe Request | 探测请求 |
| 0x5 | Probe Response | 探测响应 |
| 0x8 | Beacon | 信标 |
| 0x9 | ATIM | ATIM通知 |
| 0xA | Disassociation | 解关联 |
| 0xB | Authentication | 认证 |
| 0xC | Deauthentication | 去认证 |
| 0xD | Action | 动作帧 |

Control frames:
| Subtype | Name |
|---------|------|
| 0x7 | Control Wrapper |
| 0x8 | Block ACK Request |
| 0x9 | Block ACK |
| 0xA | PS‑Poll |
| 0xB | RTS |
| 0xC | **CTS** (commonly captured) |
| 0xD | ACK |
| 0xE | CF‑End |
| 0xF | CF‑End+CF‑ACK |

Data frames are also supported (subtypes 0x0‑0xF, including QoS variants).

### Reason Codes

The firmware includes a comprehensive table of reason codes (IEEE 802.11) with both English descriptions and Chinese translations. The most common ones used in deauthentication attacks are:
- 1: Unspecified
- 2: Previous authentication no longer valid
- 3: STA is leaving (deauth)
- 4: Disassociated due to inactivity
- 8: STA is leaving (disassoc)
- 15: 4‑way handshake timeout
- 16: Group key handshake timeout
- etc. (see code for full list).

### Watchdog & Reliability

A software watchdog in `loop()` checks if the main loop has been stuck for more than `WATCHDOG_TIMEOUT` (10 seconds). If so, it restarts the ESP32 to recover from any deadlock. This ensures long‑term stability in headless deployments.

---

## Python GUI: Frame Capture Visualizer

The GUI is a single Python script (`ESP32_802.11帧捕获工具v2.3专用解析器.py`) that uses only standard libraries (`tkinter`, `serial`, `threading`, `queue`, `re`, `time`, `webbrowser`). It is designed to work with the serial output of the above firmware.

### Installation & Dependencies

- Python 3.7 or newer.
- Install `pyserial`:
  ```bash
  pip install pyserial
  ```
- `tkinter` is included with Python on most platforms (on Linux you may need `python3-tk`).

### Running the GUI

```bash
python ESP32_802.11帧捕获工具v2.3专用解析器.py
```

### GUI Layout & Components

The main window is divided into several areas:

#### Serial Control Bar & Marquee
- **Port selection** – combo box with detected serial ports.
- **Refresh** – rescans for new ports.
- **Baud rate** – default 115200 (other rates supported).
- **Open/Close** – toggles serial connection.
- **Clear display** – resets all data and charts.
- **Status label** – shows connection state.
- **Marquee** – scrolls a welcome message and project title at the top right.

#### Real‑time Statistics Panel (left side)
- **Total frames** – sum of all captured frames.
- **8 frame types** – each with an incremental counter:
  - Deauth (去认证)
  - Disassoc (解关联)
  - Assoc Req (关联请求)
  - Reassoc Req (重关联)
  - Probe Req (探测请求)
  - Beacon (信标)
  - CTS
  - Other (其他)
- The statistics update automatically whenever the firmware prints a statistics block.

#### Temperature Display
- Large red digits showing the last received chip temperature (in °C).
- Updates when the firmware outputs a temperature line (part of statistics block).

#### Frame Type Distribution Chart (right side)
- A colour‑coded bar chart that dynamically scales to the maximum value.
- Bars represent the 8 frame types with distinct colours:
  - Deauth – red
  - Disassoc – orange
  - Assoc – blue
  - Reassoc – light blue
  - Probe – yellow‑green
  - Beacon – green
  - CTS – purple
  - Other – grey
- Chart automatically redraws when statistics change.

#### Frame List Tabs (bottom centre)
Three notebook tabs:
1. **All frames** – shows every captured frame that passes the filters. Columns:
   - Time (ms)
   - CH (channel)
   - Type (e.g., Deauth, Disassoc, Beacon, CTS, Data, Other)
   - Subtype (detailed description)
   - DA (Destination Address)
   - SA (Source Address)
   - BSSID
   - Info (detailed fields: Seq, Duration, Reason, FC flags, etc.)
2. **Deauth frames** – dedicated to Deauth and Disassoc frames, with a “Reason” column.
3. **Management frames** – shows all management frames except Deauth/Disassoc (e.g., Beacon, Probe, Assoc, Reassoc).
Each list maintains a maximum of 500/300 entries to avoid memory issues; the oldest entries are discarded.

#### Raw Log Viewer
- A scrollable text box that displays every line received from the serial port.
- Automatically scrolls to the bottom.
- Useful for debugging and for seeing hex dumps that are not parsed by the GUI.

#### Author & Status Bar
- **Author bar** – shows name, email, and clickable links to GitHub, personal site, Bilibili, CSDN, Blog Park. The links open in the default browser.
- **Status bar** – at the very bottom, shows:
  - Connection state (Running/Idle)
  - Frame rate (fps) = total_frames / elapsed_time
  - Elapsed time (seconds)
  - Current channel (extracted from channel switch lines)

### Parsing Engine: Regex & Data Model

#### Regular Expressions
All regex patterns are pre‑compiled in `__init__` for speed:
| Pattern | Purpose | Groups |
|---------|---------|--------|
| `re_boot` | `ESP32 802\.11 帧捕获工具 v([\d.]+)` | version |
| `re_mac` | `MAC:\s*([0-9A-F:]{17})` | MAC address |
| `re_channel_switch` | `--- Switching to CH(\d+) ---` | channel |
| `re_channel_fixed` | `Fixed on CH(\d+)` | channel |
| `re_temp_line` | `Temperature.*?([\d.]+)` | temperature value |
| `re_reassoc` | `⚠️\s+\[REASSOC DETECTED\]` | – |
| `re_stats` | multi‑line pattern capturing all 8 counters | 8 groups |
| `re_mgmt_frame` | frame header: `[CH(\d+)]\s*([⭐🔹🔸📋⚡📦❓])\s*\[([^\]]+)\]\s*len=(\d+)\s*\n\s*DA=([0-9A-F:]+)\s+SA=([0-9A-F:]+)\s+BSSID=([0-9A-F:]+)` | ch, icon, type, len, DA, SA, BSSID |
| `re_detailed_seq` | `Seq=(\d+)\s*\(Frag=(\d+)\)` | seq, frag |
| `re_detailed_duration` | `Duration=(\d+) µs` | duration |
| `re_detailed_reason` | `Reason=(\d+)\s*\(([^)]+)\)` | reason code, description |
| `re_detailed_cts_dur` | `CTS Duration=(\d+) µs` | duration |
| `re_detailed_fc` | `ToDS=(\d+)\s+FromDS=(\d+)\s+MoreFrag=(\d+)\s+Retry=(\d+)\s+MoreData=(\d+)\s+Protected=(\d+)\s+Order=(\d+)` | 7 FC flags |

#### Data Model
- `self.stats` – dictionary with keys: `deauth`, `disassoc`, `assoc_req`, `reassoc_req`, `probe_req`, `beacon`, `cts`, `other`.
- `self.total_frames` – sum of all counters.
- `self.temperature` – last temperature value.
- `self.current_channel` – last known channel (from `Fixed on` or `Switching to`).
- `self.frame_history` – list of dictionaries, each containing: `time`, `ch`, `type`, `subtype`, `da`, `sa`, `bssid`, `info`, `raw`. Max 500 entries.
- `self.firmware_info` – version string.
- `self.mac_address` – device MAC (only stored, not displayed).

#### Frame Parsing Logic
When a line matches the frame header (`re_mgmt_frame`), the parser:
1. Extracts `ch`, `icon`, `frame_type_full`, `len`, `da`, `sa`, `bssid`.
2. Determines the main frame type and subtype by inspecting `frame_type_full`:
   - Contains `"Deauth"` → type `"Deauth"`, subtype `"广播去认证"` if `"Broadcast"` in string else `"单播去认证"`.
   - Contains `"Disassociation"` → `"Disassoc"`, `"解关联"`.
   - Contains `"Beacon"` → `"Beacon"`, `"信标"`.
   - Contains `"Probe Request"` → `"ProbeReq"`, `"探测请求"`.
   - Contains `"Association Request"` → `"AssocReq"`, `"关联请求"`.
   - Contains `"Reassociation Request"` → `"ReassocReq"`, `"重关联请求"`.
   - Contains `"CTS"` → `"CTS"`, `"CTS"`.
   - Contains `"Data"` → `"Data"`, uses the full subtype name from the frame line.
   - Else → `"Other"`, uses the full name.
3. Applies additional regex searches (`re_detailed_seq`, `re_detailed_duration`, `re_detailed_reason`, `re_detailed_cts_dur`, `re_detailed_fc`) to extract detailed information and concatenates them into an `info` string.
4. Extracts timestamp from `[(\d+) ms]` at the beginning of the line.
5. Builds a frame data dictionary and appends to `self.frame_history` (dropping oldest if over 500).
6. Inserts the frame into the “All frames” tree, and conditionally into the “Deauth frames” or “Management frames” trees.

### GUI Interaction & Feedback
- **Real‑time updates** – every 100 ms the GUI checks the queue and calls `update_ui()` which refreshes statistics, trees, chart, and status bar.
- **Clear display** – resets all internal counters, clears trees, logs, and redraws the chart.
- **Marquee** – runs continuously, updating every 120 ms.
- **Status bar** – shows dynamic information including channel changes (detected from serial).

---

## Full Example Workflow

1. **Prepare the firmware**:
   - Edit `main.cpp` to set desired capture filters, output options, channel mode, and LED pins.
   - Build and upload using PlatformIO (select correct environment).
2. **Run the Python GUI**:
   - Install `pyserial`.
   - Execute the script.
   - Select the correct serial port and baud rate (115200).
   - Click “Open Serial”.
3. **Observe**:
   - The firmware starts, prints boot info, and begins scanning/hopping.
   - The GUI displays boot info in the raw log.
   - As frames are captured, they appear in the frame lists; statistics update every 5 seconds; the chart redraws; temperature appears.
   - If a reassociation event occurs, the status bar shows a warning.
4. **Interact**:
   - Use “Clear display” to reset.
   - Click author links to open web pages.
   - Monitor frame rates and channel changes in the status bar.

---

## Performance & Optimization

- **Firmware**:
  - Pre‑built frame templates for Deauth/Disassoc to reduce per‑frame overhead.
  - Hardware RNG for random sequence numbers (not used in this version, but present in other deauther projects).
  - Rate‑limiting (15 ms between frame prints) prevents serial overflow.
  - Watchdog ensures recovery from hangs.
- **GUI**:
  - Pre‑compiled regular expressions for fast parsing.
  - Separate thread for serial reading; queue for decoupling.
  - Treeviews limited to 500 entries to maintain responsiveness.
  - Chart redraw only on statistics update (not per frame).

---

## Troubleshooting & FAQ

**Q: The GUI shows no frames / statistics remain zero.**  
A: Check that the firmware is actually capturing frames. In the raw log, you should see `[SCAN]` or frame lines. If not, verify that the ESP32 is in promiscuous mode and that your filters are not too restrictive. Also ensure the serial connection is correct and baud rate matches.

**Q: I see hex dump lines, but the GUI does not parse them.**  
A: The GUI does not parse hex dumps; they are only shown in the raw log. This is intentional because hex dumps are for advanced analysis. You can copy them from the raw log for external tools.

**Q: The chart does not update correctly.**  
A: The chart is redrawn only when statistics change (when a `STATISTICS` block is parsed). If statistics are not printed (e.g., because `STAT_INTERVAL_MS` is too long), the chart will stay static. You can reduce the interval in the firmware.

**Q: The GUI freezes or lags.**  
A: The GUI runs all parsing and UI updates in the main thread after reading from the queue. If the queue receives a huge burst of data, the main thread might become busy. You can reduce the serial rate (increase `lastPrint` delay) or reduce the number of frames printed.

**Q: Temperature shows `--.- °C`.**  
A: The firmware must have `LED_ENABLED` defined and the temperature sensor must be supported on your chip. Not all ESP32 variants have an internal temperature sensor (ESP32‑C3 does). Check the raw log for temperature lines.

---

## Contributing

Contributions are welcome! You can help by:
- Reporting issues or suggesting improvements.
- Submitting pull requests for bug fixes or new features.
- Translating the GUI interface to other languages (current labels are in Chinese/English mix).

Guidelines:
- Follow PEP 8 for Python code.
- Use descriptive variable names and comments.
- Test your changes on actual hardware before submitting.

---

## Author & Credits

**Author:** 小吴同学电气设计 (t84RT)  
- Email: xiaowu112899@outlook.com  
- GitHub: [t84RT](https://github.com/t84RT)  
- Personal Site: [https://t84RT.github.io](https://t84RT.github.io)  
- Bilibili: [space.bilibili.com/482117704](https://space.bilibili.com/482117704)  
- CSDN: [blog.csdn.net/weixin_45922157](https://blog.csdn.net/weixin_45922157)  
- Blog Park: [cnblogs.com/Student-Wu-s-Electrical-Design-t84RT](https://www.cnblogs.com/Student-Wu-s-Electrical-Design-t84RT/)

This project was inspired by the need to visualise 802.11 frame captures from ESP32‑based sniffers. Special thanks to the ESP32 community and the PlatformIO team.

---

## License

This project is licensed under the MIT License – see the [LICENSE](LICENSE) file for details.
