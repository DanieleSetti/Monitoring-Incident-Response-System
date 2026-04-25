# Architecture Documentation

## 1. Overview

### Problem Addressed

This project demonstrates how to design, monitor, and operate a cloud-based application under real-world conditions.

The focus is not only on deploying infrastructure, but on ensuring that the system is:

- observable
- diagnosable
- recoverable under failure

The system is intentionally designed to simulate production scenarios where failures occur and must be detected and resolved.

---

### System Goal

To build a system where:

service runs → failure occurs → issue is detected → root cause is identified → system is restored

---

### Technologies Used

- AWS (VPC, EC2, Auto Scaling, ALB, RDS)
- Prometheus (metrics collection)
- Node Exporter (system metrics)
- Grafana (visualization)
- CloudWatch (baseline monitoring)
- Docker (monitoring stack)

---

## 2. Architecture

### High-Level Flow

Client  
↓  
Application Load Balancer  
↓  
EC2 Instances (Auto Scaling Group)  
↓  
RDS (PostgreSQL)

Monitoring Layer:

- Prometheus (running on EC2)
- Node Exporter (on each instance)
- Grafana (dashboard)
- Alert rules (Prometheus)

---

### Network Design

- Public subnets:
  - ALB
  - Prometheus/Grafana

- Private subnets:
  - Application EC2 instances
  - RDS database

This ensures:
- isolation of compute and database
- no direct public access to application instances

---

### Observability Layer

Monitoring is implemented as a separate layer without modifying core infrastructure.

Components:

- Prometheus scrapes metrics from:
  - local node exporter
  - EC2 instances in private subnets

- Grafana visualizes:
  - CPU usage
  - memory usage
  - network traffic
  - instance availability

- Alert rules evaluate system state

---

## 3. Key Design Decisions & Trade-offs

### 1. Separate Monitoring Layer

**Decision:**  
Monitoring is deployed independently from application infrastructure.

**Reason:**  
Avoid coupling and allow independent debugging.

---

### 2. Prometheus over CloudWatch

**Decision:**  
Prometheus + Grafana used for primary monitoring.

**Trade-off:**
- + More flexible and industry-relevant
- - Requires manual setup and networking

CloudWatch is still used for baseline AWS-level metrics.

---

### 3. Static Target Configuration

**Decision:**  
Prometheus uses static IPs for EC2 instances.

**Trade-off:**
- + Simple and fast to implement
- - Breaks with Auto Scaling (ephemeral instances)

---

### 4. Node Exporter via User Data

**Decision:**  
Node Exporter is installed during EC2 initialization.

**Reason:**
Ensures monitoring works automatically for new instances.

---

### 5. No Service Discovery (yet)

**Decision:**  
Dynamic discovery (EC2/ASG) is not implemented.

**Reason:**
Focus is on understanding monitoring fundamentals rather than infrastructure complexity.

---

## 4. Challenges & Lessons Learned

### 1. Cloud-init Failure

Issue:
Node Exporter was not running.

Root cause:
Docker was not installed before execution, and `set -e` stopped the script.

Lesson:
Initialization scripts must be resilient and validated.

---

### 2. Connection Refused vs Network Issues

Observation:
Prometheus showed "connection refused".

Insight:
This indicates the service is not running, not a network issue.

Lesson:
Differentiate between:
- network failure (timeout)
- service failure (refused)

---

### 3. Ephemeral Nature of ASG

Problem:
Instances are replaced dynamically.

Impact:
Static Prometheus targets become invalid.

Lesson:
Monitoring should target the service, not individual instances.

---

### 4. Stateless vs Stateful Systems

Observation:
Grafana dashboards were lost after container restart.

Lesson:
- containers are ephemeral
- persistent storage is required for state

---

## 5. Deployment Flow

1. Infrastructure is provisioned (VPC, ALB, ASG, RDS)
2. EC2 instances launch via Auto Scaling
3. User Data:
   - installs application
   - starts Node Exporter
4. Prometheus scrapes metrics
5. Grafana visualizes data
6. Alerts evaluate system health

---

## 6. Failure Scenarios

### Scenario 1 — High CPU

- cause: artificial load
- detection: CPU alert
- visibility: Grafana spike
- resolution: stop process

---

### Scenario 2 — Instance Monitoring Failure

- cause: exporter stopped
- detection: `up == 0`
- resolution: restart exporter

---

### Scenario 3 — Application Failure

- cause: backend stopped
- detection: degraded metrics / ALB behavior
- resolution: restart service

---

## 7. Monitoring Strategy

The system monitors:

- CPU utilization
- memory usage
- network throughput
- instance availability

Key principle:

❗ monitor system behavior, not individual components

---

## 8. Limitations

- static Prometheus targets
- no centralized logging
- no alert notifications (Alertmanager not configured)
- limited application-level metrics

---

## 9. Future Improvements

- EC2 service discovery for Prometheus
- Alertmanager (Slack / Telegram)
- centralized logging (ELK / CloudWatch Logs)
- application-level metrics (latency, error rate)
- HTTPS and security hardening

---

## 10. Key Takeaways

This project demonstrates:

- how to observe a distributed system
- how to detect failures using metrics
- how to investigate incidents
- how to restore system functionality

The main shift is from:

“deploying infrastructure”

to:

“understanding and operating a system in production”
