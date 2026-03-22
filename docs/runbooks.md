# Runbooks

This page collects short operational procedures for common recovery and maintenance tasks in the homelab.

## Purpose

These runbooks are meant to make recovery and maintenance faster, more repeatable, and less dependent on memory.

Over time, larger procedures may be broken out into their own dedicated pages.

---

## 1. Power Outage Recovery Check

### Goal
Confirm that core infrastructure returns cleanly after a power event and identify anything requiring manual intervention.

### Check Order
1. Confirm router/firewall is powered on and reachable
2. Confirm ISP connection is restored
3. Confirm main server is powered on
4. Confirm NAS is powered on
5. Confirm DNS is working for clients
6. Confirm key services are reachable

### Validate
- OPNsense reachable
- LAN clients can access the network
- DNS resolution is working
- `seranogenomics` is online
- `nas` is online
- Home Assistant is reachable
- Dockerized services are reachable as expected

### Common Failure Points
- main server stalls during remote unlock/initramfs
- NAS may fail to boot cleanly after power restoration
- some services may need manual validation even if the host is online

---

## 2. Main Server Remote Unlock Recovery

### Goal
Recover the main server if it stalls during initramfs/dropbear remote unlock.

### Symptoms
- server powers on but does not finish booting
- normal services do not come online
- preboot remote unlock is not reachable as expected

### Recovery
1. Confirm the server actually powered on
2. Check whether preboot/dropbear networking is reachable
3. If remote unlock is unavailable, use local/manual unlock access
4. Complete unlock process
5. Confirm the system continues booting normally
6. Verify key services after boot

### Follow-Up
- document what failed
- note whether networking or unlock itself was the blocker
- update preboot/dropbear cleanup notes if needed

---

## 3. NAS Power-Loss Boot Recovery

### Goal
Recover the NAS if it does not boot properly after a power event.

### Symptoms
- NAS does not start normally after power is restored
- system appears stuck or unresponsive
- normal boot only resumes after full power drain/manual intervention

### Recovery
1. Confirm NAS power state
2. If needed, fully remove power
3. Drain residual power using the current known-good recovery method
4. Restore power and attempt normal boot again
5. Confirm storage/services return as expected

### Follow-Up
- note whether the issue repeated
- continue BIOS/CMOS/power-loss investigation
- update hardware reliability notes

---

## 4. VPN Connectivity Check

### Goal
Validate that remote VPN access is working correctly.

### Validate
- VPN client can connect
- VPN client receives expected subnet access
- LAN resources are reachable
- internet access behaves as expected for the current configuration
- DNS resolution works correctly for the VPN client

### If Problems Appear
1. confirm VPN tunnel is up
2. confirm client received expected configuration
3. review firewall/routing behavior
4. confirm DNS path is working
5. compare behavior against documented expected state

---

## 5. DNS Troubleshooting Check

### Goal
Confirm that local DNS is functioning correctly.

### Current DNS Path
- clients -> AdGuard Home on `192.168.5.1:53`
- Unbound on `5353`
- upstream resolver path beyond that

### Validate
- clients can resolve normal domains
- clients can resolve expected local resources
- AdGuard is reachable
- Unbound is responding as expected

### If Problems Appear
1. confirm network connectivity to the router
2. confirm AdGuard is listening on the expected address/port
3. confirm Unbound is available
4. test resolution from a known LAN client
5. check whether the problem is all clients or only specific devices

---

## 6. Monthly Maintenance Check

### Goal
Keep core systems updated on a predictable cadence.

### Priority Systems
- OPNsense
- `nas`
- `seranogenomics`

### Checklist
- review available updates
- apply updates during a low-risk window
- confirm system returns normally after reboot if required
- validate DNS, VPN, storage, and core services after updates
- record anything unusual

---

## Future Improvements

This page will eventually expand into more detailed procedures for:
- service restart/recovery
- Docker stack maintenance
- outage validation
- NAS-specific recovery
- VPN troubleshooting
- backup/restore workflows
