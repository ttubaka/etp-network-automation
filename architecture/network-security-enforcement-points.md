# Network Security Enforcement Points Model

## Overview

This document defines all security inspection and enforcement points within the enterprise network architecture.

Security controls are applied at:

- Network Edge
- Internal Segmentation Boundaries
- Access Layer (NAC)
- East-West Traffic Paths
- Monitoring and Logging Layers

The model aligns with Zero-Trust principles and assumes breach.

---

# Threat Model

## Assumed Threat Actors

| Threat Actor                 | Capability Level | Motivation                              |
| ---------------------------- | ---------------- | --------------------------------------- |
| External cybercriminal       | Medium–High      | Financial gain (ransomware, data theft) |
| Insider (malicious employee) | Medium           | Data exfiltration, sabotage             |
| Compromised endpoint         | Automated        | Lateral propagation                     |
| Supply chain compromise      | High             | Persistence, espionage                  |


## Assumed Attack Paths

1. Phishing email compromises user credentials
2. Malware executes on user workstation
3. Lateral movement attempts via:
   - SMB
   - RDP
   - Unrestricted internal VLAN access
4. Privilege escalation
5. Data exfiltration or ransomware deployment

The architecture assumes attackers may gain initial foothold inside the User VLAN.

---

# Edge Firewall

## Location
Between Internet and enterprise network.

## Functions

- Block unauthorized inbound traffic
- Stateful inspection
- NAT
- Geo/IP filtering
- IPS capabilities

## Inspection Type

- North-South traffic inspection
- Deep Packet Inspection (DPI)
- TLS inspection (where appropriate)

---

# Internal Segmentation Firewall (ISFW)

## Location

Between internal VLANs and critical resources.

## Purpose

- Enforce Zero-Trust internally
- Restrict east-west lateral movement
- Protect sensitive systems (databases, medical systems, management systems)

---

# East-West Traffic Inspection (Expanded)

East-West traffic refers to internal traffic moving between VLANs.

Without inspection, attackers can pivot freely.

The ISFW performs:

- Layer 3 and Layer 7 inspection between:
  - User VLAN → Server VLAN
  - User VLAN → Medical VLAN
  - Server VLAN → Database tier
- Application-aware filtering
- Protocol validation
- Threat signature inspection

Example:

User VLAN cannot directly access database servers.
Only application servers may communicate with databases.

This enforces workload isolation and service chaining.

---

# Ransomware Blast Radius Containment

## Scenario

A user workstation in VLAN10 becomes infected with ransomware.

## Containment Mechanisms

- User VLAN isolated from:
  - Server VLAN (except required ports)
  - Medical Devices VLAN
  - Management VLAN
- SMB traffic restricted
- Admin shares blocked
- Lateral RDP denied

Because segmentation exists:

- Malware cannot scan entire enterprise
- Cannot directly reach database servers
- Cannot encrypt medical equipment
- Cannot access backup systems

Blast radius is limited to the User VLAN segment.

Segmentation reduces enterprise-wide impact.

---

# Network Access Control (NAC)

## Location

Access layer (switch level)

## Functions

- 802.1X authentication
- Device posture validation
- Dynamic VLAN assignment
- Quarantine non-compliant devices

Prevents:

- Rogue devices
- Unauthorized internal access

---

# IDS/IPS

## Placement

- Behind Edge Firewall
- Internal network aggregation points

## Functions

- Detect anomalous behavior
- Identify command-and-control traffic
- Detect ransomware propagation attempts
- Block exploit signatures

Monitors both north-south and east-west flows.

---

# Logging Systems

Log Sources:

- Edge Firewall
- Internal Firewall
- NAC
- IDS/IPS
- Domain Controllers
- Critical Servers

Logs forwarded to centralized SIEM.

---

# SIEM Integration

SIEM provides:

- Event correlation
- Behavioral analytics
- Alert generation
- Threat hunting capabilities
- Forensic investigation support

Example Correlation Event:

- Multiple failed logins
- Followed by unusual SMB scanning
- Followed by privilege escalation

Alert generated for SOC response.

---

# Deny-by-Default Policy Model

All inter-VLAN traffic follows:

> Implicit DENY ALL

Only explicitly approved traffic is allowed.

## Concrete Firewall Rule Example

Example: Allow HTTPS from Users to Application Servers only.

Rule 10:

Source: VLAN10 (User)

Destination: VLAN20 (App Servers)

Protocol: TCP

Port: 443

Action: ALLOW

Log: Enabled

Implicit rule at bottom:

Rule 999:

Source: ANY

Destination: ANY

Action: DENY

Log: Enabled


This enforces least privilege and prevents unauthorized lateral movement.

---

# Performance vs Inspection Depth Tradeoff

Security inspection introduces latency.

Considerations:

- Deep packet inspection increases CPU usage
- TLS decryption impacts throughput
- IPS signature inspection adds processing overhead

Tradeoff:

| High Inspection | High Performance |
|----------------|-----------------|
| More security visibility | Lower latency |
| Greater threat detection | Reduced CPU load |
| Higher operational cost | Improved user experience |

Design must balance:

- Critical asset protection
- Application performance requirements
- Hardware capacity planning

High-risk segments (e.g., database tier) receive deeper inspection.

---

# Residual Risk

Even with full enforcement architecture, risks remain:

- Zero-day exploits bypassing signatures
- Stolen valid credentials (insider misuse)
- Encrypted malware traffic evading inspection
- Misconfigured firewall rules
- Supply chain compromise
- Cloud lateral movement outside on-prem controls

Mitigation requires:

- Continuous monitoring
- Regular rule audits
- Threat intelligence updates
- Security awareness training
- Incident response readiness

Zero-Trust reduces risk — it does not eliminate it.

---

# Architecture Diagram

<img width="1024" height="559" alt="image" src="https://github.com/user-attachments/assets/db604439-d3f1-49e5-9767-93fdad57504d" />



 NAC enforced at access layer
 
IDS/IPS monitoring internal aggregation

All logs forwarded to SIEM

Implicit Deny Between All Segments


---

# Summary

This layered enforcement architecture provides:

- North-South protection at the edge
- East-West inspection internally
- VLAN-based segmentation
- Ransomware blast radius containment
- Identity-based access control
- Centralized monitoring and analytics
- Deny-by-default policy enforcement

The model assumes breach and limits impact through segmentation, inspection, and continuous monitoring.
