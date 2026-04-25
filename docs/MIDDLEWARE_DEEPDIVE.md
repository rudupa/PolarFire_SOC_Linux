# Middleware Deep Dive for Autonomous Vehicle Linux Platforms

## Table of Contents

- [1. Purpose](#1-purpose)
- [2. Middleware in the AV Stack](#2-middleware-in-the-av-stack)
- [3. Middleware Boundaries](#3-middleware-boundaries)
- [4. Core Middleware Components](#4-core-middleware-components)
- [5. Design Principles and Architecture](#5-design-principles-and-architecture)
- [6. Tools and Frameworks by Lifecycle](#6-tools-and-frameworks-by-lifecycle)
- [7. Development Workflow](#7-development-workflow)
- [8. Monitoring and Observability](#8-monitoring-and-observability)
- [9. Maintenance and Operations](#9-maintenance-and-operations)
- [10. Security and Safety Expectations](#10-security-and-safety-expectations)
- [11. Reference Deployment Patterns](#11-reference-deployment-patterns)
- [12. Practical Checklists](#12-practical-checklists)

## 1. Purpose

This document describes middleware for Autonomous Vehicle (AV) platforms as a product layer with clear ownership, interfaces, runtime guarantees, and operational controls.

Goals:
- Define middleware scope and boundaries
- Identify core components
- List practical tools and frameworks for design, development, monitoring, and maintenance
- Provide implementation patterns suitable for embedded Linux systems

## 2. Middleware in the AV Stack

A simplified AV stack:

1. Platform layer
- Linux kernel, drivers, networking, storage, time sync, process supervision, security primitives

2. Middleware layer
- Messaging, discovery, serialization, data contracts, routing, buffering, health, and policy enforcement

3. Application layer
- Perception, localization, planning, control, diagnostics, HMI, telemetry business logic

Middleware is the contract and transport fabric between platform capabilities and application behavior.

## 3. Middleware Boundaries

### 3.1 Boundary with platform layer

Platform provides:
- OS scheduling, memory, sockets, filesystems
- Device access and kernel drivers
- Boot, service manager, and low-level security

Middleware consumes platform services but should not own:
- Hardware drivers
- Bootloader and kernel config policy
- Global OS user management and package management

Middleware should define required platform capabilities, for example:
- Minimum clock sync quality
- Network MTU and interface requirements
- Required kernel features for shared memory or zero-copy transport

### 3.2 Boundary with application layer

Applications own:
- Business logic and decision algorithms
- Domain-specific state machines
- Vehicle behavior and mission logic

Middleware owns:
- Message schema transport and compatibility policy
- Service discovery and endpoint liveness
- Delivery semantics, retries, and backpressure behavior

Applications should depend on middleware contracts, not transport internals.

### 3.3 Ownership model

- Platform team owns OS and hardware readiness
- Middleware team owns communication and runtime policy
- Application teams own feature logic

This ownership model reduces coupling and speeds fault isolation.

## 4. Core Middleware Components

### 4.1 Communication and transport

- DDS domain participants, topics, and QoS profiles
- Zenoh routers/sessions for edge and cross-domain data distribution
- MQTT clients/brokers for cloud telemetry and command channels

### 4.2 Data contract layer

- IDL or schema definitions
- Versioning rules
- Compatibility gates in CI

### 4.3 Routing and discovery

- Endpoint discovery
- Topic namespace policy
- Bridge and gateway components (DDS to Zenoh, DDS to MQTT)

### 4.4 Reliability controls

- Retry strategy
- Queue policies and drop strategy
- Store-and-forward for intermittent links

### 4.5 Runtime control plane

- Health probes and heartbeats
- Dynamic configuration reload
- Feature flags and kill-switch controls

### 4.6 Security plane

- Authentication and authorization hooks
- Transport encryption
- Credential distribution and rotation flow

## 5. Design Principles and Architecture

### 5.1 Contract first design

- Define schemas and service contracts before implementation
- Separate control-plane and data-plane topics
- Use explicit version fields and compatibility guarantees

### 5.2 Determinism and latency budgeting

For each critical pipeline, define:
- Maximum end-to-end latency
- Maximum jitter
- Allowed drop percentage

Then map QoS and queue settings to these targets.

### 5.3 Failure aware design

- Design for network partitions and process restarts
- Avoid hidden infinite retries
- Prefer bounded queues and explicit overload behavior

### 5.4 Evolvability

- Backward-compatible schema evolution
- Topic deprecation windows
- Multi-version interoperability during rollout

## 6. Tools and Frameworks by Lifecycle

### 6.1 Design

Communication and architecture:
- ROS 2 architecture patterns (when used)
- DDS vendor documentation and QoS calculators
- Zenoh topology design guides
- AsyncAPI and OpenAPI style interface documentation

Modeling and review:
- Mermaid or PlantUML for data-flow diagrams
- ADR (Architecture Decision Record) templates

### 6.2 Development

Core implementation frameworks:
- DDS implementations such as Cyclone DDS or Fast DDS
- Eclipse Zenoh libraries (C, C++, Rust)
- MQTT clients such as Eclipse Paho

Build and packaging:
- Bazel or CMake for reproducible builds
- Buildroot or Yocto integration for target images

Quality tools:
- clang-tidy, cppcheck, sanitizers
- Unit test frameworks (GoogleTest, Catch2, pytest)
- Contract test harnesses for schema compatibility

### 6.3 Monitoring

Metrics and tracing:
- Prometheus exporters for middleware stats
- Grafana dashboards for latency, drop rate, queue depth
- OpenTelemetry for traces and correlation IDs

Logs and events:
- journald plus log forwarder (fluent-bit/vector)
- Structured logs with request or frame correlation IDs

### 6.4 Maintenance

Release and operations:
- CI/CD with staged rollout and rollback
- Artifact repository with binary, config, and symbol retention
- SBOM generation and vulnerability scanning

Runtime maintenance:
- systemd service policies for restart and watchdog
- Configuration management with versioned profiles
- On-device diagnostics and remote support scripts

## 7. Development Workflow

1. Define contracts
- Schema, topic naming, QoS class, ownership

2. Implement adapters and middleware services
- Serialization, transport bindings, bridge components

3. Validate locally
- Unit tests, schema compatibility tests, fault injection

4. Integrate on target
- Real network and time-sync conditions

5. Qualify
- Soak tests, stress tests, and recovery tests

6. Release
- Gradual rollout with telemetry-based acceptance

## 8. Monitoring and Observability

### 8.1 Minimum middleware SLO metrics

- Publish and subscribe throughput
- End-to-end latency percentiles (p50, p95, p99)
- Message drop rate and queue occupancy
- Discovery convergence time
- Reconnect success rate

### 8.2 Alert strategy

- Threshold alerts for sustained latency and drop rates
- Ratio alerts for reconnect storms
- Heartbeat missed alerts for critical nodes

### 8.3 Incident triage data

Always collect:
- Middleware configuration version
- Topology snapshot
- Recent latency and drop histograms
- Relevant system logs and process status

## 9. Maintenance and Operations

### 9.1 Version and compatibility policy

- Semantic versioning for middleware APIs and schemas
- Explicit support matrix across releases
- Defined deprecation timeline for topics and fields

### 9.2 Change management

- Peer review for QoS changes
- Canary rollout before fleet-wide rollout
- Automated rollback criteria

### 9.3 Performance regression control

- Baseline benchmarks in CI
- Capacity tests under worst-case sensor rates
- CPU and memory budget enforcement

## 10. Security and Safety Expectations

### 10.1 Security baseline

- Mutual TLS where applicable
- Per-topic authorization policy
- Secret rotation and certificate renewal automation
- Hardening of brokers and routers

### 10.2 Safety baseline

- Middleware failures must fail-safe, not fail-open
- Critical command channels require stricter policy and monitoring
- Separation between safety-critical and non-critical data paths

## 11. Reference Deployment Patterns

### 11.1 In-vehicle real-time data plane

- DDS for perception, localization, planning, and control data flows
- Shared-memory or zero-copy path for high-rate sensor streams where possible

### 11.2 Edge and cloud integration plane

- Zenoh routers for cross-domain distribution and query
- MQTT for telemetry, fleet state, and remote commands

### 11.3 Bridge pattern

- DDS internal topics
- Bridge service transforms and forwards selected data to Zenoh or MQTT
- Strict filtering to avoid leaking high-rate raw streams unnecessarily

## 12. Practical Checklists

### 12.1 Design checklist

- Contract versioning rules approved
- Topic naming and ownership defined
- QoS profiles mapped to latency and reliability budgets
- Security controls defined per channel

### 12.2 Development checklist

- Unit and integration tests passing
- Fault injection scenarios validated
- Resource budgets measured on target hardware

### 12.3 Operations checklist

- Dashboards and alerts in place
- Rollback plan tested
- Runbook available for common middleware failures
- Compatibility matrix up to date
