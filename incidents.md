# Incidents and Fixes

## Power outage recovery test
### What happened
A simulated/real outage was used to validate UPS/NUT shutdown behavior.

### Findings
- UPS/NUT shutdown behavior worked as expected
- router came back online
- main server powered on but stalled at remote unlock in initramfs
- NAS had power-loss boot issues and required manual intervention

### Follow-up work
- improve preboot/dropbear networking
- continue NAS BIOS/CMOS persistence investigation
- document recovery steps more clearly
