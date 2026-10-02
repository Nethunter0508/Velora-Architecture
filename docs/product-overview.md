# Velora — Product Overview

> **Drive Safer. Help Sooner.**

Velora is a vehicle-independent connected safety, accident-detection, and emergency-response platform designed to retrofit vehicles with a dedicated safety device and connect that device to a mobile application, cloud backend, real-time telemetry pipeline, and future fleet/emergency-response workflows.

This document describes **what Velora is, its important product features, how those features work, and what is implemented versus planned**.

> **Status rule:** **Implemented** means the current development system has the capability. **Foundation** means architecture/backend support exists but an external service, hardware, or production configuration is still required. **Planned** means roadmap work and must not be presented as operational.

---

## 1. Product Definition

Velora connects:

```text
Vehicle
  ↓
Velora Device
  ↓
Sensors + GNSS + Cellular
  ↓
Secure MQTT
  ↓
Velora Backend
  ↓
Detection + Verification + Incident Management
  ↓
Mobile / Fleet Applications
  ↓
Notifications + Authorized Response Integrations
```

The product is intended for:

- individual drivers,
- families,
- taxis and ride-hailing vehicles,
- buses,
- trucks and logistics fleets,
- commercial fleets,
- older vehicles without modern connected-safety systems.

Velora is designed for real-world deployment rather than a hackathon prototype. Hardware, firmware, connectivity, data integrity, security, detection, response, fleet operations, and production validation are deliberately separated into testable stages.

Velora does **not** currently claim universal access to police, ambulance, hospitals, emergency dispatch, towing, or insurance systems. Those capabilities require real technical integrations, agreements, legal review, and operational partnerships.

---

# 2. Problem and Product Need

### Older vehicles may lack connected safety

Modern vehicles increasingly include telematics, automatic crash notification, embedded connectivity, location services, and manufacturer cloud systems. Older vehicles often do not.

Velora's retrofit architecture is intended to add selected connected-safety capabilities without requiring a vehicle replacement.

### An event needs context

A useful safety platform may need to know:

- what happened,
- when it happened,
- where it happened,
- which vehicle/device was involved,
- whether the device was healthy,
- how severe the event appears,
- whether the driver cancelled or confirmed it,
- who should be notified,
- and which response integrations are actually available.

### False positives matter

Hard braking, potholes, speed bumps, sharp turns, GPS loss, and device restarts are not automatically crashes. Velora therefore treats accident detection as a multi-stage workflow rather than one sensor threshold.

---

# 3. Core Safety Workflow

```text
DETECT
  ↓
VERIFY
  ↓
ALERT
  ↓
RESPOND
  ↓
RECOVER
  ↓
LEARN
```

## DETECT

Potential sources:

- automatic sensor detection,
- physical SOS,
- mobile SOS,
- future device/vehicle events,
- future machine-learning detection.

**Current:** mobile Manual SOS and authenticated device events are implemented. Automatic crash detection is not yet activated.

## VERIFY

Potential evidence:

- acceleration,
- rotation,
- speed,
- orientation,
- GNSS,
- event duration,
- post-event movement,
- sensor health,
- user confirmation,
- future ML confidence.

## ALERT

Possible channels:

- in-app incident,
- push notification,
- emergency contact,
- fleet operator,
- authorized external integration.

**Current:** notification records, unread state, preferences, and device registration are implemented. Real FCM delivery is not configured yet.

## RESPOND

Future response can include approved emergency-contact, fleet, roadside, insurance, hospital, ambulance, or emergency-service integrations where real integrations exist.

## RECOVER

Potential workflows:

- incident history,
- investigation,
- roadside assistance,
- insurance,
- repair,
- fleet review.

## LEARN

Validated data can support:

- detection tuning,
- false-positive analysis,
- sensor quality analysis,
- device reliability,
- fleet safety analytics,
- ML dataset creation and evaluation.

---

# 4. Product Components

```text
┌─────────────────────────────────────────────┐
│                 VELORA PLATFORM             │
├─────────────────────────────────────────────┤
│ Flutter Mobile │ Future Web / Fleet         │
├─────────────────────────────────────────────┤
│ REST API       │ Real-Time WebSockets       │
├─────────────────────────────────────────────┤
│ Auth / Sessions│ Notifications              │
├─────────────────────────────────────────────┤
│ MQTT Ingestion │ Device Protocol             │
├─────────────────────────────────────────────┤
│ PostgreSQL     │ Telemetry / Incidents       │
├─────────────────────────────────────────────┤
│ Mosquitto MQTT │ Future Cloud Infrastructure │
├─────────────────────────────────────────────┤
│ ESP32 Device   │ Sensors + Cellular          │
└─────────────────────────────────────────────┘
```

---

# 5. User Accounts and Authentication

## Registration and login — Implemented

Users can register and log in with email/password.

Security characteristics:

- lowercase-normalized email,
- Argon2id password hashing,
- 12–128 character password policy,
- short-lived JWT access tokens,
- minimal JWT claims,
- server-side authorization.

## Refresh sessions — Implemented

Velora uses rotating refresh sessions rather than extending access tokens for hours.

Capabilities include:

- protected server-side refresh-token representation,
- rotation on use,
- single-session logout,
- refresh replay detection,
- refresh-family revocation after replay,
- mobile silent refresh,
- single-flight refresh to prevent refresh races,
- one retry of the original request after a 401.

Future hardening includes absolute session lifetime and user-facing session/device management.

---

# 6. Vehicle Management

Users can create and manage vehicles.

Vehicle information includes:

- name,
- make,
- model,
- year,
- license plate,
- VIN,
- timestamps.

Validation includes:

- realistic year bounds,
- VIN length and I/O/Q character restrictions,
- normalized license plates,
- owner-only access,
- owner-scoped VIN uniqueness.

Vehicle/device relationships are designed to preserve historical incident meaning when records change.

---

# 7. Device Management

A device represents a physical or simulated Velora safety unit assigned to a vehicle.

Device data includes:

- device ID,
- vehicle ID,
- device identifier,
- name,
- status,
- firmware version,
- last-seen timestamp,
- device cryptographic identity/public key information.

Ownership is derived through:

```text
User → Vehicle → Device
```

This prevents a user from accessing another user's device through guessed IDs.

Device lifecycle is intended to evolve through:

```text
Provisioned → Active → Online/Offline → Maintenance/Disabled → Revoked
```

Production manufacturing enrollment and secure provisioning remain future work.

---

# 8. Emergency Contacts

Users can manage:

- contact name,
- phone,
- relationship,
- priority,
- primary-contact status.

The backend supports:

- multiple contacts,
- one primary contact,
- transactional primary-contact handling,
- owner-only access.

**Important:** storing a contact does not currently mean Velora automatically calls or messages that person. A real SMS/voice workflow requires a configured provider and tested operational process.

---

# 9. Medical Profile

A user can maintain one medical profile containing:

- blood group,
- allergies,
- medical conditions,
- medications,
- preferred hospital,
- preferred doctor,
- additional notes.

## Medical-data protection

Medical data is treated as sensitive.

Current protection includes:

- AES-256-GCM encryption,
- dedicated encryption key configuration,
- startup key validation,
- versioned encrypted representation,
- metadata-only audit logging for sensitive operations,
- ownership authorization.

Future emergency sharing requires explicit authorization, data minimization, secure transport, legal/privacy review, and recipient agreements.

---

# 10. Incident Management

Incidents are the central safety object.

An incident can contain:

- user,
- vehicle,
- device,
- status,
- source,
- latitude/longitude,
- occurred-at timestamp,
- created/updated timestamps,
- vehicle-label snapshot,
- device-identifier snapshot.

## Sources

Supported domain sources include:

- `MANUAL_SOS`
- `AUTOMATIC_DETECTION`
- `DEVICE_EVENT`

The automatic source exists in the domain model; it does not mean automatic crash detection is already deployed.

## Status model

The domain supports:

```text
POTENTIAL_EVENT
      ↓
VERIFYING
      ↓
CANCELLED / CONFIRMED
      ↓
EMERGENCY
      ↓
RESOLVED
```

The backend does not allow arbitrary creation of an `EMERGENCY` incident where no real escalation workflow exists.

## Incident history

Incident history supports pagination and preserves useful historical snapshots so later vehicle/device changes do not erase context.

---

# 11. Manual SOS — Implemented

Manual SOS is currently the primary user-facing safety trigger.

Typical mobile flow:

```text
Press SOS
   ↓
Cancellable countdown
   ↓
Cancel or continue
   ↓
Attempt foreground location
   ↓
Create incident
   ↓
Generate notification record
   ↓
Show incident state
```

If location is available, it is attached to the incident. If not, the incident is still recorded and Velora explicitly reports that location was unavailable.

Manual SOS currently does **not** claim:

- police dispatch,
- ambulance dispatch,
- hospital notification,
- emergency-control-room access,
- guaranteed SMS/voice delivery.

---

# 12. Location and Maps

The mobile application currently uses:

- Flutter,
- `flutter_map`,
- `latlong2`,
- OpenStreetMap tiles for development.

Capabilities include:

- current-location display,
- Find Me / recenter,
- incident markers,
- incident marker details,
- incident detail maps,
- missing-location handling.

The location service is abstracted behind an application interface.

The current behavior is foreground-oriented:

- location requested at point of use,
- bounded request duration,
- no continuous timer/stream for the map flow,
- no Android background-location permission,
- no coordinate logging,
- no coordinate fabrication.

Invalid coordinates are rejected rather than clamped.

### Production map considerations

A public development tile service is not automatically a production-scale map solution. Production needs to address provider terms, attribution, usage limits, caching, geocoding, reliability, and commercial licensing where applicable.

---

# 13. Notifications

The notification foundation includes:

- paginated notification history,
- newest-first ordering,
- unread count,
- mark-one-as-read,
- mark-all-as-read,
- notification preferences,
- device registration,
- device-token revocation.

Incident creation/status changes can generate notification records.

Notification failure does not roll back incident persistence.

This preserves:

```text
Incident = authoritative
Notification = delivery mechanism
```

---

# 14. Push Notification Foundation

A real FCM HTTP v1 sender architecture exists, including:

- service-account assertion,
- RS256 signing,
- access-token caching,
- error classification.

However, the current product does **not** pretend that push is configured.

There is:

- no fake Firebase configuration,
- no fake `google-services.json`,
- no claim of universal push delivery.

The mobile app currently uses an unconfigured push implementation when a real Firebase project is unavailable.

Future work:

- Firebase project configuration,
- Android push configuration,
- iOS/APNs,
- background/cold-start notification handling,
- push-tap routing,
- delivery metrics,
- retries and durable queues.

---

# 15. Mobile Application

The mobile app is built with:

- Flutter,
- Riverpod,
- go_router,
- Dio,
- secure token storage.

Current application areas include:

- splash/authentication,
- registration,
- login,
- dashboard,
- vehicles,
- devices,
- emergency contacts,
- medical profile,
- incidents,
- maps,
- manual SOS,
- notifications,
- settings,
- logout/session handling.

The UI is intentionally truthful. For example, it does not claim a device is live when it has never connected and does not present unavailable emergency integrations as operational.

## Mobile real-time behavior

WebSocket events are treated as triggers to refresh authoritative REST state rather than as permanent truth.

This gives the app:

```text
Real-time notification
       +
Authoritative REST state
```

---

# 16. Web / Fleet Application — Planned

The future web/fleet product is intended for:

- fleet managers,
- operators,
- control rooms,
- administrators,
- incident investigators.

Potential features:

- fleet map,
- live vehicle status,
- device health,
- incident queue,
- incident detail,
- live operational events,
- alerts,
- telemetry views,
- safety analytics,
- administrative controls,
- role-based fleet access.

It is not yet represented as a complete production dashboard.

---

# 17. Physical Velora Device

The intended retrofit device includes:

```text
ESP32
 ├── IMU
 ├── GNSS
 ├── Cellular Modem
 ├── SOS Button
 ├── Buzzer
 ├── Status LEDs
 ├── Power / Regulation
 ├── Backup Battery
 └── USB / Programming / Debug
```

Later extensions may include:

- OBD,
- CAN,
- ignition input,
- additional vehicle signals.

OBD/CAN is a future extension and is not assumed to be available in every vehicle.

---

# 18. Hardware Features

## ESP32

Responsible for:

- sensor acquisition,
- device state,
- communications,
- local event generation,
- SOS input,
- status feedback,
- diagnostics,
- secure message signing.

## IMU

Provides:

- acceleration,
- angular velocity,
- orientation-related measurements.

These signals support future harsh-event and crash detection.

## GNSS

Provides:

- latitude,
- longitude,
- speed,
- heading,
- timing.

GNSS quality can degrade due to obstruction, tunnels, antenna problems, or other environmental conditions.

## Cellular

Provides device-to-cloud connectivity independent of the driver's phone.

Production decisions include:

- modem generation,
- carrier,
- SIM/eSIM,
- antenna,
- coverage,
- roaming,
- power consumption,
- reconnect behavior,
- buffering.

## SOS button

Provides a dedicated physical emergency input.

Production firmware should implement:

- debounce,
- deliberate-press handling,
- feedback,
- event creation,
- offline handling,
- secure transmission.

## Buzzer

Can provide local feedback for:

- SOS activation,
- countdown,
- transmission status,
- warnings,
- faults,
- self-test.

## Status LEDs

Can represent:

- power,
- connectivity,
- GNSS state,
- device health,
- incident state,
- fault state.

## Power

A production device must not simply connect an ESP32 development board directly to an automotive rail.

A proper power system requires:

- protection,
- regulation,
- transient handling,
- thermal analysis,
- EMC/EMI evaluation,
- enclosure considerations,
- current measurement.

## Backup battery

Can support continued operation after vehicle power interruption for:

- final incident transmission,
- emergency signaling,
- controlled shutdown,
- short-duration operation.

Battery safety and thermal/charging design require dedicated validation.

---

# 19. Hardware Bring-Up — Step 14

Step 14 is hardware bring-up, not the final firmware.

It covers:

- board configuration,
- GPIO definitions,
- IMU communication,
- GNSS communication,
- SOS input,
- buzzer/LED output,
- cellular interface,
- serial diagnostics,
- hardware self-test,
- sensor initialization,
- sensor fault detection,
- device identity,
- basic firmware/hardware interfaces.

---

# 20. Full Firmware — Step 15

The full firmware roadmap includes:

- sensor sampling,
- device state machine,
- secure MQTT,
- Ed25519 signing,
- sequence numbers,
- boot IDs,
- local buffering,
- reconnect logic,
- watchdog behavior,
- diagnostics,
- configuration,
- power management,
- firmware versioning,
- OTA update strategy,
- secure update controls where supported.

---

# 21. Device Simulator — Implemented

A standalone simulator behaves like a future ESP32 device and publishes realistic device messages.

It supports:

- lifecycle/status,
- heartbeat/status,
- GPS,
- motion/IMU telemetry,
- battery,
- device events,
- simulated SOS.

Messages identify themselves as simulated.

## Scenarios

Current scenarios include:

- idle,
- normal driving,
- stop-and-go,
- hard braking,
- GPS loss,
- connectivity loss,
- reconnect,
- device restart,
- SOS button,
- noisy sensors,
- duplicate messages,
- delayed messages,
- out-of-order messages,
- low battery,
- sensor fault,
- boundary values.

## Determinism

The simulator supports deterministic generation from configuration/seed, useful for:

- regression testing,
- debugging,
- protocol tests,
- reproducible incidents,
- load tests.

The simulator is not proof of physical hardware performance.

---

# 22. Device Protocol

Current MQTT topics:

```text
velora/v1/devices/{deviceId}/status
velora/v1/devices/{deviceId}/telemetry
velora/v1/devices/{deviceId}/location
velora/v1/devices/{deviceId}/events
velora/v1/devices/{deviceId}/commands
```

The protocol contract is shared through:

```text
packages/shared/
```

This prevents the simulator and backend from drifting into separate message formats.

---

# 23. Device Authentication — Implemented Foundation

Device identity is not trusted merely because a device ID appears in an MQTT topic.

The current backend uses per-device Ed25519 message signing:

```text
Device private key
      ↓
Sign complete message
      ↓
MQTT
      ↓
Backend
      ↓
Public-key lookup
      ↓
Signature verification
      ↓
Accept / reject
```

The backend stores the public key.

The device private key is not returned repeatedly or logged by the server.

Key rotation and revocation are part of the current security foundation; secure manufacturing enrollment remains future work.

---

# 24. MQTT Ingestion — Implemented

MQTT ingestion validates device input approximately in this order:

```text
Payload size
  ↓
Topic structure
  ↓
JSON / protocol decoding
  ↓
Device identity
  ↓
Cryptographic signature
  ↓
Device allowance/status
  ↓
Schema validation
  ↓
Channel validation
  ↓
Timestamp validation
  ↓
Idempotency / ordering
  ↓
Persistence
```

This is intentionally a hostile-input boundary.

---

# 25. MQTT Reliability

## Idempotency

Duplicate delivery is expected. Database-level uniqueness/idempotency protects persistence.

## Sequence numbers

Sequence numbers identify:

- normal progression,
- duplicates,
- gaps,
- stale messages,
- out-of-order messages.

Missing data is never invented.

## Boot IDs

A device restart creates a new boot session so sequence `1` after reboot is not confused with sequence `1` from the previous boot.

## Two clocks

The backend distinguishes:

- device event time,
- server ingestion time.

Server ingestion time is authoritative for operational `lastSeenAt`.

## Late messages

Valid late messages can be retained while sequence anomalies remain visible.

---

# 26. Device Events and Emergency Separation

A device event does not automatically equal an emergency.

For example:

```text
Device SOS
   ↓
Authenticate
   ↓
Validate
   ↓
Persist device event
   ↓
Separate safety workflow decides next action
```

This prevents the ingestion layer from silently contacting emergency services.

---

# 27. Real-Time WebSockets

WebSockets provide real-time updates after authoritative persistence:

```text
MQTT
 ↓
Validate
 ↓
Persist
 ↓
Commit
 ↓
Internal event
 ↓
WebSocket
```

## Authentication

Connections use the same access-token identity model as the REST API.

Invalid connections are refused. Expiring authentication is not silently treated as permanently valid.

## Authorization

Users cannot subscribe to another user's vehicle/device streams.

## Delivery model

Velora does not claim exactly-once WebSocket delivery.

Instead:

```text
WebSocket event
      +
REST authoritative state
```

If an event is missed, reconnect and refetch state.

## Telemetry thinning

Raw high-frequency IMU data is not intended to be broadcast to every UI client. Real-time UI traffic is kept operationally useful while detailed telemetry remains a storage/analysis concern.

---

# 28. Backend Architecture

The backend stack is:

- NestJS,
- TypeScript,
- Prisma,
- PostgreSQL,
- Mosquitto/MQTT,
- WebSockets.

The REST API provides authoritative operations for:

- authentication,
- vehicles,
- devices,
- emergency contacts,
- medical profiles,
- incidents,
- notifications,
- preferences,
- notification devices,
- sessions.

PostgreSQL stores domain and telemetry information.

---

# 29. Data Model

Conceptually:

```text
User
 ├── Vehicles
 │     └── Devices
 │           ├── Telemetry
 │           ├── Locations
 │           ├── Device Events
 │           └── Device Identity
 │
 ├── Emergency Contacts
 ├── Medical Profile
 ├── Incidents
 ├── Notifications
 ├── Sessions
 └── Audit Records
```

PostgreSQL is authoritative. Real-time transports do not replace persistence.

---

# 30. Crash Detection — Planned

Automatic crash detection is a major future capability.

Development path:

```text
Sensor Data
   ↓
Signal Validation
   ↓
Rule-Based Detection
   ↓
Verification
   ↓
Controlled Dataset
   ↓
Machine Learning
   ↓
Model Evaluation
   ↓
Production Monitoring
```

## Rule-based detection

Potential signals:

- longitudinal/lateral/vertical acceleration,
- gyroscope,
- speed,
- heading,
- orientation,
- sudden stop,
- post-event immobility,
- GNSS behavior.

The first detector should be understandable, deterministic, versioned, and testable.

## Verification

A candidate event may be evaluated using:

- multi-sensor agreement,
- severity,
- duration,
- speed,
- post-event movement,
- device health,
- driver confirmation.

## ML

ML may later produce:

- crash probability,
- event classification,
- severity estimate,
- confidence.

Models require real datasets, validation, monitoring, drift detection, versioning, and rollback. Offline accuracy alone is not sufficient for safety-critical deployment.

---

# 31. Sensor Dataset — Planned

The future dataset can include:

- accelerometer,
- gyroscope,
- GNSS,
- speed,
- orientation,
- timestamps,
- device state,
- event labels.

Dataset governance should include:

- provenance,
- labeling methodology,
- privacy/consent,
- quality checks,
- versioning,
- train/validation/test separation.

Safe controlled collection is required; dangerous intentional crashes are not an acceptable test methodology.

---

# 32. Emergency-Response Strategy

Potential response recipients include:

- emergency contacts,
- fleet control rooms,
- roadside/towing partners,
- insurers,
- hospitals,
- ambulance providers,
- police/emergency dispatch.

A recipient is considered integrated only when the technical integration, authorization, operational agreement, testing, and availability model actually exist.

A future emergency data package may contain appropriately minimized information such as:

```text
Incident ID
Vehicle / device identity
Event time
Location + location quality
Direction
Severity/confidence
Authorized emergency-contact context
Authorized medical information
```

The exact package must be determined by recipient requirements, privacy law, consent, and operational agreements.

---

# 33. Security

Current security foundations include:

- Argon2id password hashing,
- short-lived JWT access tokens,
- rotating refresh sessions,
- secure mobile token storage,
- owner authorization,
- device cryptographic authentication,
- encrypted medical data,
- protected push tokens,
- sensitive-operation auditing,
- authentication rate limiting.

Production hardening still requires:

- TLS everywhere,
- MQTT ACLs,
- secure secret management,
- key rotation,
- centralized rate limiting,
- penetration testing,
- security monitoring,
- dependency monitoring,
- backup/disaster recovery,
- incident-response procedures.

---

# 34. Privacy and Data Governance

Velora can process:

- precise location,
- movement history,
- vehicle identity,
- incident history,
- medical information,
- emergency contacts.

Privacy principles should include:

- data minimization,
- purpose limitation,
- authorization,
- encryption,
- retention controls,
- auditability,
- transparency,
- controlled emergency access,
- appropriate deletion/export processes.

Production privacy requirements depend on jurisdiction and deployment model.

---

# 35. Reliability and Failure Handling

| Failure | Expected behavior |
|---|---|
| GPS unavailable | Record event without fabricated coordinates |
| Cellular unavailable | Device detects outage and reconnects; future firmware can buffer appropriate data |
| MQTT unavailable | Reconnect and recover |
| Duplicate message | Database idempotency prevents duplicate records |
| Out-of-order message | Preserve valid data and record anomaly |
| Device restart | New boot ID |
| WebSocket disconnect | Reconnect and refetch REST state |
| Push failure | Incident remains persisted |
| Invalid signature | Reject device message |
| Disabled device | Reject device message |
| Cross-user API request | Reject authorization |
| Cross-user WebSocket subscription | Refuse subscription |
| Invalid coordinates | Reject rather than clamp/fabricate |
| Backend restart | PostgreSQL remains authoritative |

---

# 36. Observability — Production Requirement

Production should monitor:

- API latency,
- authentication failures,
- MQTT acceptance/rejection,
- signature failures,
- connected devices,
- device last-seen state,
- telemetry rate,
- sequence gaps,
- duplicate rate,
- GNSS quality,
- notification delivery,
- WebSocket connections,
- incident lifecycle timing,
- crash-detection metrics,
- hardware faults.

A mature deployment needs:

```text
Metrics
Logs
Traces
Dashboards
Alerts
Audit records
```

Current development testing and in-memory measurements are not a substitute for production observability.

---

# 37. Testing

Velora uses multiple testing layers.

## Unit

Covers:

- business logic,
- validation,
- authentication,
- cryptographic behavior,
- protocol,
- simulator generation,
- notifications,
- WebSockets.

## Integration

Covers:

- PostgreSQL,
- Mosquitto,
- API,
- device simulator,
- MQTT ingestion,
- WebSocket delivery.

## End-to-end

Representative path:

```text
Simulator
  ↓
Signed MQTT
  ↓
Mosquitto
  ↓
MQTT ingestion
  ↓
PostgreSQL
  ↓
Internal event
  ↓
WebSocket
  ↓
Authenticated client
```

## Failure testing

Scenarios include:

- bad credentials,
- session expiry,
- offline API,
- duplicate messages,
- delayed messages,
- out-of-order messages,
- GPS loss,
- connectivity loss,
- broker restart,
- device restart,
- sensor fault,
- low battery,
- malformed input,
- unauthorized subscriptions,
- cross-account access.

---

# 38. Production Hardware and Manufacturing

A production device requires more than an ESP32 development board.

Lifecycle:

```text
Component selection
  ↓
Electrical design
  ↓
PCB
  ↓
Prototype
  ↓
Bring-up
  ↓
Environmental / EMC testing
  ↓
Pilot manufacturing
  ↓
Production test fixture
  ↓
Secure device provisioning
  ↓
Final assembly
  ↓
Field deployment
```

Validation must cover:

- automotive voltage/transients,
- thermal conditions,
- vibration,
- enclosure,
- cellular,
- GNSS,
- antenna design,
- power,
- battery,
- EMC/EMI,
- connector reliability,
- long-duration operation.

Production provisioning should establish unique device identity, cryptographic material, firmware/hardware revision, and lifecycle state.

---

# 39. Deployment Architecture

Development currently uses Dockerized PostgreSQL and Mosquitto.

A production direction can evolve toward:

```text
Internet / Cellular
       ↓
Load Balancer
       ↓
┌──────┼──────┐
API   API    API
└──────┼──────┘
       ↓
Shared Event Infrastructure
       ↓
┌──────┼───────────┐
Postgres       MQTT Infrastructure
       ↓
Storage / Analytics / ML
       ↓
Push + External Integrations
```

The exact cloud provider and topology should follow actual reliability, traffic, cost, regulatory, and team requirements.

---

# 40. Scaling Strategy

### Development

- one PostgreSQL,
- one MQTT broker,
- one API process,
- simulator,
- local mobile app.

### Pilot

- managed/high-availability PostgreSQL,
- secured MQTT,
- monitored API,
- real push,
- backups,
- alerting.

### Regional production

- multiple API instances,
- load balancing,
- shared event bus,
- centralized rate limiting,
- durable queues/outbox,
- monitored MQTT infrastructure,
- observability.

### Global production

Potentially:

- regional infrastructure,
- data residency,
- regional cellular strategy,
- localized response integrations,
- multi-region recovery,
- large-scale telemetry processing.

The system should scale based on measured workloads rather than premature complexity.

---

# 41. Business Model

Potential models include:

## Hardware + subscription

Device purchase/lease plus recurring platform fee for connectivity and cloud services.

## Fleet SaaS

Pricing based on vehicle/device count, feature tier, telemetry volume, or support.

## OEM / white-label

Provide device, backend, APIs, and safety workflows to mobility/vehicle partners.

## Insurance partnerships

Potential uses include risk analytics, safety programs, incident evidence, and claims workflows, subject to commercial, actuarial, legal, and privacy validation.

---

# 42. Target Segments

### Individual drivers

- retrofit safety,
- Manual SOS,
- incident history,
- future automatic detection,
- emergency contacts.

### Families

- multiple vehicles,
- safety visibility,
- device health,
- alerts.

### Taxi / ride-hailing

- driver SOS,
- tracking,
- incident management,
- fleet monitoring.

### Bus operators

- vehicle location,
- emergency buttons,
- device health,
- incident monitoring.

### Truck / logistics

- location,
- device health,
- harsh-event telemetry,
- accident detection,
- driver safety,
- analytics.

### Commercial fleets

- mixed fleet visibility,
- centralized incidents,
- device management,
- operational analytics.

---

# 43. Competitive Landscape

The connected-vehicle market already includes:

- Geotab,
- Samsara,
- Motive,
- Lytx,
- Verizon Connect,
- regional vehicle-tracking providers,
- OEM connected-vehicle services.

Velora does not claim these systems do not exist or that Velora is automatically superior.

The intended product direction is:

> **A retrofit-oriented safety platform starting with an independent safety device and evolving toward individual-driver, fleet, and authorized emergency-response workflows.**

Any competitive advantage must ultimately be demonstrated through product usability, detection performance, hardware reliability, response reliability, deployment economics, integrations, compliance, and customer outcomes.

---

# 44. India and Global Expansion

India is a major potential market because of its large and diverse vehicle population, commercial fleets, buses, taxis, logistics, and growing connected-vehicle adoption.

Commercial deployment can involve requirements around:

- vehicle tracking,
- emergency buttons,
- telecom/cellular certification,
- privacy/data protection,
- radio/hardware certification,
- automotive requirements,
- state or vehicle-category-specific rules.

Global deployment additionally requires:

- cellular-band compatibility,
- carrier strategy,
- SIM/eSIM,
- data residency,
- privacy laws,
- emergency-number systems,
- mapping providers,
- notification providers,
- vehicle standards,
- hardware certifications,
- local response partnerships.

Regulatory and legal review is required before commercial claims.

---

# 45. Product Roadmap

```text
1. Development Environment
2. Project Foundation
3. Docker Infrastructure
4. Backend Foundation
5. Authentication
6. Vehicle / Device / Incident APIs
7. Flutter Mobile
7.5 Refresh Token / Session Management
8. Maps & Location
9. Manual SOS
10. Notifications
11. Device Simulator
12. MQTT Backend
13. Real-Time WebSockets
14. ESP32 Hardware
15. Firmware
16. Rule-Based Crash Detection
17. Verification / Emergency Workflow
18. Web / Fleet Dashboard
19. Sensor Dataset
20. Machine Learning
21. Security / Reliability
22. Power / Backup Battery
23. Controlled Testing
24. Pilot
25. Production / External Integrations
```

---

# 46. Current Product Status

## Implemented

- project foundation,
- Docker PostgreSQL,
- Mosquitto development broker,
- NestJS backend,
- Prisma/PostgreSQL,
- authentication,
- JWT access tokens,
- rotating refresh sessions,
- vehicle CRUD,
- device CRUD,
- emergency contacts,
- encrypted medical profile,
- incident APIs/history,
- Flutter mobile application,
- maps/location,
- Manual SOS,
- notification foundation,
- device simulator,
- shared device protocol,
- Ed25519 device authentication,
- MQTT telemetry/location/event ingestion,
- sequence/boot tracking,
- real-time WebSockets,
- extensive unit/integration/E2E testing.

## Foundation / Not Yet Fully Operational

- real FCM push delivery,
- production MQTT security,
- production cloud deployment,
- production map provider,
- device provisioning/manufacturing,
- fleet web application,
- durable distributed event infrastructure.

## Planned

- physical ESP32 hardware,
- full firmware,
- rule-based automatic crash detection,
- verification workflow,
- fleet dashboard,
- sensor dataset,
- ML crash detection,
- production power/backup battery,
- controlled safety testing,
- pilot,
- emergency/roadside/insurance integrations,
- production emergency-response partnerships.

---

# 47. Important Product Limitations

Velora does not yet claim:

- universal automatic crash detection,
- production ML crash detection,
- guaranteed emergency-service dispatch,
- guaranteed ambulance/police/hospital response,
- guaranteed emergency-contact calls/SMS,
- global cellular coverage,
- production certification of the prototype hardware,
- universal OBD/CAN support,
- production-scale public map tiles,
- crash-detection validation across every vehicle type.

These are explicit engineering, validation, integration, and operational requirements.

---

# 48. Product Success Criteria

Velora should eventually demonstrate:

### Device

- stable power,
- reliable sensor acquisition,
- reliable GNSS,
- reliable cellular,
- secure identity,
- safe recovery after faults.

### Platform

- authenticated ingestion,
- no cross-user data access,
- durable incidents,
- reliable real-time updates,
- observable failures,
- recoverable infrastructure.

### Detection

- measured false-positive rate,
- measured false-negative risk,
- vehicle/environment diversity,
- controlled validation,
- model/rule versioning.

### Response

- real integrations,
- tested delivery,
- clear failure behavior,
- operational ownership,
- auditability.

### Product

- understandable UX,
- fast incident visibility,
- trustworthy location handling,
- fleet usability,
- sustainable hardware/cloud economics.

---

# 49. End-to-End Product Flow

## Normal driving

```text
Vehicle
  ↓
Velora Device
  ↓
IMU / GNSS / Device State
  ↓
Secure telemetry
  ↓
MQTT
  ↓
Velora ingestion
  ↓
PostgreSQL
  ↓
Real-time state
  ↓
Mobile / Fleet UI
```

## Manual SOS

```text
Driver
  ↓
Mobile SOS
  ↓
Countdown
  ↓
Location attempt
  ↓
Incident API
  ↓
PostgreSQL
  ↓
Notification + WebSocket
  ↓
Incident visible
```

## Future automatic crash

```text
Vehicle motion
  ↓
IMU + GNSS
  ↓
ESP32 firmware
  ↓
Signed telemetry
  ↓
MQTT
  ↓
Ingestion
  ↓
Crash detector
  ↓
Potential event
  ↓
Verification
  ↓
Confirmed event
  ↓
Configured response
  ↓
Recovery / learning
```

---

# 50. Long-Term Vision

```text
                         VELORA
                           │
          ┌────────────────┼─────────────────┐
          │                │                 │
       DEVICE           MOBILE            WEB/FLEET
          │                │                 │
          └────────────────┼─────────────────┘
                           │
                     VELORA CLOUD
                           │
        ┌──────────────────┼──────────────────┐
        │                  │                  │
    TELEMETRY           INCIDENTS          ALERTS
        │                  │                  │
        └──────────────────┼──────────────────┘
                           │
                  DETECTION + ML
                           │
                    VERIFICATION
                           │
                    RESPONSE HUB
                           │
       ┌─────────────┬────┼─────┬──────────────┐
       │             │    │     │              │
   Contacts       Fleet Roadside Insurance Emergency
                              Assistance       Partners
```

The central product principle remains:

> **Sense accurately. Authenticate data. Verify events. Alert reliably. Respond only through real, authorized workflows. Learn from validated evidence.**

---

# 51. Final Summary

Velora is not just a crash-detection algorithm and not just a GPS tracker.

It is a layered safety platform:

1. **Physical sensing** — IMU, GNSS, cellular, SOS, local feedback and power.
2. **Secure device identity** — per-device cryptographic authentication.
3. **Device communication** — MQTT with validation, signatures, sequence tracking and reconnect behavior.
4. **Cloud ingestion** — hostile-input validation and PostgreSQL persistence.
5. **Application backend** — accounts, vehicles, devices, incidents, contacts, medical data, notifications and sessions.
6. **Mobile application** — management, maps, incident history, Manual SOS and notifications.
7. **Real-time layer** — authenticated WebSockets backed by authoritative REST state.
8. **Crash detection** — future rule-based detection followed by validated ML.
9. **Verification** — separates potential events from confirmed emergencies.
10. **Response** — future authorized emergency-contact, fleet, roadside, insurance and emergency-service integrations.
11. **Fleet platform** — future web/control-room operations.
12. **Learning** — datasets, evaluation, model improvement and operational analytics.
13. **Production engineering** — hardware validation, security, observability, scaling, manufacturing, compliance and recovery.

The ultimate reliability chain is:

```text
REAL SENSOR
    ↓
CORRECT DEVICE IDENTITY
    ↓
AUTHENTIC MESSAGE
    ↓
VALIDATED DATA
    ↓
AUTHORITATIVE STORAGE
    ↓
CORRECT DETECTION
    ↓
CAREFUL VERIFICATION
    ↓
RELIABLE ALERT
    ↓
REAL AUTHORIZED RESPONSE
    ↓
MEASURABLE RECOVERY
```

That chain—not merely a working demo—is the standard required for Velora to become a dependable real-world safety product.
