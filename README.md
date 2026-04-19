#AWS Infrastructure with Monitoring & Incident Response

## Overview

This project demonstrates a production-like AWS infrastructure for a web application, extended with a monitoring and incident response layer.

The goal is not only to build infrastructure, but to show how a system is **observed, diagnosed, and recovered when failures occur**.

The project evolves from a standard cloud deployment into an **operations-focused system**, reflecting real-world DevOps responsibilities.

---

## Current Architecture

Client → ALB → EC2 (Nginx + Node.js) → RDS

* Multi-AZ setup
* Private subnets for compute and database
* Auto Scaling Group for high availability
* Immutable deployments via Instance Refresh

---

## New Layer: Observability & Operations

The project is being extended with an **operations layer**, focusing on:

* Monitoring system behavior
* Detecting failures
* Responding to incidents
* Understanding system health beyond infrastructure

---

## Monitoring Strategy

### CloudWatch (Implemented)

Monitoring is currently based on AWS-native metrics, focusing on **system-level behavior**, not individual instances.

#### Key Metrics:

* **HTTPCode_ELB_5XX_Count**

  * Detects backend failures
* **UnHealthyHostCount**

  * Detects failing instances behind the load balancer

#### Alerts:

* 5XX errors > threshold
* Unhealthy targets > 0

#### Important Design Choice:

Monitoring is based on **ALB metrics**, not EC2 instance metrics.

Reason:

> Instances are ephemeral in Auto Scaling environments, so monitoring must focus on service behavior rather than individual machines.

---

## Incident Simulation (Completed)

The system was tested using real failure scenarios:

### Scenario 1 — Instance Termination

* EC2 instances were manually terminated
* Auto Scaling replaced instances
* ALB temporarily had unhealthy targets
* Alert was triggered

### Scenario 2 — Application Failure

* Backend became unavailable
* 5XX errors increased
* Alert was triggered

### Result:

* Alerts correctly transitioned: **OK → ALARM → OK**
* System recovered automatically via ASG

---

## Next Step: Prometheus + Grafana

To extend beyond AWS-native monitoring, the next phase introduces:

* **Prometheus** (metrics collection)
* **Grafana** (visualization)

### Goals:

* Build custom monitoring outside AWS
* Understand metric collection and scraping
* Create dashboards for system behavior
* Implement custom alerting logic

### Initial Scope:

* Node Exporter (CPU, memory)
* Basic dashboards (Grafana)
* Single-instance setup (no HA, no overengineering)

---

## Future Work

### 1. Advanced Monitoring

* Prometheus integration with EC2
* Service discovery (ASG-aware)
* Application-level metrics

### 2. Logging

* Centralized log collection
* Error filtering and debugging

### 3. Alerting Improvements

* Reduce false positives
* Introduce thresholds and time windows

### 4. Incident Response

Simulate and document failures:

* Service crash
* High CPU usage
* Application errors

For each scenario:

* Detection
* Investigation
* Resolution

---

## Definition of Done (Target State)

This project will be considered complete when:

* System failures are automatically detected
* Alerts are triggered reliably
* Root cause can be identified using metrics/logs
* Recovery process is clearly defined
* All scenarios are documented

