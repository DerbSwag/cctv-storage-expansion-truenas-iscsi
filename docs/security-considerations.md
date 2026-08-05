# Security Considerations

## Current Security Controls

- iSCSI targets are restricted to `10.20.30.40/32`.
- Authentication is disabled for simplicity during the initial deployment.
- FortiGate local policy has NAT disabled for NVR-to-TrueNAS traffic.
- iSCSI targets are not exposed to the internet.

## Recommended Hardening

Replace the broad local `ALL` service policy with a dedicated iSCSI policy:

```text
Source:      10.20.30.40
Destination: 10.10.10.20
Service:     TCP 3260
NAT:         Disabled
Log:         Enabled
```

Additional recommendations:

- Keep TrueNAS Web UI accessible only from admin networks.
- Keep Hikvision and TrueNAS firmware updated after testing compatibility.
- Use UPS for Hyper-V host, NVR, switch, and FortiGate.
- Monitor physical host drive capacity.
- Back up TrueNAS configuration after storage changes.
- Avoid using iSCSI storage as a backup. It is only recording capacity.

## Known Limitations

### Single-Disk Stripe

Each pool uses one disk. If a disk fails, data in that pool can be lost.

### Single Point of Failure

If the Hyper-V host, Windows OS, TrueNAS VM, or physical drive fails, the iSCSI targets become unavailable.

### Dynamic VHDX

Dynamic VHDX files grow as data is written. If the host disk fills, TrueNAS pools and Hikvision recording can fail.

### No SMART Visibility in TrueNAS

TrueNAS sees virtual disks, not the physical HDDs. SMART and temperature checks must be done on the Windows Hyper-V host.

