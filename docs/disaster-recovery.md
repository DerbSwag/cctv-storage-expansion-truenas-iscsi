# Disaster Recovery

## If the TrueNAS VM Is Down

1. Check the Windows Hyper-V host.
2. Check physical drive free space.
3. Start the TrueNAS VM.
4. Wait until the Web UI is available at `https://10.10.10.20`.
5. Confirm all pools are online:

```bash
zpool status
```

6. Confirm iSCSI is running:

```bash
sudo ss -lntp | grep 3260
```

7. Confirm targets still exist:

```bash
sudo midclt call iscsi.target.query
sudo midclt call iscsi.extent.query
```

8. Check Hikvision HDD Management.
9. Confirm Disk 17, Disk 18, and Disk 19 return to `Normal`.

## Do Not

- Do not format an IP SAN disk when it is only offline.
- Do not create a new pool over an existing disk.
- Do not wipe disks during recovery.
- Do not remove targets or extents during recovery.
- Do not recreate the TrueNAS VM unless the VHDX files and pool state have been verified.

## Auto-Start Recommendations

Hyper-V:

```text
Automatic Start Action: Always start this virtual machine automatically
Startup Delay: 60-120 seconds
Automatic Stop Action: Shut down the guest operating system
```

TrueNAS:

```text
System -> Services -> iSCSI -> Start Automatically: Enabled
```

Windows host:

```powershell
powercfg /change standby-timeout-ac 0
powercfg /change hibernate-timeout-ac 0
powercfg /hibernate off
```

