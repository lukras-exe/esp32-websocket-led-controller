# ESP32 WebSocket LED Server

A self-contained LED brightness controller running entirely on an ESP32 — no external server, no cloud, no dependencies. The device hosts its own HTML/CSS/JS frontend out of SPIFFS flash and communicates with the browser over a live WebSocket connection to drive PWM output in real time.

---

## How It Works

```
Browser (Slider UI)
      │
      │  WebSocket (ws://<esp32-ip>/ws)
      ▼
ESP32 WebSocket Server
      │
      │  ledcSetup() / ledcAttachPin()
      ▼
PWM Output → LED
```

1. ESP32 boots and connects to Wi-Fi
2. A web server serves the frontend (HTML/CSS/JS) stored in SPIFFS flash — no SD card, no external host
3. Browser connects to the ESP32 via WebSocket
4. Slider input sends brightness values (0–255) over the WebSocket in real time
5. ESP32 receives the value and updates PWM duty cycle instantly

---

## Features

- **No external server** — frontend lives in SPIFFS flash on the device itself
- **Bidirectional WebSocket** — low-latency, persistent connection between browser and ESP32
- **Real-time PWM control** — LED brightness updates immediately on slider movement
- **Responsive UI** — usable from any device on the same network

---

## Hardware

| Component | Details |
|---|---|
| Microcontroller | ESP32 (tested on ESP32 DevKit V1) |
| LED | Standard 5mm LED with current-limiting resistor |
| Resistor | 220Ω (adjust based on LED forward voltage) |
| Power | USB via DevKit onboard regulator |

**Wiring:**

```
ESP32 GPIO 13 → 220Ω resistor → LED anode
LED cathode → GND
```

---

## Software Dependencies

- [Arduino ESP32 Core](https://github.com/espressif/arduino-esp32) — tested on **v2.x** (see note below)
- [ESPAsyncWebServer](https://github.com/me-no-dev/ESPAsyncWebServer)
- [AsyncTCP](https://github.com/me-no-dev/AsyncTCP)
- Arduino IDE or PlatformIO

---

## Project Structure

```
esp32-websocket-led/
├── esp32-websocket-led.ino   # Main firmware
├── data/
│   ├── index.html            # Slider UI (served from SPIFFS)
│   ├── style.css
│   └── script.js
└── README.md
```

---

## Setup & Flash

### 1. Install dependencies
In Arduino IDE, install via Library Manager:
- ESPAsyncWebServer
- AsyncTCP

### 2. Upload SPIFFS filesystem
- Install the [Arduino ESP32 SPIFFS upload tool](https://github.com/me-no-dev/arduino-esp32fs-plugin)
- Place frontend files in `/data`
- Tools → ESP32 Sketch Data Upload

### 3. Configure Wi-Fi
In `esp32-websocket-led.ino`, update:
```cpp
const char* ssid     = "YOUR_SSID";
const char* password = "YOUR_PASSWORD";
```

### 4. Flash and run
Upload the sketch. Open Serial Monitor at 115200 baud to get the ESP32's IP address, then navigate to it in a browser.

---

## PWM Note — Arduino ESP32 Core Version

This project uses the **stable two-call PWM init pattern** compatible with ESP32 Arduino Core v2.x:

```cpp
ledcSetup(channel, frequency, resolution);
ledcAttachPin(pin, channel);
ledcWrite(channel, dutyCycle);
```

> ⚠️ **If you're on Arduino ESP32 Core v3.x**, the API changed to a single `ledcAttachChannel()` call. The above pattern will produce a compiler warning or error. Either pin your core version to 2.x or migrate to the new API.

This was a real debugging issue encountered during development — the breaking change between core versions is not obvious from the error messages.

---

## Demo

> _Add a photo or GIF of the hardware + browser UI here_

---

## License

MIT
