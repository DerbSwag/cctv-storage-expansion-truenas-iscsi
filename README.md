# CCTV Storage Expansion with TrueNAS iSCSI

Portfolio case study documenting a lab implementation that expanded recording capacity for a Hikvision NVR. The solution repurposed approved legacy hardware: a retired PC became a Hyper-V host and available HDDs became dedicated storage for a TrueNAS VM.

The repository uses anonymized network addresses, target names, and firewall identifiers. It does not contain production credentials, real company information, or screenshots with sensitive details.

## Problem

The NVR needed more recording capacity, but replacing the NVR or purchasing a new NAS was not immediately practical. The objectives were to:

- extend storage with available hardware;
- keep the NVR unchanged;
- connect storage across routed VLANs; and
- leave a repeatable path for future capacity expansion.

## Solution

TrueNAS Community Edition runs as a virtual machine on Microsoft Hyper-V. It presents three ZFS-backed iSCSI targets to the NVR, which discovers and attaches them through Hikvision's `IP SAN` storage type. A FortiGate firewall routes the storage traffic between the server and CCTV VLANs.

The first proof of concept used NFS. Packet captures showed that the NVR requested NFSv3 over UDP, while the evaluated TrueNAS configuration exposed NFS on TCP 2049. The implementation therefore moved to iSCSI/IP SAN.

## Result

| NVR Disk | TrueNAS Pool | Zvol | iSCSI Target | Capacity | State |
|---|---|---|---|---:|---|
| Disk 17 | `NVR_CCTV_3` | `NVR_IPSAN_900` | `target-900` | 680 GB | Normal / R/W |
| Disk 18 | `CCTV` | `NVR_IPSAN_800` | `target-800` | 600 GB | Normal / R/W |
| Disk 19 | `NVR_CCTV` | `NVR_IPSAN_460` | `target-460` | 340 GB | Normal / R/W |

```text
680 GB + 600 GB + 340 GB = 1,620 GB of additional recording capacity
```

## Architecture

```text
Server VLAN (10.10.10.0/24)
  Windows Hyper-V host
    TrueNAS VM (10.10.10.20)
      CCTV       -> NVR_IPSAN_800 -> target-800 -> 600 GB
      NVR_CCTV   -> NVR_IPSAN_460 -> target-460 -> 340 GB
      NVR_CCTV_3 -> NVR_IPSAN_900 -> target-900 -> 680 GB
                  |
                  | iSCSI TCP 3260
                  v
  FortiGate inter-VLAN routing
                  |
                  v
CCTV VLAN (10.20.30.0/24)
  Hikvision NVR (10.20.30.40)
```

## Scope and Limitations

This is a lab implementation built from approved legacy hardware, not a high-availability storage design. Each pool uses a single disk and Dynamic VHDX files, so it has no RAID redundancy and is not a backup solution. See [Security Considerations](docs/security-considerations.md) and [Disaster Recovery](docs/disaster-recovery.md) before applying this design elsewhere.

## Documentation

| Path | Purpose |
|---|---|
| [Architecture](docs/architecture.md) | Network, VM, storage, and traffic design |
| [Implementation Guide](docs/implementation-guide.md) | Build procedure from Hyper-V to Hikvision |
| [Storage Design](docs/storage-design.md) | Pool, zvol, target, and capacity planning |
| [Troubleshooting](docs/troubleshooting.md) | NFS, iSCSI, TrueNAS, and FortiGate investigation |
| [Disaster Recovery](docs/disaster-recovery.md) | VM and target recovery process |
| [Security Considerations](docs/security-considerations.md) | Risks and hardening notes |
| [Lessons Learned](docs/lessons-learned.md) | Findings from the NFS and iSCSI implementation |

## Resume Summary

Designed and implemented a lab-based CCTV storage expansion using approved legacy hardware. Deployed TrueNAS on Microsoft Hyper-V and integrated three ZFS-backed iSCSI/IP SAN targets with a Hikvision NVR across VLANs through a FortiGate firewall, adding 1.62 TB of recording capacity. Diagnosed NFS compatibility and iSCSI connectivity with packet capture, RPC inspection, and firewall flow analysis.
