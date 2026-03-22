# Hardware

This page documents the major physical components in the homelab and their roles in the overall environment.

## Core Network Hardware

### ISP Modem / Router
**Role:** Internet handoff and Wi-Fi access point  
**Notes:**
- Provides the current internet connection
- Internet service is currently **2.5 Gbps**
- Wi-Fi is handled directly by the ISP modem/router
- No separate access point is currently connected to the switch

### OPNsense Router / Firewall
**Hostname:** `texhnolyze.home.arpa`  
**Role:** Primary routing, firewalling, and VPN control  
**Notes:**
- Main network control point for the homelab
- Handles LAN access, firewall behavior, and VPN connectivity
- One of the most critical pieces of infrastructure in the environment

### 2.5 GbE Switch
**Role:** Core wired switching layer  
**Notes:**
- Connects the main wired infrastructure
- Used for higher-speed connectivity between core devices
- Important part of the current 2.5 gig network design

## Core Compute and Storage

### Main Server
**Hostname:** `seranogenomics`  
**IP:** `192.168.5.7`  
**Role:** Primary Linux server / application host  
**Notes:**
- Runs core self-hosted services and Docker workloads
- One of the main operational focus points in the lab
- Recovery behavior after outages is an important documentation area

### NAS
**Hostname:** `nas`  
**IP:** `192.168.5.8`  
**Role:** Primary storage platform  
**Notes:**
- Stores media, archive, and infrastructure-related data
- Current known weakness is power-loss / boot reliability
- Important for both storage and resilience planning

## Virtualized / Attached Systems

### Home Assistant VM
**IP:** `192.168.5.14`  
**Role:** Home automation platform  
**Notes:**
- Separate VM in the environment
- Long-lived service that is part of the broader self-hosting stack

### Gaming PC
**Hostname:** `ZACH-PC`  
**IP:** `192.168.5.15`  
**Role:** Client endpoint / workstation  
**Notes:**
- Useful as a known-good LAN client for testing connectivity and validation

## Power / Reliability Hardware

### UPS
**Role:** Backup power and controlled shutdown support  
**Notes:**
- Works with NUT for outage handling
- Successfully validated for shutdown behavior
- Recovery testing exposed weaknesses in startup behavior on some systems

## Current Hardware Priorities

- improve post-outage recovery behavior
- continue NAS power-loss / boot investigation
- improve remote unlock reliability for the main server
- maintain clean documentation of how critical hardware fits together

## Future Hardware Considerations

Potential future hardware-related work includes:
- NAS hardware refresh
- additional storage expansion
- better cooling/airflow for the network closet
- camera integration and documentation
