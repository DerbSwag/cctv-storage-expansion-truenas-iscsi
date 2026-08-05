# Runbook: TrueNAS VM Recovery

Use this when the TrueNAS VM is powered off, rebooted, or unreachable.

## Recovery Steps

1. Check that the Windows Hyper-V host is powered on.
2. Check that the host is not sleeping or hibernating.
3. Check free space on the physical drives storing VHDX files.
4. Start the TrueNAS VM in Hyper-V.
5. Wait until the Web UI is available at `https://10.10.10.20`.
6. Confirm all pools are online:

```bash
zpool status
```

7. Confirm iSCSI is listening:

```bash
sudo ss -lntp | grep 3260
```

8. Confirm Hikvision Disk 17-19 return to `Normal`.

## Do Not

- Do not format IP SAN disks.
- Do not create pools over old disks.
- Do not wipe disks.
- Do not delete iSCSI targets during recovery.

