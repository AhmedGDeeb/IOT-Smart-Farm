# Periodic Imaging & Smoke / Fire Detection — IOT Smart Farm

## 1. Purpose

Greenhouses (plastic planting houses) and agricultural plots face serious risks from sudden temperature spikes, electrical or heating unit malfunctions, dry vegetation, and fires—especially at night or when farmers are off-site.

This document describes how **IOT Smart Farm** combines:
- **Periodic visual capture** (`camera_capture.cpp`) to monitor greenhouse and crop conditions over time.
- **Smoke and fire detection** (`fire_smoke_detect.cpp`) paired with environmental telemetry (`sensors.cpp`) to raise timely alerts with visual evidence.
- **Server & mobile alerting** (`fire_detector_service.py`, `notification_service.py`, and the mobile app) so farmers can verify conditions quickly and take appropriate action.

---

## 2. Operational Modes

### 2.1 Periodic Greenhouse Imaging
- The solar-powered `ESP32-CAM` edge node captures scheduled still images of the greenhouse interior/perimeter at configurable intervals (adjusted to balance visual coverage against solar/battery power and bandwidth).
- Images are compressed (e.g., JPEG) and sent through the local network via the backbone node to the server (`server/api/routes/images.py`) and surfaced in the dashboard and mobile app (`CameraGalleryScreen`).
- Helps farmers track crop growth, canopy/soil appearance, and general greenhouse state without constant physical travel.

### 2.2 Smoke & Fire Detection Pipeline
1. **Detection & Trigger (`fire_smoke_detect.cpp`):**
   - Evaluates camera frames and correlates with environmental context (such as sudden temperature rise or abnormal readings from `sensors.cpp`) to flag potential smoke or fire conditions.
   - Captures an immediate snapshot/event frame when an anomaly is suspected so visual evidence accompanies the alert.
2. **Transport & Buffering (`local_network.cpp`, `offline_buffer.cpp`):**
   - Prioritizes delivery of urgent alert events and snapshot frames through the backbone node toward the Local Server and/or SIM link.
   - Buffers events/frames locally when a link is temporarily unavailable and flushes them as soon as connectivity returns.
3. **Verification & Notification (`fire_detector_service.py`, `notification_service.py`):**
   - Logs the event in `server/api/routes/alerts.py` and `images.py`, applies confirmation/cooldown rules to reduce duplicate or false alarms, and sends notifications to the farmer's mobile app (`AlertBanner` / `AlertsHistoryScreen`).

---

## 3. Practical Field Considerations (Greenhouses & Solar Operation)

- **Lighting & Glare:** Bright sunlight passing through plastic sheeting or harsh night illumination/shadows can cause reflections or false visual triggers; thresholds and sampling intervals should be tuned to real greenhouse lighting.
- **Dust, Condensation & Lens Obstruction:** High humidity, condensation, or dust in agricultural environments can blur or block the camera lens; periodic inspection and cleaning are part of routine maintenance.
- **Power & Bandwidth Trade-Offs:** Frequent high-resolution image uploads over cellular links consume more battery and data than small telemetry packets. Periodic capture intervals and event-driven high-priority uploads help conserve both energy and bandwidth.
- **Safety First:** Automated alerts and camera snapshots assist human decision-making and early response; they do not replace proper electrical safety, safe heater installation, or emergency response procedures.

---

## 4. Related Modules

- `firmware/edge-node/src/camera_capture.cpp` — Periodic image capture and compression
- `firmware/edge-node/src/fire_smoke_detect.cpp` — Smoke/fire detection and event trigger logic
- `firmware/edge-node/src/sensors.cpp` — Temperature, humidity, light, and rain telemetry
- `firmware/backbone-node/src/offline_buffer.cpp` — Outage queuing and replay
- `server/api/services/fire_detector_service.py` — Event handling and escalation logic
- `server/api/services/notification_service.py` — Farmer alert delivery
- `mobile-app/src/screens/CameraGalleryScreen.dart` — Photo history and alert inspection

---

## 5. Implementation Status Note

This document outlines the imaging and smoke/fire detection workflow for **IOT Smart Farm**. Proprietary source code implementations are maintained privately and omitted from public source files.