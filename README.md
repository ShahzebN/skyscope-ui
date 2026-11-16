# SkyScope Pro: Edge-to-Cloud Airspace Telemetry & Radar System

![Build Status](https://img.shields.io/badge/build-passing-brightgreen)
![Platform](https://img.shields.io/badge/hardware-ESP32--S3-blue)
![Cloud](https://img.shields.io/badge/backend-Firebase%20RTDB-orange)
![Live Demo](https://img.shields.io/badge/demo-GitHub%20Pages-00d2ff)
![License](https://img.shields.io/badge/license-MIT-green)

**Live Global Radar:** [https://shahzebn.github.io/skyscope-ui/](https://shahzebn.github.io/skyscope-ui/)

---

## Abstract
**SkyScope Pro** is an edge-to-cloud IoT airspace radar and meteorological telemetry platform powered by an **Espressif ESP32-S3** microcontroller. Designed to monitor real-time commercial and regional aviation across the **Middle East and Pakistan** (Lat: `12.0°N` to `42.0°N`, Lon: `25.0°E` to `77.0°E`), the system enables globally accessible, low-latency radar visualization from any browser without exposing local home networks or exhausting embedded microcontroller memory.

---

## System Architecture

To eliminate embedded SRAM heap fragmentation, bypass browser Cross-Origin Resource Sharing (CORS) restrictions, and avoid router port-forwarding/CGNAT limitations, SkyScope Pro implements a **decoupled Edge-Push Cloud Relay architecture**:

```text
┌───────────────────────────┐        HTTPS GET (TLS)         ┌───────────────────────────┐
│   Telemetry Data Feed     │ ◄───────────────────────────── │   ESP32-S3 Edge Node      │
│ (ME & Pakistan Bounds)    │ ─────────────────────────────► │  (C++ / WiFiClientSecure) │
└───────────────────────────┘        Raw JSON Stream         └─────────────┬─────────────┘
                                                                           │
                                                      Authenticated HTTPS  │
                                                      PUT (?auth=SECRET)   │
                                                                           ▼
┌───────────────────────────┐        Public HTTPS GET        ┌───────────────────────────┐
│  Global Client Browsers   │ ◄───────────────────────────── │ Firebase Realtime DB      │
│  (GitHub Pages Frontend)  │        (/radar.json)           │ (Read-Only Public Node)   │
└───────────────────────────┘                                └───────────────────────────┘
```

### 1. Edge Ingestion & Cloud Push (`ESP32-S3 Firmware`)
* **TLS Telemetry Acquisition:** Every 10 seconds, the ESP32-S3 initiates an outbound `WiFiClientSecure` HTTPS request to ingest live aircraft state vectors strictly bounded to the Middle East and Pakistan airspace.
* **Authenticated Cloud Relay:** The microcontroller pushes the payload via an authenticated `HTTPS PUT` request to a dedicated `/radar.json` node in **Firebase Realtime Database**. Because all connections are outbound from the ESP32, the node operates behind standard NAT/CGNAT firewalls with zero inbound ports exposed.

### 2. Zero-Trust Cloud Security Model (`Firebase RTDB`)
* **Asymmetric Access Control:** Database security rules enforce public read access (`".read": true`) on `/radar` while strictly denying unauthenticated writes (`".write": false`).
* **Embedded Write Token:** Write authorization is restricted exclusively to the ESP32-S3 hardware node via an admin Database Secret stored in flash memory, preventing external tampering while allowing unlimited concurrent global viewers.

### 3. Serverless Client & Telemetry Engine (`GitHub Pages`)
* **Decoupled Hosting:** The entire HTML5/CSS3/JavaScript application is statically hosted on **GitHub Pages**, completely offloading UI rendering and asset delivery from the microcontroller.
* **Client-Side Data Enrichment:**
  * **ICAO Prioritization:** Parses raw telemetry arrays to prioritize 4-letter ICAO callsigns and converts 3-letter IATA airport codes into standard ICAO identifiers (e.g., `ISB` $\rightarrow$ `OPIS`, `DXB` $\rightarrow$ `OMDB`).
  * **Carrier Resolution:** Maps 80+ regional and international ICAO operator prefixes to full airline names and dynamic carrier logos.
  * **Aircraft Imagery:** Queries the Planespotters public REST API by airframe registration tail to display real-time aircraft photography with local client-side caching.
  * **Meteorological Layers:** Overlays live RainViewer precipitation radar, OpenWeatherMap cloud/wind tile streams, and regional lightning telemetry onto a dark-mode Leaflet.js map.

---

## Key Features
* **Global Accessibility:** Viewable anywhere in the world via GitHub Pages whenever the ESP32-S3 edge node is powered on.
* **Multi-Criteria Filtering:** Instant client-side filtering by ICAO callsign, aircraft type (`A388`, `B77W`, `A321`), airline code (`PIA`, `UAE`, `QTR`), registration, or origin/destination ICAO.
* **Aviation Unit Conversions:** Dynamic toggling between Knots / km/h / mph, Feet / Meters, and UTC / Local time formats.
* **Responsive Command Dock:** Adaptive layout engineered for both desktop workstations and mobile devices.

---

## Deployment & Setup

### Hardware & Dependencies
* **Microcontroller:** Espressif ESP32-S3 (2.4 GHz Wi-Fi)
* **Arduino Libraries:** `WiFi.h`, `HTTPClient.h`, `WiFiClientSecure.h`
* **Cloud Services:** Firebase Realtime Database, GitHub Pages

### Configuration Steps
1. **Database Setup:**
   * Create a Firebase Realtime Database instance and configure the security rules:
     ```json
     {
       "rules": {
         "radar": {
           ".read": true,
           ".write": false
         }
       }
     }
     ```
2. **Frontend Deployment:**
   * Update the `fetch()` endpoint in `index.html` to point to `https://<YOUR-PROJECT>.firebaseio.com/radar.json`.
   * Enable **GitHub Pages** on the `main` branch (`/(root)`).
3. **Firmware Flashing:**
   * Configure `ssid`, `password`, and `firebaseUrl` (including your `?auth=<DATABASE_SECRET>` token) in the ESP32-S3 `.ino` sketch and flash via Arduino IDE.

---

## License
Distributed under the MIT License. See `LICENSE` for more information.
