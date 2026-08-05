# Storage Design

## Final Storage Layout

| Pool | Virtual Disk | Usable Pool | Zvol | Zvol Size | iSCSI Target | Hikvision Disk |
|---|---:|---:|---|---:|---|---|
| `CCTV` | 800 GiB | 771.25 GiB | `NVR_IPSAN_800` | 600 GiB | `hikvision-800` | Disk 18 |
| `NVR_CCTV` | 460 GiB | 441.88 GiB | `NVR_IPSAN_460` | 340 GiB | `hikvision-460` | Disk 19 |
| `NVR_CCTV_3` | 900 GiB | 868.25 GiB | `NVR_IPSAN_900` | 680 GiB | `hikvision-900` | Disk 17 |

Target IQNs:

```text
iqn.2005-10.org.freenas.ctl:hikvision-800
iqn.2005-10.org.freenas.ctl:hikvision-460
iqn.2005-10.org.freenas.ctl:hikvision-900
```

Total external storage added to the NVR:

```text
680 GB + 600 GB + 340 GB = 1,620 GB
```

## ZFS Capacity Planning

Each iSCSI LUN is a thick-provisioned zvol. TrueNAS reserves the allocated zvol capacity up front, so pool usage appears high even when the NVR has not filled the disk internally.

| Pool | Usable | Zvol | Pool Available | Usage |
|---|---:|---:|---:|---:|
| `CCTV` | 771.25 GiB | 600 GiB | 161.79 GiB | 79% |
| `NVR_CCTV` | 441.88 GiB | 340 GiB | 96.54 GiB | 78.2% |
| `NVR_CCTV_3` | 868.25 GiB | 680 GiB | 177.61 GiB | 79.5% |

The available capacity is intentionally reserved for:

- ZFS metadata
- Copy-on-write overhead
- Fragmentation control
- Internal allocation overhead
- Reduced risk of a full pool
- Stable continuous-write CCTV workload

## iSCSI Settings

The targets use these settings:

| Setting | Value |
|---|---|
| Extent type | Device |
| Backing storage | Zvol |
| Sharing platform | Legacy OS |
| Zvol block size | 16 KiB |
| Sync | Standard |
| Compression | LZ4 |
| Deduplication | Off |
| Authentication | None |
| Authorized network | 192.168.101.139/32 |
| Portal | Shared portal on TCP 3260 |

## Risk Note

Each pool is a single-disk stripe:

```text
1 physical disk -> 1 VHDX -> 1 ZFS pool -> 1 zvol -> 1 iSCSI target
```

If a physical disk, VHDX, or pool fails, data in that pool can be lost. This design expands recording capacity, but it is not RAID and it is not a backup.

