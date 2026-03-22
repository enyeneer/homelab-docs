# UPS / Power Outage Recovery Test

## Objective
Validate shutdown and recovery behavior during a power-loss event and identify weak points in startup/recovery.

## Environment
- Router / firewall: OPNsense (`texhnolyze.home.arpa`)
- Main server: `seranogenomics` (`192.168.5.7`)
- NAS: `nas` (`192.168.5.8`)
- LAN: `192.168.5.0/24`
- OpenVPN subnet: `192.168.6.0/24`
- DNS path: clients -> AdGuard Home on `192.168.5.1:53` -> upstream
- UPS / NUT in place for shutdown handling

## Test Goal
Confirm that critical systems shut down safely during outage conditions and recover cleanly when power returns.

## What Happened
A power outage / outage simulation was performed to verify UPS-triggered shutdown behavior and observe service recovery on restore.

## Results
- UPS / NUT shutdown behavior worked as expected
- Router returned successfully after power restoration
- Main server powered on, but stalled at dropbear / initramfs remote unlock
- NAS did not boot cleanly until power was fully drained via rear PSU switch, then booted normally after manual intervention

## What Worked
- UPS-triggered shutdown flow
- Router recovery after restore
- Main server auto power-on behavior appears functional

## Problems Found
- Preboot / dropbear networking was not reliably reachable for remote unlock
- NAS still has a power-loss / CMOS / BIOS persistence issue
- Recovery currently depends on some manual intervention

## Lessons Learned
- Shutdown automation is in much better shape than startup/recovery
- Preboot remote unlock remains one of the weakest links in the stack
- NAS boot reliability needs more investigation before the setup can be considered resilient

## Follow-up Actions
- Clean up preboot / dropbear networking
- Document the remote unlock recovery procedure
- Continue NAS BIOS / CMOS / power-loss boot investigation
- Retest after changes are made

## Status
Open / in progress
