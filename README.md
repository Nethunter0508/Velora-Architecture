# Velora

> **Drive Safer. Help Sooner.**

Velora is a vehicle-independent connected safety, accident-detection, and emergency-response platform designed to retrofit vehicles with a dedicated safety device and connect that device to a mobile application, cloud backend, real-time telemetry pipeline, and—eventually—fleet operations and emergency-response workflows.

Velora is designed for **real-world deployment**, not as a hackathon prototype. The architecture is intentionally being built in stages so that hardware, firmware, connectivity, data integrity, security, crash detection, emergency workflows, fleet operations, and production validation can each be tested independently.

> **Current-status principle:** This README distinguishes implemented capabilities from planned capabilities. A feature described in the roadmap is not represented as operational merely because its architecture has been designed.

---

## Table of Contents

1. [Executive Summary](#executive-summary)
2. [The Problem](#the-problem)
3. [Why Existing Approaches Leave Gaps](#why-existing-approaches-leave-gaps)
4. [The Velora Solution](#the-velora-solution)
5. [Who Velora Is For](#who-velora-is-for)
6. [Core Product Workflow](#core-product-workflow)
7. [Product Components](#product-components)
8. [System Architecture](#system-architecture)
9. [End-to-End Data Flow](#end-to-end-data-flow)
10. [Mobile Application](#mobile-application)
11. [Web / Fleet Application](#web--fleet-application)
12. [Physical Velora Device](#physical-velora-device)
13. [Device Hardware Architecture](#device-hardware-architecture)
14. [Device-to-Cloud Communication](#device-to-cloud-communication)
15. [Backend Architecture](#backend-architecture)
16. [MQTT Ingestion](#mqtt-ingestion)
17. [Real-Time WebSockets](#real-time-websockets)
18. [Data Model](#data-model)
19. [Security](#security)
20. [Privacy and Data Governance](#privacy-and-data-governance)
21. [Crash Detection Strategy](#crash-detection-strategy)
22. [Emergency-Response Strategy](#emergency-response-strategy)
23. [Notifications](#notifications)
24. [Reliability and Failure Handling](#reliability-and-failure-handling)
25. [Deployment Architecture](#deployment-architecture)
26. [Production Architecture](#production-architecture)
27. [Scaling Strategy](#scaling-strategy)
28. [Cost Model](#cost-model)
29. [Business Model](#business-model)
30. [Market and Target Segments](#market-and-target-segments)
31. [Competitive Landscape](#competitive-landscape)
32. [Velora's Positioning](#veloras-positioning)
33. [India and Regulatory Considerations](#india-and-regulatory-considerations)
34. [Global Expansion](#global-expansion)
35. [Go-to-Market Strategy](#go-to-market-strategy)
36. [Manufacturing and Hardware Strategy](#manufacturing-and-hardware-strategy)
37. [Implementation Roadmap](#implementation-roadmap)
38. [Current Technical Status](#current-technical-status)
39. [Production Readiness Checklist](#production-readiness-checklist)
40. [Risks and Open Problems](#risks-and-open-problems)
41. [Repository Structure](#repository-structure)
42. [Developer Setup](#developer-setup)
43. [Testing Strategy](#testing-strategy)
44. [Engineering Principles](#engineering-principles)
45. [Long-Term Vision](#long-term-vision)
46. [References and Further Reading](#references-and-further-reading)
47. [License](#license)

---

# Executive Summary

Velora is intended to provide a safety layer for vehicles that may not already have advanced connected-safety systems.

The platform combines:

- a retrofit IoT safety device,
- accelerometer and gyroscope data,
- GNSS/GPS positioning,
- vehicle/device telemetry,
- cellular connectivity,
- a physical SOS button,
- a secure cloud backend,
- a mobile application,
- real-time telemetry,
- incident history,
- future crash-detection algorithms,
- future fleet operations,
- and future emergency-response integrations where lawful, technically possible, and supported by real partnerships.

The long-term system is intended to work across:

- private cars,
- taxis,
- buses,
- trucks,
- commercial fleets,
- older vehicles,
- mixed fleets,
- public-service vehicles,
- and other compatible road vehicles.

The product is **vehicle-independent** in the sense that its core safety device can operate without requiring a particular vehicle manufacturer or a specific modern infotainment platform.

---

# The Problem

Road accidents create several interconnected problems.

## 1. Detection can be difficult

After a serious collision:

- occupants may be unable to use a phone,
- a phone may be damaged,
- the driver may be unconscious,
- the vehicle may be in a low-connectivity area,
- witnesses may not know what happened,
- and there may be no reliable digital record of the event.

A connected device can provide another sensing path.

## 2. Location is critical

Knowing that an incident occurred is not enough.

A response system may need:

- latitude,
- longitude,
- timestamp,
- vehicle/device identity,
- direction,
- recent movement,
- and confidence/quality information.

## 3. Older vehicles may lack modern connected safety

New vehicles increasingly include:

- telematics,
- automatic crash notification,
- connected services,
- advanced driver assistance,
- embedded cellular connectivity,
- and manufacturer cloud services.

Older vehicles often do not.

A retrofit architecture can potentially extend selected connected-safety capabilities to vehicles that were not designed with them.

## 4. Fleet operators need centralized visibility

A fleet may contain hundreds or thousands of vehicles.

Operators may need to know:

- where vehicles are,
- whether devices are online,
- whether telemetry is arriving,
- whether incidents occurred,
- whether a device has a sensor fault,
- and which vehicles require attention.

## 5. False alarms are dangerous

A hard brake is not necessarily a crash.

A pothole is not necessarily a crash.

A device restart is not a crash.

A robust accident-detection platform therefore needs to distinguish ordinary vehicle dynamics from potentially dangerous events.

---

# Why Existing Approaches Leave Gaps

The market already contains mature telematics, fleet-management, video-safety, GPS-tracking, and connected-vehicle products.

Major platforms include companies such as:

- Geotab
- Samsara
- Motive
- Lytx
- Verizon Connect
- and regional vehicle-tracking providers.

These products demonstrate that connected vehicle data can support fleet operations, safety, compliance, maintenance, and analytics.

Velora is not attempting to claim that these systems do not exist or that Velora is automatically superior.

Instead, Velora is being designed around a particular product direction:

> **A retrofit-oriented safety platform whose architecture starts with an independent safety device and is designed to evolve from individual drivers to fleets and, where partnerships permit, emergency-response workflows.**

The competitive landscape is discussed in more detail below.

---

# The Velora Solution

Velora combines three layers.

## Layer 1 — Physical sensing

The device can collect:

- acceleration,
- rotation,
- position,
- speed,
- heading,
- device state,
- battery information,
- and physical SOS input.

## Layer 2 — Secure connectivity

The device communicates with Velora's backend through:

```text
ESP32/device
    ↓
Cellular/network
    ↓
MQTT broker
    ↓
Velora ingestion service
```

The backend validates the device identity and message before accepting the data.

## Layer 3 — Applications and intelligence

The backend provides:

- REST APIs,
- persistent telemetry,
- incident data,
- notifications,
- real-time WebSockets,
- mobile application services,
- and future fleet/dashboard services.

Future processing can then operate on the telemetry:

```text
Telemetry
    ↓
Signal validation
    ↓
Rule-based crash detection
    ↓
Event verification
    ↓
Emergency workflow
    ↓
Response / recovery
    ↓
Learning and model improvement
```

---

# Who Velora Is For

## Individual drivers

Potential benefits:

- retrofit safety device,
- manual SOS,
- incident history,
- location awareness,
- future automatic crash detection,
- emergency contact workflows.

## Families

Potential use cases:

- monitoring vehicles used by family members,
- incident visibility,
- device health,
- safety alerts.

## Taxi and ride-hailing operators

Potential use cases:

- vehicle tracking,
- driver safety,
- SOS,
- incident management,
- fleet monitoring.

## Bus operators

Potential use cases:

- real-time vehicle location,
- emergency buttons,
- device health,
- incident monitoring,
- operational dashboards.

India already has vehicle-location/emergency-button requirements for certain public-service vehicle categories under the country's regulatory framework, making compliance-aware telematics an important market consideration.

## Truck and logistics fleets

Potential use cases:

- location,
- device health,
- harsh-event telemetry,
- accident detection,
- driver/vehicle safety,
- future insurance and operational analytics.

## Commercial fleets

Potential use cases:

- mixed vehicle tracking,
- safety events,
- centralized incident management,
- maintenance signals,
- utilization analytics.

---

# Core Product Workflow

Velora's long-term safety workflow is:

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
- physical SOS button,
- mobile SOS,
- future vehicle/device events.

## VERIFY

The system should determine whether an event is:

- likely accidental,
- uncertain,
- benign,
- or potentially serious.

Verification may use:

- sensor data,
- speed,
- acceleration,
- orientation,
- GPS,
- event duration,
- post-event motion,
- user confirmation,
- and eventually ML.

## ALERT

Depending on the configured workflow:

- mobile notification,
- in-app incident,
- fleet operator alert,
- emergency-contact notification,
- or a future authorized external integration.

## RESPOND

Response capabilities depend on actual integrations.

Velora must not claim that police, ambulance, hospitals, or emergency dispatch are contacted unless a real integration or operational partnership exists.

## RECOVER

Potential future functions:

- incident history,
- towing/roadside workflows,
- insurance workflows,
- repair workflows,
- fleet investigation.

## LEARN

Data can eventually support:

- sensor validation,
- rule tuning,
- labeled accident datasets,
- false-positive analysis,
- ML model training,
- fleet safety analytics.

---

# Product Components

Velora is composed of:

```text
┌───────────────────────────────────────────┐
│              Velora Platform              │
├───────────────────────────────────────────┤
│ Mobile App        │ Future Web/Fleet App  │
├───────────────────────────────────────────┤
│ REST API          │ WebSockets            │
├───────────────────────────────────────────┤
│ MQTT Ingestion    │ Notification System   │
├───────────────────────────────────────────┤
│ PostgreSQL        │ Event / Telemetry     │
├───────────────────────────────────────────┤
│ Mosquitto / MQTT  │ Future Cloud Infra    │
├───────────────────────────────────────────┤
│ ESP32 Device      │ Sensors + Cellular    │
└───────────────────────────────────────────┘
```

---

# System Architecture

## Current logical architecture

```text
                  ┌──────────────────┐
                  │ Flutter Mobile   │
                  │ Application      │
                  └────────┬─────────┘
                           │ REST / WS
                           ▼
┌────────────────────────────────────────────────┐
│                Velora Backend                  │
│                                                │
│ Auth │ Vehicles │ Devices │ Incidents         │
│ Notifications │ MQTT │ WebSockets │ Location  │
└───────────────┬─────────────────────┬──────────┘
                │                     │
                ▼                     ▼
        ┌──────────────┐       ┌──────────────┐
        │ PostgreSQL   │       │ Mosquitto    │
        │              │       │ MQTT Broker  │
        └──────────────┘       └───────┬──────┘
                                        │
                                        ▼
                                ┌──────────────┐
                                │ Device /     │
                                │ Simulator    │
                                └──────────────┘
```

The simulator represents the future physical device during development.

---

# End-to-End Data Flow

A normal telemetry path is:

```text
Sensor
  ↓
ESP32 firmware
  ↓
Device message
  ↓
Device authentication/signature
  ↓
Cellular/network transport
  ↓
MQTT broker
  ↓
MQTT ingestion
  ↓
Validation
  ↓
Authentication
  ↓
Idempotency
  ↓
Sequence / boot handling
  ↓
PostgreSQL
  ↓
Internal backend event
  ↓
WebSocket
  ↓
Authorized application
```

The important principle is:

> **PostgreSQL is authoritative. WebSockets are a real-time delivery mechanism. MQTT is a device transport mechanism.**

---

# Mobile Application

The mobile application is built with Flutter.

## Current capabilities

The application currently supports:

- account registration,
- login,
- rotating refresh sessions,
- vehicle management,
- device management,
- emergency contacts,
- medical profile,
- incident history,
- maps/location,
- manual SOS,
- notification history,
- notification preferences,
- WebSocket real-time updates,
- settings and logout.

The application uses:

- Flutter,
- Riverpod,
- go_router,
- Dio,
- secure token storage,
- flutter_map,
- and the Velora backend APIs.

## Mobile safety principles

The application must remain useful if:

- WebSockets fail,
- the network is temporarily unavailable,
- push notifications are unavailable,
- the backend temporarily cannot be reached.

REST remains authoritative.

The application must never display fabricated live safety information.

---

# Web / Fleet Application

The web/fleet dashboard is a planned product layer.

It is intended to support:

## Fleet overview

- fleet size,
- online/offline devices,
- active incidents,
- recent incidents,
- vehicle locations,
- device health.

## Vehicle view

- vehicle information,
- linked device,
- recent telemetry,
- location,
- incident history,
- device health.

## Device view

- device identity,
- firmware version,
- last seen,
- connectivity,
- battery,
- sensor status,
- recent messages,
- diagnostic events.

## Incident operations

- incident location,
- status,
- source,
- vehicle,
- device,
- timestamps,
- event history,
- verification workflow.

## Future fleet analytics

- harsh braking,
- acceleration patterns,
- safety trends,
- incident frequency,
- vehicle utilization,
- device reliability,
- maintenance indicators.

The web dashboard must use the same authorization model as the mobile application.

---

# Physical Velora Device

The intended physical Velora device is an embedded safety computer installed inside a vehicle.

## Conceptual hardware

```text
                   ┌─────────────────────┐
                   │      ESP32 MCU       │
                   │   Device Controller  │
                   └─────────┬───────────┘
                             │
       ┌─────────────────────┼──────────────────────┐
       │                     │                      │
       ▼                     ▼                      ▼
   ┌────────┐           ┌────────┐            ┌──────────┐
   │  IMU   │           │ GNSS   │            │ Cellular │
   │ Accel  │           │ GPS    │            │  Modem   │
   │ Gyro   │           │        │            │          │
   └────────┘           └────────┘            └──────────┘
       │                     │                      │
       └──────────────┬──────┴──────────────────────┘
                      │
              ┌───────┴────────┐
              ▼                ▼
         ┌─────────┐      ┌──────────┐
         │ SOS     │      │ Buzzer / │
         │ Button  │      │ LEDs     │
         └─────────┘      └──────────┘
                      │
                 Power System
                      │
                Vehicle / Battery
```

The exact component selection and electrical design belong to the hardware phase and must be validated with physical prototypes.

---

# Device Hardware Architecture

## ESP32

Responsibilities:

- sensor communication,
- device state,
- local processing,
- connectivity control,
- message creation,
- cryptographic signing,
- SOS input,
- diagnostics.

## IMU

Provides:

- accelerometer measurements,
- gyroscope measurements,
- orientation/motion information.

The IMU is central to future crash-detection logic.

## GNSS

Provides:

- latitude,
- longitude,
- speed,
- heading where available,
- timing information.

GPS/GNSS quality must be represented honestly.

The device must not invent location when a fix is unavailable.

## Cellular modem

The production device is intended to communicate independently of the driver's phone.

A cellular design allows:

```text
Device → Cellular → Internet → MQTT → Velora
```

instead of:

```text
Device → Driver Phone → Internet
```

This distinction is important for accident scenarios where the phone may be unavailable.

The exact modem, carrier strategy, SIM/eSIM architecture, roaming strategy, and regional bands must be selected for each target market.

## SOS button

A physical emergency button provides a manual device-side trigger.

The button must be debounced and protected against accidental activation according to the final product requirements.

## Buzzer / LEDs

Possible uses:

- startup state,
- network state,
- SOS confirmation,
- fault indication,
- diagnostics.

Avoid creating confusing alert patterns that could be interpreted as emergency dispatch confirmation.

---

# Physical Device Connection and Installation

The physical device should eventually be connected using:

```text
Vehicle power
    ↓
Protection / regulation
    ↓
Velora power system
    ↓
ESP32 + sensors + modem
```

The exact wiring must be finalized only after:

- component selection,
- current measurements,
- automotive voltage/transient analysis,
- thermal analysis,
- enclosure design,
- EMC/EMI evaluation,
- and safety testing.

The production device should not simply connect an ESP32 development board directly to an automotive power rail.

Potential production interfaces include:

- vehicle power,
- optional ignition signal,
- optional OBD/CAN interface,
- cellular antenna,
- GNSS antenna,
- external SOS button,
- service/programming interface.

OBD/CAN should be treated as a later extension rather than assumed to be available in every vehicle.

---

# Device-to-Cloud Communication

The intended production path is:

```text
ESP32
  │
  ├── IMU
  ├── GNSS
  ├── SOS
  ├── Device state
  │
  ▼
Cellular modem
  │
  ▼
Internet
  │
  ▼
MQTT
  │
  ▼
Velora ingestion
```

## MQTT

MQTT is used because it is well suited to:

- constrained devices,
- intermittent networks,
- low-overhead messaging,
- device telemetry,
- publish/subscribe communication.

The current Velora topic namespace is:

```text
velora/v1/devices/{deviceId}/status
velora/v1/devices/{deviceId}/telemetry
velora/v1/devices/{deviceId}/location
velora/v1/devices/{deviceId}/events
velora/v1/devices/{deviceId}/commands
```

The exact protocol contract is shared through:

```text
packages/shared/
```

so the simulator and backend do not maintain separate copies.

---

# Backend Architecture

The backend is built with:

- NestJS
- TypeScript
- Prisma
- PostgreSQL
- MQTT/Mosquitto
- WebSockets

## REST API

The REST layer handles authoritative operations such as:

- authentication,
- vehicle CRUD,
- device CRUD,
- emergency contacts,
- medical profile,
- incident history,
- notification history,
- notification preferences,
- session management.

## PostgreSQL

PostgreSQL is the authoritative persistent store.

It stores domain information such as:

- users,
- vehicles,
- devices,
- sessions,
- emergency contacts,
- medical profile,
- incidents,
- notifications,
- device credentials/public keys,
- telemetry,
- location,
- device events,
- sequence/boot state.

The exact schema evolves with the roadmap.

---

# MQTT Ingestion

The MQTT backend treats device messages as hostile input.

The ingestion pipeline validates approximately in this order:

```text
Payload size
    ↓
Topic structure
    ↓
JSON/protocol decoding
    ↓
Device identity
    ↓
Cryptographic signature
    ↓
Device allowance/status
    ↓
Schema validation
    ↓
Channel/type validation
    ↓
Timestamp validation
    ↓
Idempotency / ordering
    ↓
Persistence
```

## Device authentication

Device identity is not trusted merely because a `deviceId` appears in the topic.

The current backend uses per-device Ed25519 signing.

The server stores the public key.

The device signs its messages.

This avoids storing a shared secret that could allow a database compromise to forge messages for an entire fleet.

## Idempotency

MQTT delivery can result in duplicate messages.

Velora uses persistent database-level uniqueness/idempotency mechanisms rather than depending only on an in-memory cache.

## Sequence numbers

Sequence numbers allow the backend to identify:

- normal progression,
- duplicate messages,
- missing ranges,
- stale/out-of-order messages.

Missing data is never fabricated.

## Boot IDs

A device restart creates a new boot session.

This allows:

```text
Boot A:
1 → 2 → 3

restart

Boot B:
1 → 2 → 3
```

without incorrectly treating the second sequence as duplicate data.

## Device timestamps

The backend distinguishes:

- device event time,
- server ingestion time.

Server ingestion time remains authoritative for server-side `lastSeenAt`.

---

# Real-Time WebSockets

WebSockets provide real-time updates after authoritative backend operations succeed.

The flow is:

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

WebSocket clients cannot use the connection to bypass REST authorization.

## Authentication

WebSocket connections authenticate using the existing Velora access-token model.

Expired authentication closes the connection instead of silently keeping an expired session alive.

## Authorization

Subscription authorization is checked server-side.

A user cannot subscribe to another user's vehicle or device.

## Delivery semantics

Velora does not claim exactly-once WebSocket delivery.

The intended model is:

> **Real-time event + authoritative REST state**

If an event is missed:

1. the client reconnects,
2. the client retrieves authoritative state from REST,
3. the UI becomes current again.

This is safer than treating a transient WebSocket event as the database truth.

---

# Data Model

The conceptual domain model is:

```text
User
 ├── Vehicles
 │     └── Devices
 │           ├── Telemetry
 │           ├── Locations
 │           └── Device Events
 │
 ├── Emergency Contacts
 ├── Medical Profile
 ├── Incidents
 ├── Notifications
 └── Sessions
```

An incident may reference:

- user,
- vehicle,
- device,
- location,
- source,
- status,
- timestamps.

Incident history preserves useful snapshots so that historical records remain understandable even if related vehicle/device records later change.

---

# Security

Velora treats safety-related data as sensitive.

## Authentication

The platform uses:

- password hashing,
- JWT access tokens,
- rotating refresh sessions,
- secure mobile token storage.

Access tokens are short-lived.

Refresh tokens are rotated and stored server-side only as protected representations.

## Device security

Device messages are cryptographically authenticated.

Device private keys must never be logged or persisted by the backend.

## Medical data

Medical profile data is treated as sensitive.

The current backend encrypts medical profile fields using AES-256-GCM with a separate encryption key.

## Push tokens

Push tokens are protected separately from normal application data and are not returned or logged unnecessarily.

## Authorization

User-owned resources are checked server-side.

The platform must prevent:

- IDOR,
- cross-user access,
- forged device identity,
- unauthorized telemetry subscriptions,
- unauthorized incident access.

## Production security requirements

Before production deployment, Velora still requires:

- TLS everywhere,
- hardened MQTT ACLs,
- secure secret management,
- key rotation,
- certificate management,
- centralized rate limiting,
- penetration testing,
- dependency/security monitoring,
- backup and disaster recovery,
- security incident response,
- auditability appropriate to the deployment.

---

# Privacy and Data Governance

Velora can process highly sensitive data:

- location,
- movement history,
- vehicle identity,
- incident history,
- medical information,
- emergency contacts.

Therefore privacy must be designed into the system rather than added later.

Principles:

1. Collect only data required for the feature.
2. Minimize unnecessary location retention.
3. Encrypt sensitive data at rest.
4. Encrypt network traffic in production.
5. Restrict data by ownership and role.
6. Avoid logging precise location unnecessarily.
7. Never log authentication secrets.
8. Define retention periods.
9. Support deletion/export requirements where legally required.
10. Separate operational telemetry from personally identifying information where practical.

The final privacy/legal design must be adapted to each deployment jurisdiction.

---

# Crash Detection Strategy

Crash detection is deliberately separated from basic telemetry ingestion.

## Why?

A sensor spike alone does not prove an accident.

A future detector can combine:

- longitudinal acceleration,
- lateral acceleration,
- vertical acceleration,
- gyroscope data,
- speed,
- heading,
- vehicle motion before the event,
- vehicle motion after the event,
- orientation changes,
- GPS,
- duration,
- and other signals.

## Development sequence

### Stage 1 — Sensor validation

Verify that:

- sensors are calibrated,
- timestamps are correct,
- units are correct,
- sampling rates are stable,
- data is not silently dropped.

### Stage 2 — Rule-based detection

Start with interpretable rules.

Possible signals:

- sudden deceleration,
- abnormal acceleration,
- large rotational change,
- orientation change,
- speed drop,
- multiple correlated sensor changes.

Rules should produce a **potential event**, not automatically declare a confirmed accident.

### Stage 3 — Verification

Use:

- user confirmation,
- post-event motion,
- additional sensor evidence,
- time windows,
- device health,
- location context.

### Stage 4 — Dataset

Collect controlled and real-world labeled data.

### Stage 5 — ML

Use ML only after sufficient high-quality data exists.

The ML system must be evaluated for:

- false positives,
- false negatives,
- different vehicle types,
- different road conditions,
- sensor placement,
- weather/environmental effects,
- device variation,
- regional driving conditions.

---

# Emergency-Response Strategy

The long-term workflow is:

```text
Potential Event
      ↓
Verification
      ↓
Confirmed Event
      ↓
Configured Alerting
      ↓
Response
```

But Velora must never claim an external response capability until the corresponding integration actually exists.

## Current reality

Velora currently records and processes incidents.

The system does **not** universally dispatch:

- police,
- ambulances,
- hospitals,
- emergency services,
- tow providers,
- or emergency contacts.

Those require actual integrations, contracts, APIs, operational processes, or other verified mechanisms.

## Future integrations

Potential integrations include:

- emergency contacts,
- fleet control centers,
- roadside assistance,
- insurers,
- hospitals,
- emergency-response systems,
- government systems,
- towing providers.

Each integration should be treated as a separate production project.

---

# Notifications

The notification architecture supports:

- in-app notification history,
- unread counts,
- read/unread state,
- notification preferences,
- device token registration architecture,
- multiple devices,
- push-provider abstraction.

The current architecture intentionally does not pretend that FCM/APNs delivery is live without the required production credentials and infrastructure.

Future push delivery requires:

- real Firebase/APNs configuration,
- real-device testing,
- background notification handling,
- cold-start navigation,
- delivery monitoring,
- retry strategy,
- provider failure handling.

Notification delivery must never roll back the underlying incident transaction.

---

# Reliability and Failure Handling

Velora is being designed for failure.

Potential failures include:

- GPS unavailable,
- cellular unavailable,
- MQTT broker unavailable,
- duplicate messages,
- delayed messages,
- out-of-order messages,
- device restart,
- sensor failure,
- low battery,
- backend restart,
- WebSocket disconnect,
- mobile offline state.

## Core principle

Failure should degrade the system gracefully rather than fabricate success.

Examples:

### GPS unavailable

Do not invent a location.

### MQTT unavailable

Device firmware should eventually support local buffering and retry according to the production firmware design.

### WebSocket unavailable

Use REST to recover authoritative state.

### Push unavailable

Persist the notification/incident independently.

### Duplicate MQTT message

Idempotently ignore the duplicate.

### Missing telemetry

Record the gap; never invent measurements.

---

# Deployment Architecture

## Development

Current development environment:

```text
Developer machine
│
├── Docker
│   ├── PostgreSQL
│   └── Mosquitto
│
├── NestJS API
│
├── Flutter Android app
│
└── Device simulator
```

The current local broker is intended for development/testing.

It is not a production MQTT deployment.

---

# Production Architecture

A future production deployment can evolve toward:

```text
                     Internet
                        │
              ┌─────────┴─────────┐
              │ Load Balancer /   │
              │ API Gateway       │
              └─────────┬─────────┘
                        │
        ┌───────────────┼────────────────┐
        ▼               ▼                ▼
   API Instance    API Instance     API Instance
        │               │                │
        └───────────────┼────────────────┘
                        │
                Shared Event Bus
                        │
       ┌────────────────┼─────────────────┐
       ▼                ▼                 ▼
 PostgreSQL        MQTT Cluster       Workers
       │                │                 │
       │                ▼                 │
       │          Device Ingestion        │
       │                                  │
       └──────────────┬───────────────────┘
                      ▼
              Observability
              Logs / Metrics
              Traces / Alerts
```

The exact infrastructure can be:

- cloud-managed PostgreSQL,
- managed MQTT,
- Kubernetes/container services,
- VM-based services,
- serverless components where appropriate,
- managed Redis/pub/sub,
- object storage for datasets,
- CDN for web assets.

The architecture should be selected according to fleet size, latency, cost, and regulatory requirements.

---

# Scaling Strategy

Velora should scale in stages.

## Stage 1 — Development

Tens of simulated devices.

Focus:

- correctness,
- protocol design,
- automated testing.

## Stage 2 — Pilot

Hundreds of real devices.

Focus:

- connectivity,
- hardware reliability,
- sensor quality,
- crash-detection validation,
- support processes.

## Stage 3 — Regional fleet

Thousands to tens of thousands of devices.

Requirements:

- managed MQTT,
- multiple API instances,
- shared WebSocket pub/sub,
- database indexing,
- telemetry retention policy,
- metrics,
- alerting,
- automated deployment,
- device provisioning.

## Stage 4 — Large fleet

Hundreds of thousands or more devices.

Requirements may include:

- horizontally scalable ingestion,
- partitioned telemetry storage,
- stream processing,
- regional infrastructure,
- multi-region architecture,
- fleet-level rate limiting,
- data lifecycle management,
- distributed observability,
- regional data residency.

## Scaling rule

Do not introduce distributed infrastructure merely because it sounds scalable.

Add complexity when measured workload and reliability requirements justify it.

---

# Cost Model

Actual cost depends heavily on:

- component selection,
- cellular country/carrier,
- device volume,
- enclosure,
- manufacturing quantity,
- cloud provider,
- telemetry frequency,
- storage retention,
- support model,
- compliance requirements,
- certifications.

Therefore exact production unit economics should be calculated from supplier quotes and measured cloud usage.

## Hardware cost categories

A device BOM can include:

| Component | Cost driver |
|---|---|
| ESP32/MCU | MCU choice and volume |
| IMU | sensor grade and accuracy |
| GNSS | receiver and antenna |
| Cellular modem | LTE/NB-IoT/LTE-M/other regional support |
| SIM/eSIM | carrier and data plan |
| Antennas | GNSS/cellular design |
| SOS button | industrial/automotive rating |
| Buzzer/LEDs | enclosure and interface |
| Power circuitry | automotive protection/regulation |
| Backup battery | capacity and certification |
| PCB | board size and volume |
| Enclosure | material/tooling/IP rating |
| Connectors/cables | installation requirements |
| Assembly | manufacturing process |
| Testing | QA and calibration |
| Certification | target-market requirements |

## Cloud cost categories

- API compute
- MQTT infrastructure
- PostgreSQL
- telemetry storage
- backups
- object storage
- bandwidth
- WebSocket connections
- monitoring/logging
- push notification infrastructure
- maps/tiles
- DNS/CDN
- CI/CD

## Business unit economics

The eventual model should track:

```text
Revenue per device/month
    -
Cellular cost
    -
Cloud cost
    -
Support
    -
Hardware amortization
    -
Payment fees
    -
Warranty/replacement
    -
Sales/customer acquisition
    =
Contribution margin
```

Do not set final prices from guesses.

Validate them against:

- customer willingness to pay,
- competitor pricing,
- fleet economics,
- hardware BOM,
- cellular cost,
- support cost,
- expected incident volume.

---

# Business Model

Velora can support multiple revenue streams.

## 1. Hardware sale

Customer buys the device.

Potential advantages:

- simple purchase model.

Potential limitation:

- lower recurring revenue.

## 2. Hardware + subscription

Customer purchases or finances hardware and pays a recurring service fee.

Subscription can cover:

- connectivity,
- cloud,
- telemetry,
- incident management,
- notifications,
- fleet dashboard,
- support.

## 3. Fleet SaaS

Charge fleet operators:

- per vehicle,
- per device,
- per active month,
- or according to fleet tier.

Potential tiers:

```text
Starter
Professional
Enterprise
```

## 4. Enterprise integrations

Revenue can come from:

- custom integrations,
- fleet APIs,
- data services,
- operational dashboards,
- support contracts.

## 5. Insurance partnerships

Potential future use:

- safety scoring,
- verified incident records,
- risk analytics.

This requires careful actuarial validation and regulatory/legal review.

## 6. OEM / distributor partnerships

Potential channels:

- vehicle service centers,
- dealers,
- fleet operators,
- insurance companies,
- logistics companies,
- transportation authorities.

---

# Market and Target Segments

## Consumer

- private car owners,
- families,
- older vehicles,
- high-risk driving regions.

## Commercial

- taxis,
- logistics,
- delivery fleets,
- buses,
- trucks,
- school transport,
- corporate fleets.

## Public-sector

Potential future applications:

- public transport,
- municipal fleets,
- emergency-support vehicles,
- regulated commercial vehicles.

Public-sector deployment requires procurement, certification, security, and regulatory processes.

---

# Competitive Landscape

Velora operates in an existing market.

## Geotab

Geotab provides connected vehicle and asset solutions, telematics, fleet data, integrations, and safety/operational capabilities. Its public materials state that its platform connects millions of vehicles/assets and processes very large data volumes.

Website: https://www.geotab.com/

## Samsara

Samsara provides connected operations and fleet-management products spanning telematics, safety, video, compliance, maintenance, and other workflows.

Website: https://www.samsara.com/

## Motive

Motive is a major fleet-management and safety platform serving commercial transportation use cases.

Website: https://gomotive.com/

## Lytx

Lytx focuses strongly on video telematics and driver-safety analytics.

Website: https://www.lytx.com/

## Verizon Connect

Verizon Connect provides fleet-management and telematics capabilities.

Website: https://www.verizonconnect.com/

## Regional AIS-140/VLTD ecosystem

India also has an established ecosystem of vehicle-location tracking and emergency-button providers serving regulated/public-service transportation.

The Government of India's AIS-140 framework defines requirements for vehicle-location tracking and emergency-button systems in applicable public transport contexts.

## What this means for Velora

Velora is entering a competitive category.

The product should therefore compete through a clearly defined combination of:

- retrofit accessibility,
- safety-first architecture,
- device independence,
- transparent data ownership,
- strong security,
- lower deployment complexity for appropriate segments,
- regional customization,
- reliable device operation,
- and partnerships.

These are **product hypotheses to validate**, not claims that Velora is already superior to established providers.

---

# Velora's Positioning

A possible long-term positioning statement is:

> **Velora is a connected safety platform that brings independent sensing, incident intelligence, and response workflows to vehicles through a retrofit device and software ecosystem.**

Potential differentiation areas:

### Vehicle independence

Designed to work without requiring a specific vehicle manufacturer ecosystem.

### Retrofit model

Can target vehicles that lack modern connected-safety features.

### Safety-first architecture

The system treats:

- device identity,
- telemetry integrity,
- incident history,
- failure handling,
- and authorization

as core engineering concerns.

### Hardware + software ownership

The platform controls:

- device,
- firmware,
- backend,
- mobile app,
- future web application,
- detection pipeline.

### Regional adaptation

The product can be adapted for:

- local cellular networks,
- local regulations,
- local languages,
- local emergency workflows,
- local fleet needs.

---

# India and Regulatory Considerations

India is an important potential market for Velora.

India has established regulatory frameworks around vehicle location tracking and emergency buttons.

The AIS-140 standard and related vehicle-location-tracking requirements are particularly relevant to public-service and certain commercial vehicle deployments.

The government VLT/EAS ecosystem describes:

- vehicle location tracking,
- emergency buttons,
- monitoring centers,
- emergency alert systems,
- and integration possibilities with emergency-response systems.

This means a production Velora product targeting regulated public-service vehicles must be designed around the applicable certification and approval process rather than treating a generic GPS tracker as automatically compliant.

## Important

Compliance is not the same as having GPS functionality.

Depending on product/market:

- hardware certification,
- telecom approvals,
- AIS/BIS requirements,
- electrical/EMC testing,
- vehicle installation requirements,
- data protection,
- cybersecurity,
- emergency integration,
- and government enlistment

may apply.

Legal and certification professionals must validate the final requirements before commercial deployment.

---

# Global Expansion

Velora can eventually expand geographically by treating the platform as a common core plus regional modules.

## Common core

- device protocol,
- cloud architecture,
- account model,
- telemetry model,
- incident model,
- crash-detection framework,
- mobile application,
- fleet platform.

## Regional layer

- cellular carriers,
- SIM/eSIM,
- maps,
- languages,
- data residency,
- emergency numbers,
- emergency integrations,
- vehicle regulations,
- certifications,
- taxation,
- payment methods,
- privacy requirements.

## Expansion sequence

A sensible expansion strategy is:

```text
India pilot
   ↓
Regional fleet deployment
   ↓
Other South Asian markets
   ↓
Middle East / Africa opportunities
   ↓
Southeast Asia
   ↓
Selected European / North American markets
```

This is a strategic sequence to validate, not a prediction of market success.

---

# Go-to-Market Strategy

## Phase 1 — Controlled pilots

Start with:

- a small fleet,
- known vehicle types,
- controlled device installations,
- measurable outcomes.

Measure:

- device uptime,
- GPS quality,
- cellular reliability,
- false-positive rate,
- false-negative rate,
- battery performance,
- installation reliability,
- support tickets.

## Phase 2 — Fleet partnerships

Target organizations where a safety platform has measurable value:

- taxi fleets,
- logistics fleets,
- buses,
- corporate vehicles,
- commercial transport.

## Phase 3 — Channel partnerships

Potential channels:

- vehicle workshops,
- fleet-management companies,
- insurers,
- dealers,
- telematics installers,
- transport operators.

## Phase 4 — Platform expansion

Add:

- APIs,
- integrations,
- fleet analytics,
- insurance workflows,
- maintenance signals,
- third-party ecosystem.

---

# Manufacturing and Hardware Strategy

The hardware roadmap should evolve through stages.

## Prototype

Use development boards and off-the-shelf modules.

Goal:

- validate sensor selection,
- firmware,
- communications,
- crash-data collection.

## Engineering prototype

Move toward:

- custom PCB,
- production-oriented connectors,
- stable power design,
- enclosure,
- antennas,
- thermal analysis.

## Pilot hardware

Focus on:

- repeatability,
- installation,
- environmental testing,
- vibration,
- temperature,
- cellular performance,
- GPS performance,
- power behavior.

## Production hardware

Require:

- supplier qualification,
- manufacturing test fixtures,
- calibration procedures,
- serialization,
- secure provisioning,
- firmware update process,
- quality control,
- traceability,
- certification.

---

# Implementation Roadmap

The agreed development roadmap is:

| Step | Phase | Status |
|---:|---|---|
| 1 | Development Environment | Complete |
| 2 | Project Foundation + Git | Complete |
| 3 | Docker Infrastructure | Complete |
| 4 | Backend Foundation | Complete |
| 5 | Authentication | Complete |
| 6 | Core Domain APIs | Complete |
| 7 | Flutter Mobile + Backend Integration | Complete |
| 7.5 | Refresh Token / Session Management | Complete |
| 8 | Maps & Location | Complete |
| 9 | Manual SOS | Complete |
| 10 | Production Notification Foundation | Complete |
| 11 | Device Simulator | Complete |
| 12 | MQTT Backend Ingestion | Complete |
| 13 | Real-time WebSockets | Complete |
| **14** | **ESP32 Hardware** | **Next** |
| 15 | ESP32 Firmware | Planned |
| 16 | Rule-based Crash Detection | Planned |
| 17 | Verification / Emergency Workflow | Planned |
| 18 | Web / Fleet Dashboard | Planned |
| 19 | Sensor Dataset | Planned |
| 20 | Machine Learning | Planned |
| 21 | Security / Reliability Hardening | Planned |
| 22 | Power / Backup Battery | Planned |
| 23 | Controlled Testing | Planned |
| 24 | Pilot | Planned |
| 25 | Production / External Integrations | Planned |

---

# Current Technical Status

## Completed foundation

The current system has:

- NestJS backend
- PostgreSQL
- Prisma
- Mosquitto
- Flutter mobile application
- authentication
- rotating refresh sessions
- vehicles
- devices
- emergency contacts
- medical profile
- incidents
- maps/location
- manual SOS
- notification foundation
- device simulator
- MQTT ingestion
- cryptographic device authentication
- telemetry persistence
- device events
- sequence/gap handling
- boot/restart handling
- WebSockets
- shared device protocol package.

## Current verification

The project has repeatedly used:

- unit tests,
- E2E tests,
- MQTT integration tests,
- real emulator tests,
- real PostgreSQL,
- real Mosquitto,
- simulator runs,
- reconnect testing,
- duplicate testing,
- load testing,
- build/lint/format checks.

The exact test counts change as development continues; individual phase reports are the authoritative record for historical counts.

---

# Production Readiness Checklist

A passing development test suite is not equivalent to production readiness.

Before real public deployment, Velora should satisfy:

## Hardware

- [ ] Production PCB
- [ ] Automotive-grade power design
- [ ] Thermal validation
- [ ] vibration validation
- [ ] enclosure validation
- [ ] antenna validation
- [ ] environmental testing
- [ ] manufacturing QA
- [ ] certification

## Firmware

- [ ] watchdog/recovery
- [ ] secure provisioning
- [ ] secure firmware updates
- [ ] offline buffering
- [ ] modem recovery
- [ ] GPS recovery
- [ ] sensor fault handling
- [ ] power management
- [ ] signed firmware
- [ ] rollback strategy

## Backend

- [ ] production MQTT TLS
- [ ] broker ACLs
- [ ] scalable ingestion
- [ ] durable backpressure
- [ ] telemetry retention policy
- [ ] partitioning/archival as required
- [ ] centralized rate limiting
- [ ] production secrets management
- [ ] key rotation
- [ ] disaster recovery
- [ ] backups
- [ ] observability
- [ ] alerting
- [ ] incident response

## Mobile/Web

- [ ] production push notifications
- [ ] background/cold-start handling
- [ ] iOS production validation if supported
- [ ] web authentication
- [ ] fleet dashboard
- [ ] accessibility
- [ ] localization
- [ ] crash reporting

## Safety

- [ ] validated crash dataset
- [ ] controlled crash-like testing
- [ ] false-positive measurement
- [ ] false-negative measurement
- [ ] sensor placement validation
- [ ] different vehicle validation
- [ ] different road-condition validation
- [ ] emergency workflow testing
- [ ] human verification procedures

## Legal / Compliance

- [ ] privacy policy
- [ ] terms
- [ ] data-processing agreements where applicable
- [ ] jurisdiction-specific privacy review
- [ ] hardware certification
- [ ] telecom certification
- [ ] vehicle/device regulatory review
- [ ] emergency-service agreements where applicable
- [ ] liability review
- [ ] insurance/legal review

---

# Risks and Open Problems

## 1. Crash detection accuracy

The most important technical risk is distinguishing genuine accidents from ordinary vehicle dynamics.

## 2. GPS limitations

GPS can be degraded by:

- tunnels,
- buildings,
- weather/atmospheric conditions,
- antenna placement,
- urban environments.

## 3. Cellular availability

A device cannot communicate through a network that is unavailable.

A production system therefore needs local buffering and recovery strategies.

## 4. Power failure

A crash may damage vehicle power.

Backup power becomes important for continued communication after an incident.

## 5. False alarms

Too many false alarms reduce trust and create operational cost.

## 6. Missed events

False negatives are safety-critical.

Crash-detection evaluation must therefore measure both.

## 7. Cybersecurity

A connected safety device creates an attack surface.

Security must include:

- hardware,
- firmware,
- cellular,
- MQTT,
- APIs,
- applications,
- cloud infrastructure.

## 8. Privacy

Location history can reveal sensitive information.

Retention and access must be carefully controlled.

## 9. Regulatory complexity

A safety/telematics device may face different requirements in different markets.

## 10. Emergency-service integration

Emergency dispatch is not simply an API call.

It can require:

- government systems,
- contracts,
- certification,
- monitoring centers,
- operational staff,
- liability processes.

---

# Repository Structure

The repository is organized around the major product surfaces:

```text
Velora/
├── apps/
│   ├── api/                 # NestJS backend
│   ├── mobile/              # Flutter mobile application
│   └── web/                 # Future web/fleet application
│
├── firmware/
│   └── esp32/               # ESP32 firmware
│
├── packages/
│   └── shared/              # Shared protocol/domain definitions
│
├── tools/
│   └── device-simulator/    # Production-oriented device simulator
│
├── infrastructure/
│   ├── docker/              # Docker infrastructure
│   └── mqtt/                # Mosquitto configuration
│
├── data/                    # Development/test data
├── scripts/                 # Development utilities
│
├── docs/
│   ├── architecture/
│   ├── api/
│   ├── hardware/
│   ├── security/
│   └── product/
│
├── CLAUDE.md
└── README.md
```

The exact repository structure may evolve.

---

# Developer Setup

## Requirements

Current development stack includes:

- Node.js 24.x
- npm 11.x
- Flutter
- JDK 17
- Android SDK
- Docker / Docker Compose
- PostgreSQL
- Mosquitto
- Git

## Infrastructure

Start development infrastructure with the repository's Docker Compose configuration.

Verify:

```text
PostgreSQL
Mosquitto
```

are healthy before running integration tests.

## Backend

The API runs under:

```text
http://localhost:3000/api
```

The health endpoint is:

```text
GET /api/health
```

## Mobile

The Flutter application can run against the development API.

Android emulator testing is part of the current development workflow.

## Device simulator

The device simulator can publish realistic telemetry to the development MQTT broker and supports deterministic scenarios.

See:

```text
tools/device-simulator/
```

for its current commands and configuration.

---

# Testing Strategy

Velora uses multiple layers of testing.

## Unit tests

Validate:

- business logic,
- validation,
- serialization,
- authentication,
- device protocol,
- sensor generation,
- WebSocket behavior.

## Integration tests

Validate:

- PostgreSQL,
- MQTT,
- API,
- device simulator,
- WebSocket delivery.

## End-to-end tests

Validate complete flows.

Example:

```text
Simulator
  ↓
Mosquitto
  ↓
MQTT ingestion
  ↓
PostgreSQL
  ↓
WebSocket
  ↓
Authenticated client
```

## Hardware tests

Future physical testing will validate:

- sensor readings,
- GPS,
- cellular,
- power,
- thermal behavior,
- vibration,
- button behavior,
- recovery.

## Controlled safety tests

Future crash-detection testing must use controlled and safe test methodologies.

Do not test crash algorithms by intentionally creating dangerous road accidents.

---

# Engineering Principles

## 1. Truth over demos

Never make a UI claim a feature works if the underlying integration does not exist.

## 2. PostgreSQL is authoritative

Real-time transports are not databases.

## 3. MQTT input is hostile

Every device message must be authenticated and validated.

## 4. Fail safely

A missing GPS fix is better than a fabricated location.

## 5. Security is part of the architecture

Do not postpone identity, authorization, encryption, and secrets until the end.

## 6. Production-oriented does not mean prematurely over-engineered

Build the simplest architecture that is correct now, while leaving a clean path to scale.

## 7. Measure before optimizing

Load-test actual workloads before introducing distributed infrastructure.

## 8. Separate detection from response

Detecting a potential crash does not automatically mean contacting emergency services.

## 9. Keep hardware and software contracts explicit

The simulator and backend must share one protocol definition.

## 10. Test failure, not only success

The platform must be tested under:

- disconnects,
- duplicates,
- stale messages,
- GPS loss,
- device restarts,
- broker restarts,
- authentication failures,
- malformed data.

---

# Long-Term Vision

The long-term Velora architecture is:

```text
                         VELORA
                           │
          ┌────────────────┼─────────────────┐
          │                │                 │
       DEVICE           MOBILE            WEB/FLEET
          │                │                 │
          └────────────────┼─────────────────┘
                           │
                      CLOUD PLATFORM
                           │
       ┌───────────────────┼────────────────────┐
       │                   │                    │
    TELEMETRY           INCIDENTS          NOTIFICATIONS
       │                   │                    │
       └───────────────────┼────────────────────┘
                           │
                   SAFETY INTELLIGENCE
                           │
                 ┌─────────┴─────────┐
                 │                   │
            RULE ENGINE             ML
                 │                   │
                 └─────────┬─────────┘
                           │
                       VERIFICATION
                           │
                       RESPONSE
                           │
          ┌────────────────┼─────────────────┐
          │                │                 │
       CONTACTS         FLEETS        EXTERNAL PARTNERS
                                            │
                              Emergency / Roadside / Insurance
```

The ultimate goal is not simply vehicle tracking.

The goal is to build a **connected safety infrastructure layer** that can sense a potentially dangerous event, preserve trustworthy evidence, verify what happened, notify the right people, support an appropriate response, and learn from the event.

That vision depends on successful hardware validation, reliable connectivity, high-quality data, strong cybersecurity, accurate detection, responsible emergency integrations, and regulatory compliance.

---

# References and Further Reading

The following sources are useful for understanding the surrounding market and regulatory environment:

- **Government of India / Ministry of Road Transport & Highways — AIS-140:** vehicle-location tracking and emergency-button requirements.
- **Government of India / VLT & EAS portals:** examples of regulated vehicle-location tracking and emergency-alert deployments.
- **Geotab:** connected vehicle and fleet telematics platform.
- **Samsara:** connected operations and fleet-management platform.
- **Motive:** commercial fleet-management and safety platform.
- **Lytx:** video telematics and driver-safety platform.
- **Verizon Connect:** fleet-management and telematics platform.

Regulatory requirements should always be checked against the current official notification, standard, certification body, and target-market requirements before deployment.

---

# License

Velora is proprietary software unless a separate license file or written agreement states otherwise.

See the repository's license documentation for the applicable terms.

---

## Final Product Principle

**Velora should never confuse a convincing demonstration with a safe production system.**

Every production capability must eventually be supported by:

```text
Correct hardware
      +
Correct data
      +
Secure communication
      +
Validated algorithms
      +
Reliable infrastructure
      +
Real-world testing
      +
Operational procedures
      +
Regulatory/legal review
```

Only when those layers have been independently validated should a Velora deployment be considered ready for real safety-critical use.
