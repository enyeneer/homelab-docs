## UPS / Power Outage Recovery Test

### Objective
Validate shutdown and recovery behavior during a power-loss scenario.

### Environment
- OPNsense router/firewall
- main server: seranogenomics
- NAS
- UPS/NUT-managed shutdown flow

### What happened
A power outage test was performed to confirm shutdown behavior and observe service recovery after power restoration.

### Results
- UPS/NUT shutdown behavior worked as expected
- router returned successfully
- main server powered on but stalled at dropbear/initramfs remote unlock
- NAS did not boot cleanly until full power drain/manual intervention

### Lessons learned
- shutdown automation is working
- preboot remote unlock networking needs improvement
- NAS power-loss boot reliability remains a weak point

### Follow-up actions
- document remote unlock recovery procedure
- investigate NAS CMOS/BIOS persistence issue
- retest after remediation
