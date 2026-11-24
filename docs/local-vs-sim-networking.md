# Local Network vs. SIM Module Connectivity — IOT Smart Farm

## 1. Why Dual-Path Connectivity Matters

In many agricultural zones and greenhouses, internet access can be unreliable, slow, or costly, while grid power outages are frequent. Relying solely on cloud internet connectivity can leave a farm blind or unable to act during critical moments.

**IOT Smart Farm** addresses this by separating field communication into:
1. **Local Field Network & Local Server Path:** Keeps greenhouse monitoring, automation, and local dashboard/server operations running locally even when external internet is down.
2. **SIM Module Path:** Enables cellular/remote connectivity for off-site monitoring, notifications, and data exchange when internet access is available or required.
3. **Offline Buffering & Failover:** Queues telemetry, images, and alert events locally during temporary link interruptions and syncs them once connectivity resumes.

---

## 2. Connectivity Layers

### 2.1 Edge-to-Backbone Local Network (`local_network.cpp` & `edge_ingress.cpp`)
- The solar-powered `ESP32-CAM` edge node establishes or joins a **local network** to exchange sensor readings (temperature, humidity, light, rain), periodic snapshots, smoke/fire alerts, and pump/heater control commands with the backbone node.
- Keeping initial field transport local reduces dependence on external routers or internet gateways inside the greenhouse area.

### 2.2 Local Server Path (`local_server_client.cpp`)
- The backbone node communicates with an on-site **Local Server** (e.g., a local mini-PC/host running the `server/` stack on the local network).
- **Advantages:**
  - Works without external internet access.
  - Low-latency local access to telemetry, camera snapshots, and automation rules (`automation_engine.py`).
  - Avoids recurring cellular data costs for high-frequency readings or local images.

### 2.3 SIM Module Path (`sim_module.cpp`)
- The backbone node can connect via a **SIM module** when external/remote connectivity is needed (for example, sending alerts or syncing status to a remote server/mobile client when the farmer is away from the farm).
- **Practical considerations:**
  - Cellular signal strength and coverage vary by rural location and terrain; an external antenna and placement checks help maintain link reliability.
  - Cellular data plans and power draw during transmission make it best to prioritize compact telemetry and high-priority alert/snapshot events over heavy continuous streaming.

---

## 3. Offline Buffering & Recovery (`offline_buffer.cpp`)

When neither the Local Server nor the SIM network link is immediately reachable:
1. **Local Queueing:** The backbone node stores outgoing payloads/events in local non-volatile storage (`offline_buffer.cpp`).
2. **Priority Handling:** Urgent alerts (such as smoke/fire or critical threshold warnings) and recent snapshots are prioritized ahead of routine background logs where supported.
3. **Automatic Replay:** Once the Local Server or SIM connection is restored, buffered items are transmitted and acknowledged so historical records are not lost.

---

## 4. Related Modules

- `firmware/edge-node/src/local_network.cpp` — Edge local network setup and transport
- `firmware/backbone-node/src/edge_ingress.cpp` — Ingress from edge nodes
- `firmware/backbone-node/src/local_server_client.cpp` — Local Server communication
- `firmware/backbone-node/src/sim_module.cpp` — SIM module communication
- `firmware/backbone-node/src/offline_buffer.cpp` — Outage storage and replay
- `firmware/backbone-node/src/command_relay.cpp` — Control command forwarding
- `server/api/services/notification_service.py` — Alert delivery to the mobile app

---

## 5. Implementation Status Note

This document describes the networking and connectivity design for **IOT Smart Farm**. Proprietary source code implementations are maintained privately and omitted from public source files.