# UPS / NUT Setup, Troubleshooting, and Shutdown Testing

This page documents the initial setup of the CyberPower CP1500PFCLCD UPS, the deployment of NUT across the homelab, the first real shutdown test, and the recovery issues that showed up afterward.

## Summary

The goal of this phase was to integrate the new UPS into the homelab so power outages would trigger a clean, delayed shutdown instead of a hard power loss.

The design centered on:

- NAS as the NUT server with the UPS connected by USB
- `seranogenomics` as the NUT client
- critical network gear and compute devices connected to battery-backed outlets
- a delayed shutdown after about five minutes on battery
- cancellation of shutdown if utility power returned before the timer expired

A real unplug test confirmed that the broad shutdown behavior worked, but it also exposed unresolved startup and recovery issues on both `seranogenomics` and `nas`.

## Main Goal

Set up the new **CyberPower CP1500PFCLCD** UPS with **NUT** so the homelab could survive power outages gracefully.

The intended model was:

- **NAS = NUT server** (UPS connected by USB)
- **SG / seranogenomics = NUT client**
- on utility power loss, wait roughly 300 seconds on battery
- if power remains out, perform a clean automatic shutdown
- if utility power returns before the timer expires, cancel shutdown

## Environment

### UPS
- Model: `CyberPower CP1500PFCLCD`

### NUT Design
- NAS as the primary UPS controller / NUT server
- `seranogenomics` as the NUT client

### Critical Protected Devices
- NAS
- `seranogenomics`
- OPNsense router
- switch

### Physical Constraints
- closet space was limited
- UPS had to be placed temporarily rather than in a final ideal position
- side vent layout meant the UPS needed to remain upright

## Starting State

At the beginning of this phase:

- the UPS had arrived but was not yet integrated
- no NUT configuration existed on NAS or `seranogenomics`
- the homelab did not yet have graceful outage handling
- the physical layout still needed a temporary but safe solution
- later in the process, partial NUT config existed, but the real-world shutdown path still needed validation

## Problems Addressed

This phase covered several related problems:

- safe physical UPS placement and ventilation
- deciding which devices should use battery-backed outlets
- determining the correct USB connection path
- UPS detection and discovery on NAS
- manual NUT server setup on NAS
- NUT client setup on `seranogenomics`
- implementing delayed shutdown behavior
- validating real outage behavior by unplugging the UPS from the wall
- understanding what broke during restore and reboot afterward

## Changes Made

## 1. Installed the UPS Physically

The UPS was installed in a temporary but workable physical location.

### Physical Notes
- placed temporarily on a cutting board for stability and airflow
- kept upright because of the side vent layout

### Result
The UPS was installed safely enough to begin testing and integration.

## 2. Chose the Battery-Backed Outlet Plan

Critical homelab gear was assigned to the UPS battery-backed outlets.

### Devices Placed on `SURGE + BATTERY`
- NAS
- `seranogenomics`
- OPNsense/router
- switch

### Result
The most important homelab gear would remain powered during an outage long enough for coordinated shutdown.

## 3. Connected UPS USB to NAS

The USB link from the UPS was connected to the NAS.

### Notes
- UPS USB connected to a rear motherboard USB port on the NAS
- USB2 was considered fine for this purpose

### Result
The NAS became the correct place to host UPS monitoring and NUT server logic.

## 4. Installed NUT on NAS

NUT packages were installed on the NAS.

### Installed Packages
- `nut`
- `nut-client`
- `nut-server`

### Result
The NAS had the necessary software foundation to act as the NUT server.

## 5. Worked Around Failed Auto-Discovery

UPS auto-discovery was attempted but did not work cleanly.

### Discovery Result
`lsusb` detected the CyberPower UPS, but:

- `nut-scanner -U` failed
- USB runtime library support was missing for scanner-based detection

### Result
The setup had to proceed with manual configuration instead of relying on automatic USB discovery.

## 6. Configured NAS as the NUT Server

The UPS was configured manually on the NAS and exposed over the network to clients.

### Result
The NAS successfully served UPS state to the network.

## 7. Configured `seranogenomics` as the NUT Client

`seranogenomics` was configured to monitor the NAS-hosted UPS service and obey shutdown behavior.

### Result
The main server could react to UPS state from the NAS rather than needing direct USB access itself.

## 8. Implemented Delayed Shutdown Behavior

The intended logic was a delayed shutdown instead of immediate power-off on battery.

### Intended Behavior
- power loss -> switch to battery
- wait about 300 seconds
- if still on battery -> clean shutdown
- if utility power returns before timer expires -> cancel shutdown

### Result
The homelab now had a workable planned outage-handling model.

## 9. Performed a Real Outage Test

A real unplug test was used to validate the design.

### Test Method
- UPS unplugged from wall power
- devices remained connected to battery-backed outlets
- observed battery state and delayed shutdown behavior

### Result
The broad NUT-driven shutdown flow worked as intended.

## What Worked

The following parts of the setup worked successfully:

- the UPS was detected by the NAS over USB
- NUT packages installed successfully on the NAS
- NAS successfully served UPS state to the network
- `seranogenomics` successfully acted as a NUT client
- the unplug test confirmed the basic shutdown flow worked
- systems stayed up briefly on battery
- shutdown occurred after the intended delay window
- the overall UPS/NUT protection concept was validated

This established that the homelab could perform a planned graceful shutdown instead of suffering an uncontrolled hard power loss. :contentReference[oaicite:1]{index=1}

## What Broke or Regressed

A few important problems showed up during testing and restore.

### `nut-scanner -U` Auto-Discovery
Auto-discovery failed because the required runtime USB library support was missing.

### `seranogenomics` Restore Failure
After the live outage test, `seranogenomics` did not return cleanly on its own:

- it stalled at preboot/dropbear remote unlock
- preboot networking was not reliably reachable

### NAS Restore Failure
After the same test, the NAS also had a restore-time problem:

- it did not boot normally until power was fully drained using the rear PSU switch

### Remaining Unresolved Issues
- NAS BIOS / CMOS / power-loss persistence issue remained unresolved
- SG preboot / dropbear remote unlock networking remained unresolved

So while the shutdown behavior worked, the startup and recovery behavior after outage still needed more work. :contentReference[oaicite:2]{index=2}

## Validation

### UPS Detection on NAS

```bash
lsusb | egrep -i 'cyber|power|ups' || lsusb | tail -n +1
````

Example observed output:

```text
Bus 001 Device 002: ID 0764:0601 Cyber Power System, Inc. PR1500LCDRT2U UPS
```

### NUT Installation on NAS

```bash
sudo apt-get update
sudo apt-get install -y nut nut-client nut-server
```

### Failed Scanner Discovery

```bash
sudo nut-scanner -U
```

Example error pattern:

```text
Cannot load USB library (libusb-1.0.so) : file not found. USB search disabled.
... USB scan not enabled: library not detected.
```

### Physical Notes Worth Preserving

```text
- Use UPS "SURGE + BATTERY" outlets for NAS / SG / router / switch
- Keep UPS upright because of side vents
- UPS USB -> NAS rear USB port
- USB2 is fine
```

### Power-Outage Test Findings

```text
- UPS/NUT shutdown behavior worked as expected
- on restore, router came back
- SG powered on but stalled at dropbear/initramfs remote unlock and its preboot IP was not discoverable
- NAS did not boot until power was fully drained via rear PSU switch, then booted normally
- confirmed:
  - NUT shutdown path = working
  - SG preboot networking = still broken
  - NAS post-outage boot persistence = still broken
```

## Ending State

By the end of this phase:

* the UPS was integrated enough to protect the critical homelab gear
* NAS was functioning as the NUT server with the UPS attached by USB
* `seranogenomics` was functioning as a NUT client
* a real unplug test demonstrated that graceful shutdown after a battery-runtime delay worked
* the homelab now had a working baseline for outage handling

The remaining issues were no longer about whether NUT worked. They were about:

* SG preboot remote unlock after restore
* NAS boot persistence after power loss
* general cleanup and hardening after the shutdown path was proven

## Lessons Learned

* graceful shutdown and clean restore are two different problems
* UPS integration is not complete until both the shutdown path and the recovery path are understood
* manual NUT configuration can still be perfectly workable when scanner discovery fails
* a live unplug test is one of the fastest ways to reveal what still is not resilient
* startup and preboot reliability matter just as much as shutdown logic

## Follow-Up Items

* clean up SG preboot/dropbear networking so remote unlock works reliably after outage and power restore
* investigate and fix NAS BIOS / CMOS / power-loss boot persistence
* optionally install missing USB runtime library support and retry `nut-scanner -U`
* reconfirm exact timer/cancel behavior after future config changes
* rerun a shorter controlled validation test once boot recovery issues are improved
* eventually move the UPS and gear to a cleaner long-term physical layout with better space and airflow

## Commands Worth Preserving

### UPS Detection

```bash
lsusb | egrep -i 'cyber|power|ups' || lsusb | tail -n +1
```

### Install NUT on NAS

```bash
sudo apt-get update
sudo apt-get install -y nut nut-client nut-server
```

### Scanner Attempt

```bash
sudo nut-scanner -U
```

### Important Physical Notes

```text
- SURGE + BATTERY outlets for NAS / SG / router / switch
- Keep UPS upright
- UPS USB -> NAS rear USB port
- USB2 is fine
```

## Status

Completed as the initial UPS/NUT deployment and shutdown-validation phase, with restore/recovery hardening still remaining.
