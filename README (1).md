# ISL Ground Control Software (GCS)

A single-page **Ground Control Software** dashboard for a CanSat mission, built for the **India Space Lab** brief issued under **Bharat Antariksh Sapataah / India Space Week**.

The dashboard monitors telemetry in real time, visualises mission parameters, tracks live GPS position, exposes mission-critical controls, and simulates end-to-end aerospace ground-station operations — all from a single browser page, with no build step or install required.

![Status](https://img.shields.io/badge/status-complete-brightgreen)
![Platform](https://img.shields.io/badge/platform-browser-blue)
![License](https://img.shields.io/badge/license-MIT-lightgrey)

---

## ✨ Features

- **Live telemetry dashboard** — container and payload telemetry rendered in separate panels, updating once per second (1 Hz), matching a real CanSat downlink structure.
- **4-digit fault code system** — a live, glowing tile-based fault indicator recomputed on every packet (descent rate, GPS availability, payload separation, emergency parachute).
- **Mission Control Panel** — dedicated, visually isolated panel for irreversible commands: Manual Separation, Emergency Parachute Deployment, and Redundant Activation, each producing a timestamped, colour-coded log entry.
- **Real-time graphing** — five live Chart.js plots (Altitude, Pressure, Temperature, Descent Rate, Battery Voltage) over a rolling 60-second window, animation-free for responsiveness.
- **Live GPS tracking map** — Leaflet.js + OpenStreetMap, dark-themed, with an auto-recentring marker and an accumulating descent trajectory polyline.
- **3D orientation visualisation** — a Three.js WebGL CanSat model that rotates live from Roll/Pitch/Yaw telemetry, with numeric readouts.
- **Live video streaming** — camera selection and preview via the MediaDevices / getUserMedia APIs.
- **Hardware-in-the-loop testing** — switch seamlessly between a built-in telemetry simulator and a real serial link via the Web Serial API (115200 baud), with zero code changes to the rendering pipeline.
- **Data export** — timestamped CSV telemetry logs and a composited PNG of all five graphs, both downloaded via the Blob API.

---

## 🖥️ Tech Stack

| Layer | Technology |
|---|---|
| Structure & Styling | HTML5, CSS3 (Grid + Flexbox), Google Fonts (Space Grotesk, IBM Plex Mono, Inter) |
| Application Logic | Vanilla JavaScript (ES6+) |
| Real-Time Graphing | [Chart.js](https://www.chartjs.org/) |
| Tracking Map | [Leaflet.js](https://leafletjs.com/) + OpenStreetMap tiles |
| Orientation Visualization | [Three.js](https://threejs.org/) (WebGL) |
| Live Video | MediaDevices / getUserMedia API |
| Hardware Telemetry Link | Web Serial API |
| Data Export | Blob API, Canvas `toBlob` |

---

## 🚀 Getting Started

This project is a single self-contained HTML file — no build tools, package managers, or servers are required to run it.

### Run locally
```bash
git clone https://github.com/<your-username>/<your-repo>.git
cd <your-repo>
```
Then simply open `gcs.html` in a modern Chromium-based browser (Chrome or Edge recommended).

> **Note:** The Live Video panel requires a secure context. Camera access is blocked when the file is opened directly via `file://`. Serve it locally instead:
> ```bash
> python -m http.server 8000
> # then visit http://localhost:8000/gcs.html
> ```

### Telemetry sources
Use the **Source Selector** in the control bar to switch between:
- **Simulated Feed** — a built-in generator with a realistic ascent-apogee-descent profile, for development and demonstration without hardware.
- **Web Serial (Hardware)** — connects to a microcontroller (tested with the WeGyanik Kit) over a 115200-baud serial link.

---

## 📊 Interface Layout

The dashboard is arranged with CSS Grid into five functional regions:

1. **Masthead** — mission identity, live mission clock (`T+HH:MM:SS`), packet counter, link status.
2. **Control Bar** — Start/Stop, source selector, CSV/graph export, PC time sync, packet reset.
3. **Telemetry Panels** — Container and Payload telemetry, updating every packet.
4. **Graphs Panel** — five live Chart.js plots.
5. **Error Code & Mission Control Panels** — fault indicators and critical command buttons.
6. **Situational Awareness Row** — tracking map, 3D orientation, and live video.

The visual design uses a dark, high-contrast theme with a restrained semantic palette (🟢 nominal, 🟡 caution, 🔴 fault) and monospaced type for all numeric telemetry, matching the readability priorities of real aerospace operator consoles.

---

## 🧭 Telemetry Fields

**Container Telemetry**
- Altitude (m), Pressure (hPa)
- GPS Latitude / Longitude, GPS Satellite Count
- Battery Voltage (V)

**Payload Telemetry**
- Temperature (°C), Descent Rate (m/s)
- Roll / Pitch / Yaw (°)
- Battery Voltage (V)

---

## 🚨 Fault Code System

A live 4-digit fault code is recomputed on every packet:

| Digit | Condition | `0` | `1` |
|---|---|---|---|
| 1 | Descent Rate | Within 8–10 m/s | Outside safe range |
| 2 | GPS Availability | GPS data available (≥4 satellites) | GPS data unavailable |
| 3 | Payload Separation | Separated successfully | Separation failure |
| 4 | Emergency Parachute | Inactive | Activated |

**Example:** `0100` → GPS data unavailable. `1111` → all fault conditions active.

---

## 📦 Telemetry Packet Format

A single CSV frame carries all container, payload, and fault-code fields, used identically by both the simulator and the Web Serial hardware path:

```
PACKET_COUNT,MISSION_TIME_MS,ALTITUDE,PRESSURE,GPS_LAT,GPS_LON,GPS_SATS,TEMPERATURE,DESCENT_RATE,ROLL,PITCH,YAW,BATTERY,ERR_CODE
```

**Example:**
```
042,41000,214.6,988.2,28.61391,77.20904,8,21.3,9.1,3.2,-1.8,7.62,118.4,0000
```

Any microcontroller that prints newline-terminated lines in this format over serial can drive the entire dashboard with no code changes.

---

## 🧪 Testing Strategy

- **Simulated Feed** — exercises every panel (telemetry, graphs, map, orientation, fault codes) under repeatable conditions, including a small chance of a dropped-satellite GPS fault.
- **Web Serial (hardware-in-the-loop)** — validated against a real serial link across multiple conditions: nominal descent, out-of-range descent rate, GPS dropout, and manual/emergency command triggers.

---

## ⚠️ Limitations & Future Work

- Live video requires a secure context (HTTPS or `localhost`); it will not work when opened directly from disk.
- The simulated telemetry generator uses a simplified physics model (linear/sinusoidal approximations), not a full flight-dynamics simulation.
- Web Serial is currently supported only in Chromium-based browsers (Chrome, Edge) — not yet implemented in Firefox or Safari.
- The Payload Separation fault digit is operator-commanded rather than derived from a physical separation sensor, since none was defined in the brief.
- Planned: persistent session storage, multi-mission log comparison, and configurable safe ranges per fault digit.

---

## 📄 Project Info

This project was developed as part of a CanSat/CubeSat project assignment with **India Space Lab**, covering the full assignment brief: interface layout, control bar, mission control, telemetry display, the 4-digit error code system, real-time graphing, GPS tracking, 3D orientation visualisation, live video, and data export.

- **Report period:** 17 June – 31 July 2026
- **Institute:** IIEST, Shibpur

---

## 📜 License

This project is released under the [MIT License](LICENSE) — feel free to fork, adapt, and build on it.
