# Runbook: Storage Capacity Alert

Use this when TrueNAS pool usage or Windows host disk usage is high.

## TrueNAS Checks

```bash
zpool list
zfs list
zpool status
```

## Windows Host Checks

Check the physical drive that stores each VHDX:

```powershell
Get-Volume | Select-Object DriveLetter,FileSystemLabel,SizeRemaining,Size
```

## Guidance

- Keep ZFS pools near or below 80% usage.
- Do not expand zvols to consume all available pool capacity.
- Do not allow the Windows host drive storing a Dynamic VHDX to become full.
- Add a new physical disk and create a new pool/target rather than extending existing single-disk stripe pools.

