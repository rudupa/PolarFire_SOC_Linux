# MQTT Deep Dive for Autonomous Vehicle Platforms

## Table of Contents

- [1. Purpose](#1-purpose)
- [2. Where MQTT Fits](#2-where-mqtt-fits)
- [3. Core MQTT Components](#3-core-mqtt-components)
- [4. Topic and Payload Design](#4-topic-and-payload-design)
- [5. QoS and Session Strategy](#5-qos-and-session-strategy)
- [6. Security Model](#6-security-model)
- [7. Development Workflow](#7-development-workflow)
- [8. Monitoring and Operations](#8-monitoring-and-operations)
- [9. Common Failure Modes](#9-common-failure-modes)
- [10. Best Practices](#10-best-practices)

## 1. Purpose

This document provides a practical MQTT guide for AV telemetry, fleet operations, and remote command channels on Linux-based embedded platforms.

## 2. Where MQTT Fits

MQTT is ideal for vehicle-to-cloud communication where bandwidth is variable and clients are resource-constrained.

ASCII view:

```text
[Vehicle ECU] --TLS--> [Edge Broker] --TLS--> [Cloud Broker] --> [Fleet Apps]
      |                        |                     |
      +---- telemetry ---------+----- commands ------+
```

Mermaid view:

```mermaid
flowchart LR
    V[Vehicle Gateway] -->|MQTT over TLS| E[Edge Broker]
    E -->|Bridge| C[Cloud Broker]
    C --> F[Fleet Services]
    F --> C
    C --> E
    E --> V
```

## 3. Core MQTT Components

- Client: publisher and subscriber endpoint
- Broker: central message router
- Topic namespace: hierarchical routing key
- QoS levels: 0, 1, 2 delivery semantics
- Retained messages and Last Will
- Session state and clean start behavior

## 4. Topic and Payload Design

Use stable, readable naming with ownership.

Example topic layout:

```text
fleet/<region>/<vehicle_id>/telemetry/<subsystem>
fleet/<region>/<vehicle_id>/events/<severity>
fleet/<region>/<vehicle_id>/commands/<component>
```

Payload guidance:
- include timestamp, vehicle_id, source_component, schema_version
- avoid oversized payloads in high-frequency streams
- use explicit schema version fields

## 5. QoS and Session Strategy

Suggested mapping:
- high-rate telemetry: QoS 0 or QoS 1
- critical acknowledgable command: QoS 1
- rare exactly-once business events: QoS 2 when required

Session recommendations:
- persistent sessions for intermittent links
- bounded offline queue policy to avoid unbounded memory

## 6. Security Model

Security baseline:
- mutual TLS client authentication
- strict topic ACLs by vehicle and service role
- short-lived credentials where possible
- certificate rotation and revocation process

## 7. Development Workflow

1. Define topic taxonomy and ownership
2. Define payload schema and compatibility policy
3. Implement producer and consumer clients
4. Validate reconnection and offline buffering
5. Run chaos tests with packet loss and broker restarts
6. Qualify at fleet-scale message rate

## 8. Monitoring and Operations

Track:
- publish success and failure rates
- reconnect frequency and session resume success
- broker queue depth and retained count
- command round-trip latency

Use dashboards and alerts for:
- reconnect storms
- sustained publish failures
- broker saturation

## 9. Common Failure Modes

- Topic wildcard misuse causing excessive fan-out
- ACL misconfiguration blocking critical traffic
- Retained message misuse causing stale command replay
- Oversized payloads causing latency spikes

## 10. Best Practices

- Keep MQTT for telemetry and remote ops, not tight control loops
- Enforce per-topic data contracts and version checks
- Use idempotent command handlers in the vehicle
- Separate command and telemetry topic trees
- Add end-to-end correlation IDs for incident tracing
