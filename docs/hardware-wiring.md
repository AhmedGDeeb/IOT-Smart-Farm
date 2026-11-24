# Hardware & Wiring Overview — IOT Smart Farm

## 1. Purpose

This document provides a high-level hardware and wiring guide for **IOT Smart Farm**, covering the solar-powered edge node (`ESP32-CAM`), field sensors, actuator switching (drip irrigation pump and heating device), and the backbone gateway node.

> **Note:** Exact GPIO assignments depend on the specific `ESP32-CAM` / ESP32 board revision and relay/sensor breakout modules used in your build. Always verify pin strapping and voltage levels before powering on hardware.

---

## 2. Subsystems Overview

### 2.1 Edge Node (`firmware/edge-node`)
- **Controller & Camera:** `ESP32-CAM` module (handles periodic imaging, smoke/fire frame checks, local networking, sensor polling, and actuator control).
- **Power Supply:** Solar panel + charge controller + rechargeable battery + regulated power rail for `ESP32-CAM` and peripherals (`power_management.cpp`).
- **Sensors (`sensors.cpp`):**
  - **Temperature & Humidity:** Environmental sensor inside/around the greenhouse (e.g., DHT22 / AM2302 or SHT/BME series, matching your `sensors.cpp` driver).
  - **Light Sensor:** Ambient light / LDR or digital light sensor to track daylight/shading conditions.
  - **Rain Sensor:** Rain detection board/probe to detect precipitation events.
- **Actuators (`actuators.cpp`):**
  - **Water Pump (Drip Irrigation):** Switched via an opto-isolated relay module or contactor driver rated for the pump's voltage/current.
  - **Heating Device:** Switched via an appropriate relay/contactor stage sized for the greenhouse heater load.

### 2.2 Backbone Node (`firmware/backbone-node`)
- **Gateway / Relay Controller:** ESP32/gateway node receiving telemetry, images, and alerts from edge nodes (`edge_ingress.cpp`) and forwarding commands back (`command_relay.cpp`).
- **Local Server Link (`local_server_client.cpp`):** Connects over local Wi-Fi/LAN to an on-site Local Server.
- **Cellular Link (`sim_module.cpp`):** Interfaces with a SIM module (UART/AT or data link) for remote/cellular connectivity and notifications, with local buffering (`offline_buffer.cpp`).

---

## 3. Conceptual Wiring Diagram

```text
+-----------------------+       +-------------------------------+
|  Solar Panel + Charge | ----> | Regulated Power Rail          |
|  Controller + Battery |       | (3.3V / 5V + Battery Sense)   |
+-----------------------+       +---------------+---------------+
                                                |
                   +----------------------------+----------------------------+
                   |                            |                            |
                   v                            v                            v
        +---------------------+      +---------------------+      +---------------------+
        |     ESP32-CAM       |      |  Sensors            |      |  Relay / Switch     |
        |  - Camera CSI       | <--> |  - Temp & Humidity  |      |  Stage (Isolated)   |
        |  - Local Network    |      |  - Light            | ---> |  - Drip Water Pump  |
        |  - Fire/Smoke Check |      |  - Rain             |      |  - Heating Device   |
        +----------+----------+      +---------------------+      +---------------------+
                   |
                   | Local Network (AP / Wi-Fi / Local Link)
                   v
        +---------------------+
        |    Backbone Node    |
        |  - Local Server LAN |
        |  - SIM Module Link  |
        |  - Offline Buffer   |
        +---------------------+