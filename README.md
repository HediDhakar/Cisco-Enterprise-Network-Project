# Project Report: Enterprise Branch Network Implementation

- **Lead Engineer:** HediDhakar
- **Report Date:** 2025-11-20
- **Status:** Complete

## 1. Executive Summary

This document details the successful design, deployment, and hardening of a professional-grade branch office network. The project systematically transformed a basic, insecure network into a segmented, automated, centrally managed, and fully redundant infrastructure. The resulting network is engineered to withstand core equipment failure at both Layer 3 (Routing) and Layer 2 (Switching) with no manual intervention, meeting enterprise standards for security, resiliency, and manageability.

---

## 2. Network Schemas & IP Addressing

| VLAN ID | VLAN Name | Subnet | Gateway (HSRP Virtual IP) | Purpose |
| :--- | :--- | :--- | :--- | :--- |
| **10** | USERS | `192.168.10.0 /24` | `192.168.10.1` | General user workstations and clients. |
| **20** | SERVERS | `192.168.20.0 /24` | `192.168.20.1` | Protected servers and management systems. |
| **1** | DEFAULT | N/A | N/A | Used for Native VLAN on trunks. Unused for user data. |

---

## 3. Validation & Testing Procedures

The following tests were successfully conducted throughout the project to validate each configuration and prove system resiliency.

| Test Name | Action | Expected Result | Outcome |
| :--- | :--- | :--- | :--- |
| **Basic Connectivity** | `ping` from PC to its Gateway IP. | Successful replies. | **Success** |
| **Inter-VLAN Routing** | `ping` from PC in VLAN 10 to Server in VLAN 20. | Successful replies. | **Success** |
| **ACL Firewall** | `ping` from PC (VLAN 10) to a router's physical G0/0 interface IP. | "Destination host unreachable" or timeouts. | **Success** |
| **AAA/RADIUS Auth** | Log into router with personal `HediDhakar` credentials. | Successful login. | **Success** |
| **SSH Security** | Attempt router login via `ssh`. Then attempt via `telnet`. | SSH connects. Telnet is refused. | **Success** |
| **Port Security** | Connect an unauthorized device to a secured port. | Port transitions to `err-disabled` state. | **Success** |
| **HSRP Failover** | While running a continuous ping from a PC to the server, shut down the primary router (R1). | Pings drop for a few seconds, then automatically resume as R2 becomes Active. | **Success** |
| **STP Failover** | While running a continuous ping, shut down the active Root Port on the backup switch. | Link lights update. Pings drop for a few seconds, then automatically resume as the blocked port becomes active. | **Success** |

---

## 4. Phase-by-Phase Project Breakdown

### **Phase 0: Platform Establishment**
- **Strategic Hardware Upgrade:** The project began with the identification that the initial router's operating system lacked the necessary capabilities for enterprise security. The core routing platform was upgraded to provide the advanced feature set required for all subsequent enhancements.

### **Phase 1: Core Infrastructure & Segmentation**
- **Network Segmentation (VLANs):** The flat network was segmented into logical broadcast domains (`VLAN 10` for Users, `VLAN 20` for Servers) to isolate traffic and improve security.
- **Inter-VLAN Routing:** A **"Router on a Stick"** topology was implemented by configuring router sub-interfaces to act as the default gateway for each VLAN.
- **Trunking:** 802.1Q trunking was configured on all switch-to-router and switch-to-switch links to carry traffic for multiple VLANs.

### **Phase 2: Hardened Access & Automated Services**
- **Layer 2 Security Suite:** A robust defense against common internal threats was implemented.
    - **Port Security:** Switch access ports were locked down to a single MAC address, preventing unauthorized device connections.
    - **DHCP Snooping:** Deployed to neutralize rogue DHCP server attacks and ensure clients only receive IP addresses from the legitimate server.
- **Layer 3 Firewalling (ACLs):** An Access Control List was engineered and applied to the router to strictly regulate traffic, permitting only essential management traffic into the secure server VLAN.
- **DHCP Server Automation:** The primary router was configured as a centralized DHCP server for the user VLAN.

### **Phase 3: Professional Management & Monitoring**
- **Centralized Authentication (AAA):** The AAA framework was implemented with a **RADIUS server** to ensure all device logins are authenticated against a central authority using role-based credentials.
- **Secure Management (SSH):** The clear-text Telnet protocol was replaced with **Secure Shell (SSH)** to encrypt all remote management sessions.
- **Centralized Logging (Syslog):** Routers were configured to send all system event logs to a central Syslog server for auditing and forensics.
- **Proactive Monitoring (SNMP):** The SNMP agent was enabled on routers, allowing a Network Management Station to poll for real-time health and performance metrics.

### **Phase 4: Layer 3 Resiliency (High Availability)**
- **Router Redundancy (HSRP):** The router as a single point of failure was eliminated by deploying a second router and implementing the **Hot Standby Router Protocol (HSRP)**. A live failover test was performed, proving that the standby router automatically assumes all routing responsibilities upon primary router failure.

### **Phase 5: Layer 2 Resiliency (Redundant Switching Fabric)**
- **Fully Meshed Topology:** A fully redundant physical fabric was architected and cabled where routers and switches were dual-homed, creating no single point of failure in the network core.
- **Spanning Tree Protocol (STP) Mastery:** Layer 2 loops were deliberately introduced and managed with STP. `NATIVE_VLAN_MISMATCH` errors were resolved, and `show spanning-tree` was used to verify that STP correctly elected a **Root Bridge** and placed redundant links into a safe **Blocking (`BLK`)** state.
- **Live Failover Test:** A link failure was simulated, and STP was observed automatically re-converging and promoting the backup path to active, instantly healing the network fabric.