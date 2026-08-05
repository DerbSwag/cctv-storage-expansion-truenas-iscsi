# TrueNAS IP SAN Integration for Hikvision NVR

Private infrastructure documentation for expanding a Hikvision NVR with external storage using TrueNAS Community Edition, Microsoft Hyper-V, ZFS, iSCSI/IP SAN, and FortiGate inter-VLAN routing.

## Project Overview

This project designs and implements additional recording storage for a Hikvision Hikvision NVR NVR. TrueNAS runs as a virtual machine on Microsoft Hyper-V and presents three ZFS-backed iSCSI targets to the NVR. The NVR connects to these targets using Hikvision's `IP SAN` storage type across routed VLANs through a FortiGate firewall.

The first proof of concept used NFS. Packet captures showed that the NVR requested NFSv3 over UDP, while the deployed TrueNAS system exposed NFS on TCP 2049. The production design therefore moved to iSCSI, which the NVR supports as `IP SAN`.

Final result: 3 iSCSI targets attached to the NVR, adding 1,620 GB of external recording capacity.

## Current Result

| NVR Disk | TrueNAS Pool | Zvol | iSCSI Target | Capacity | Status |
|---|---|---|---|---:|---|
| Disk 17 | `NVR_CCTV_3` | `NVR_IPSAN_900` | `target-900` | 680 GB | Normal / R/W |
| Disk 18 | `CCTV` | `NVR_IPSAN_800` | `target-800` | 600 GB | Normal / R/W |
| Disk 19 | `NVR_CCTV` | `NVR_IPSAN_460` | `target-460` | 340 GB | Normal / R/W |

Total external storage added:

```text
680 GB + 600 GB + 340 GB = 1,620 GB
```

## Architecture

```text
                     LAN / Server Network
                        10.10.10.0/24
â”Œâ”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”
â”‚ Windows Hyper-V Host                                       â”‚
â”‚                                                            â”‚
â”‚ TrueNAS VM                                                 â”‚
â”‚ IP: 10.10.10.20                                           â”‚
â”‚ CPU: 2 vCPU                                                â”‚
â”‚ RAM: 8 GB                                                  â”‚
â”‚                                                            â”‚
â”‚ â”œâ”€â”€ Pool: CCTV                                             â”‚
â”‚ â”‚   â”œâ”€â”€ Virtual Disk: 800 GiB                              â”‚
â”‚ â”‚   â”œâ”€â”€ Usable Pool: 771.25 GiB                            â”‚
â”‚ â”‚   â”œâ”€â”€ Zvol: NVR_IPSAN_800                                â”‚
â”‚ â”‚   â”œâ”€â”€ Zvol Size: 600 GiB                                 â”‚
â”‚ â”‚   â””â”€â”€ iSCSI Target: target-800                        â”‚
â”‚ â”‚                                                          â”‚
â”‚ â”œâ”€â”€ Pool: NVR_CCTV                                         â”‚
â”‚ â”‚   â”œâ”€â”€ Virtual Disk: 460 GiB                              â”‚
â”‚ â”‚   â”œâ”€â”€ Usable Pool: 441.88 GiB                            â”‚
â”‚ â”‚   â”œâ”€â”€ Zvol: NVR_IPSAN_460                                â”‚
â”‚ â”‚   â”œâ”€â”€ Zvol Size: 340 GiB                                 â”‚
â”‚ â”‚   â””â”€â”€ iSCSI Target: target-460                        â”‚
â”‚ â”‚                                                          â”‚
â”‚ â””â”€â”€ Pool: NVR_CCTV_3                                       â”‚
â”‚     â”œâ”€â”€ Virtual Disk: 900 GiB                              â”‚
â”‚     â”œâ”€â”€ Usable Pool: 868.25 GiB                            â”‚
â”‚     â”œâ”€â”€ Zvol: NVR_IPSAN_900                                â”‚
â”‚     â”œâ”€â”€ Zvol Size: 680 GiB                                 â”‚
â”‚     â””â”€â”€ iSCSI Target: target-900                        â”‚
â””â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”¬â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”˜
                              â”‚
                              â”‚ iSCSI TCP 3260
                              â”‚
                     FortiGate Firewall
                     Inter-VLAN Routing
                              â”‚
                              â–¼
                    CCTV Network
                    10.20.30.0/24

                    Hikvision Hikvision NVR
                    IP: 10.20.30.40
                    Gateway: 10.20.30.1
```

## Repository Map

| Path | Purpose |
|---|---|
| `docs/architecture.md` | Network, VM, storage, and traffic architecture |
| `docs/implementation-guide.md` | Build procedure from Hyper-V to Hikvision |
| `docs/storage-design.md` | Pool, zvol, target, and capacity planning |
| `docs/troubleshooting.md` | NFS, iSCSI, TrueNAS, and FortiGate troubleshooting |
| `docs/disaster-recovery.md` | Recovery process when TrueNAS VM or targets go offline |
| `docs/security-considerations.md` | Risk and hardening notes |
| `docs/lessons-learned.md` | Key findings from the NFS and iSCSI implementation |
| `commands/` | Operational command references |
| `runbooks/` | Short incident response procedures |

## Resume Summary

à¸ à¸²à¸©à¸²à¹„à¸—à¸¢:

à¸­à¸­à¸à¹à¸šà¸šà¹à¸¥à¸°à¸•à¸´à¸”à¸•à¸±à¹‰à¸‡à¸£à¸°à¸šà¸š External Storage à¸ªà¸³à¸«à¸£à¸±à¸š Hikvision NVR à¹‚à¸”à¸¢à¹ƒà¸Šà¹‰ TrueNAS Community Edition à¸šà¸™ Microsoft Hyper-V à¹€à¸Šà¸·à¹ˆà¸­à¸¡à¸•à¹ˆà¸­à¸œà¹ˆà¸²à¸™ iSCSI/IP SAN à¸‚à¹‰à¸²à¸¡ VLAN à¸”à¹‰à¸§à¸¢ FortiGate Firewall à¸ªà¸£à¹‰à¸²à¸‡ ZFS Pool à¹à¸¥à¸° iSCSI Target à¸ˆà¸³à¸™à¸§à¸™ 3 à¸Šà¸¸à¸” à¹€à¸žà¸´à¹ˆà¸¡à¸žà¸·à¹‰à¸™à¸—à¸µà¹ˆà¸šà¸±à¸™à¸—à¸¶à¸à¸£à¸§à¸¡ 1.62 TB à¸žà¸£à¹‰à¸­à¸¡à¸§à¸´à¹€à¸„à¸£à¸²à¸°à¸«à¹Œ NFS compatibility, RPC traffic, routing, firewall policy à¹à¸¥à¸° iSCSI session à¸”à¹‰à¸§à¸¢ tcpdump, rpcinfo à¹à¸¥à¸° FortiGate Debug Flow

English:

Designed and implemented an external storage solution for a Hikvision NVR using TrueNAS Community Edition hosted on Microsoft Hyper-V. Deployed three ZFS-backed iSCSI targets across VLANs through a FortiGate firewall, adding 1.62 TB of recording capacity. Troubleshot NFS compatibility, RPC traffic, routing, firewall policies, and iSCSI connectivity using tcpdump, rpcinfo, and FortiGate debug flow.

