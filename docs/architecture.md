# System Architecture — IOT Smart Farm

## 1. Overview

**IOT Smart Farm** is an agricultural IoT monitoring, automation, and early-warning system designed for greenhouses (plastic planting houses) and farms operating under real-world field constraints in Syria, including unstable grid electricity, intermittent internet connectivity, harsh weather, and the need for water-efficient drip irrigation and rapid hazard detection.

The architecture is organized into four main tiers:
1. **Edge Nodes (`ESP32-CAM` + Sensors + Actuators)** — Solar-powered field units that form/join a local network, collect environmental telemetry, capture periodic images, detect smoke/fire, and actuate drip irrigation pumps and heating devices.
2. **Backbone Nodes (Relay / Gateway)** — Bridge local field nodes to either an on-site **Local Server** over LAN or external connectivity via a **SIM module**, with local queuing/buffering when links are down.
3. **Server Backend & Dashboard** — Ingests telemetry/images/alerts, evaluates automation rules, stores history, and exposes a web dashboard and APIs.
4. **Mobile Application** — Provides farmers with live greenhouse status, camera snapshots, urgent alerts (fire, smoke, temperature, rain/irrigation), and manual/automated control.

---

## 2. High-Level Data & Control Flow

```text
+-------------------------------------------------------------------+
|                        FIELD / GREENHOUSE                         |
|                                                                   |
|  [Solar Panel + Battery]                                          |
|           |                                                       |
|           v                                                       |
|  +-----------------------+     +-------------------------------+  |
|  |   Edge Node (ESP32-CAM) | --> | Sensors: Temp, Humidity,      |  |
|  |  - Local Network      |     |          Light, Rain          |  |
|  |  - Periodic Photos    |     +-------------------------------+  |
|  |  - Smoke/Fire Check   |                                         |
|  |  - Power Mgmt         |     +-------------------------------+  |
|  +-----------------------+ --> | Actuators: Drip Pump, Heater  |  |
|           |                   +-------------------------------+  |
|           | Local Network (AP / Wi-Fi / Mesh-style link)          |
|           v                                                       |
|  +-----------------------+                                        |
|  |     Backbone Node     |                                        |
|  |  - Ingress & Buffering|                                        |
|  |  - Link Failover      |                                        |
|  +-----------+-----------+                                        |
|              |                                                    |
+--------------|----------------------------------------------------+
               |
     +---------+---------+
     |                   |
     v (On-Site LAN)     v (Cellular SIM)
+------------+    +------------------+
|Local Server|    | Internet / Cloud |
|  & DB/UI   |    |  Server & Alerts |
+------------+    +------------------+
     |                     |
     +----------+----------+
                |
                v
     +---------------------+
     |     Mobile App      |
     | (Farmer Dashboard,  |
     |  Photos, Controls,  |
     |  Push/Alert View)   |
     +---------------------+