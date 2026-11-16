SkyScope Pro: Edge-Optimized Airspace Telemetry & Radar System

Abstract

SkyScope Pro is a high-performance, embedded airspace radar system designed for the ESP32-S3 microcontroller. It provides real-time flight tracking, meteorological overlays, and aircraft telemetry for the Middle East and Subcontinent regions.

To overcome the strict memory constraints of embedded microcontrollers (SRAM limits) and modern browser CORS (Cross-Origin Resource Sharing) security policies, this project employs a decoupled Edge-Proxy architecture. The ESP32 acts as a secure backend proxy for telemetry data, while the heavyweight Graphical User Interface (GUI) is offloaded to a cloud-hosted static endpoint (GitHub).

System Architecture

Edge Proxy (ESP32-S3):

Establishes a secure TLS/SSL connection to flight data endpoints.

Streams chunked JSON payloads (up to 20MB) using highly optimized memory buffering, preventing heap exhaustion.

Bypasses strict CORS policies by serving as a local intermediary between the client browser and third-party APIs.

Cloud-Hosted UI (GitHub):

A lightweight HTML/JS loader on the ESP32 fetches the main application interface directly from this repository via GitHub's raw content delivery network.

Enables Over-The-Air (OTA) UI updates without requiring the microcontroller to be re-flashed.

Data Aggregation Engine (Frontend):

Telemetry: Parses live coordinates, altitude, heading, and speed natively in the browser to free up the ESP32's processing threads.

Meteorology: Integrates OpenWeatherMap and RainViewer tile servers for live cloud cover, precipitation, wind streams, and lightning strikes.

Aviation Data: Dynamically maps 3-letter IATA codes to 4-letter ICAO codes and translates operator codes into full airline names (e.g., CPA -> Cathay Pacific).

Key Features

Real-Time Tracking: 10-second polling interval with zero-latency map rendering via Leaflet.js.

Dynamic Meteorological Layers: Interactive overlays for rain, wind, clouds, and lightning.

Aircraft Identification: Automated fetching of aircraft registration photos via the Planespotters API.

Responsive Command Dock: Mobile-optimized UI with dynamic unit conversions (Metric/Imperial/Aviation standards) and time-zone handling.

Installation & Deployment

1. Hardware Requirements

ESP32-S3 Development Board

2.4 GHz WiFi Network

2. Software Setup

Frontend Hosting:

Clone or fork this repository.

Ensure index.html is accessible via raw GitHub user content or GitHub Pages.

Firmware Flashing:

Open the C++ firmware in the Arduino IDE.

Update the ssid and password variables with your network credentials.

Update the GitHub fetch URL in the handleRoot function to point to your repository.

Compile and upload to the ESP32-S3.

Technical Specifications

Microcontroller: Espressif ESP32-S3 (Xtensa Dual-Core 32-bit LX7)

Networking Stack: WiFi.h, WebServer.h, HTTPClient.h, WiFiClientSecure.h

Frontend Libraries: Leaflet.js (Mapping)

Telemetry Bounding Box: North: 42, South: 12, West: 25, East: 77 (Middle East & Pakistan)

License

This project is licensed under the MIT License.
