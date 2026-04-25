# Production Monitoring & Incident Response System on AWS

## Overview

This project extends https://github.com/DanieleSetti/Highly-Available-Web-Application-Infrastructure-on-AWS project by adding a full monitoring and incident response layer.

The system is designed not only to run a web application, but to **detect failures, alert on issues, and guide recovery** — reflecting real-world DevOps responsibilities.

The application is deployed on EC2 instances behind an Application Load Balancer, with a PostgreSQL database on Amazon RDS. Monitoring is implemented using Prometheus and Grafana, with alerting based on system and application behavior.

The goal is to demonstrate how a cloud system is **observed, diagnosed, and recovered**, not just deployed.

---

## Architecture

High-level flow:

Client → ALB → EC2 (Nginx + Node.js) → RDS

+ Monitoring Layer:
- Prometheus (metrics collection)
- Node Exporter (EC2 system metrics)
- Grafana (visualization)
- Alerting rules (incident detection)

### Key Characteristics

- Multi-AZ deployment
- Private subnets for compute and database
- Auto Scaling Group (ephemeral instances)
- Centralized monitoring system
- Alert-based incident detection

---

## Tech Stack

- **AWS**: VPC, EC2, Auto Scaling, ALB, RDS, NAT Gateway
- **Monitoring**: Prometheus, Node Exporter, Grafana
- **Backend**: Node.js (Express)
- **Web Server**: Nginx
- **CI/CD**: GitHub Actions

---

## Monitoring Setup

The system collects and visualizes:

- CPU usage (per instance)
- Memory usage
- Network traffic
- Instance availability (`up` metric)

Prometheus scrapes:
- Local node exporter
- EC2 instances in private subnets

Grafana dashboards provide real-time visibility into system behavior.

---

## Alerting

Two core alerts are implemented:

### High CPU Usage
- Trigger: CPU > 70% for 1 minute
- Type: warning

### Instance Down
- Trigger: `up == 0` for 30 seconds
- Type: critical

Alerts are evaluated in Prometheus and visible in both Prometheus and Grafana UI.

---

## Incident Scenarios

### 🔴 Incident 1 — High CPU Load

- **Simulation**: `stress --cpu 2 --timeout 120`
- **Detection**: Prometheus alert triggered
- **Investigation**: CPU spike visible in Grafana
- **Root Cause**: artificial CPU load
- **Resolution**: stop stress process
- **Verification**: CPU returns to normal, alert resolves

---

### 🔴 Incident 2 — Instance Monitoring Failure

- **Simulation**: `docker stop node_exporter`
- **Detection**: `up == 0` alert triggered
- **Investigation**: missing metrics from instance
- **Root Cause**: exporter stopped
- **Resolution**: restart exporter container
- **Verification**: metrics restored, alert resolves

---

### 🔴 Incident 3 — Application Failure (optional)

- **Simulation**: stop backend service
- **Detection**: ALB / metrics degradation
- **Investigation**: application logs + missing responses
- **Resolution**: restart service
- **Verification**: traffic restored

---

## Key Design Decisions

- Monitoring implemented as a separate layer (no changes to core infra)
- Static Prometheus targets used for simplicity
- Node Exporter deployed via EC2 initialization
- Alerts focus on system-level signals rather than instance identity

---

## Known Limitations

- Prometheus uses static IP targets (not ASG-aware)
- No centralized log aggregation yet
- No Alertmanager integration (notifications)

### Production Improvements

- EC2 service discovery (dynamic targets)
- Alertmanager (Slack / Telegram alerts)
- Centralized logging (ELK / CloudWatch Logs)
- HTTPS and security hardening

---

## What This Project Demonstrates

- Monitoring distributed systems in private networks
- Detecting failures via metrics and alerts
- Understanding system behavior under load
- Handling incidents end-to-end

---

## Documentation

Full infrastructure and architecture details:

👉 `architecture.md`
