# TrueNAS IP SAN Integration for Hikvision NVR

Private infrastructure documentation for expanding a Hikvision NVR with external storage using TrueNAS Community Edition, Microsoft Hyper-V, ZFS, iSCSI/IP SAN, and FortiGate inter-VLAN routing.

## Project Overview

This project designs and implements additional recording storage for a Hikvision DS-7732NI-I4(B) NVR. TrueNAS runs as a virtual machine on Microsoft Hyper-V and presents three ZFS-backed iSCSI targets to the NVR. The NVR connects to these targets using Hikvision's `IP SAN` storage type across routed VLANs through a FortiGate firewall.

The first proof of concept used NFS. Packet captures showed that the NVR requested NFSv3 over UDP, while the deployed TrueNAS system exposed NFS on TCP 2049. The production design therefore moved to iSCSI, which the NVR supports as `IP SAN`.

Final result: 3 iSCSI targets attached to the NVR, adding 1,620 GB of external recording capacity.

## Current Result

| NVR Disk | TrueNAS Pool | Zvol | iSCSI Target | Capacity | Status |
|---|---|---|---|---:|---|
| Disk 17 | `NVR_CCTV_3` | `NVR_IPSAN_900` | `hikvision-900` | 680 GB | Normal / R/W |
| Disk 18 | `CCTV` | `NVR_IPSAN_800` | `hikvision-800` | 600 GB | Normal / R/W |
| Disk 19 | `NVR_CCTV` | `NVR_IPSAN_460` | `hikvision-460` | 340 GB | Normal / R/W |

Total external storage added:

```text
680 GB + 600 GB + 340 GB = 1,620 GB
```

## Architecture

```text
                     LAN / Server Network
                        192.168.1.0/24
┌────────────────────────────────────────────────────────────┐
│ Windows Hyper-V Host                                       │
│                                                            │
│ TrueNAS VM                                                 │
│ IP: 192.168.1.24                                           │
│ CPU: 2 vCPU                                                │
│ RAM: 8 GB                                                  │
│                                                            │
│ ├── Pool: CCTV                                             │
│ │   ├── Virtual Disk: 800 GiB                              │
│ │   ├── Usable Pool: 771.25 GiB                            │
│ │   ├── Zvol: NVR_IPSAN_800                                │
│ │   ├── Zvol Size: 600 GiB                                 │
│ │   └── iSCSI Target: hikvision-800                        │
│ │                                                          │
│ ├── Pool: NVR_CCTV                                         │
│ │   ├── Virtual Disk: 460 GiB                              │
│ │   ├── Usable Pool: 441.88 GiB                            │
│ │   ├── Zvol: NVR_IPSAN_460                                │
│ │   ├── Zvol Size: 340 GiB                                 │
│ │   └── iSCSI Target: hikvision-460                        │
│ │                                                          │
│ └── Pool: NVR_CCTV_3                                       │
│     ├── Virtual Disk: 900 GiB                              │
│     ├── Usable Pool: 868.25 GiB                            │
│     ├── Zvol: NVR_IPSAN_900                                │
│     ├── Zvol Size: 680 GiB                                 │
│     └── iSCSI Target: hikvision-900                        │
└─────────────────────────────┬──────────────────────────────┘
                              │
                              │ iSCSI TCP 3260
                              │
                     FortiGate Firewall
                     Inter-VLAN Routing
                              │
                              ▼
                    CCTV Network
                    192.168.101.0/24

                    Hikvision DS-7732NI-I4(B)
                    IP: 192.168.101.139
                    Gateway: 192.168.101.1
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

ภาษาไทย:

ออกแบบและติดตั้งระบบ External Storage สำหรับ Hikvision NVR โดยใช้ TrueNAS Community Edition บน Microsoft Hyper-V เชื่อมต่อผ่าน iSCSI/IP SAN ข้าม VLAN ด้วย FortiGate Firewall สร้าง ZFS Pool และ iSCSI Target จำนวน 3 ชุด เพิ่มพื้นที่บันทึกรวม 1.62 TB พร้อมวิเคราะห์ NFS compatibility, RPC traffic, routing, firewall policy และ iSCSI session ด้วย tcpdump, rpcinfo และ FortiGate Debug Flow

English:

Designed and implemented an external storage solution for a Hikvision NVR using TrueNAS Community Edition hosted on Microsoft Hyper-V. Deployed three ZFS-backed iSCSI targets across VLANs through a FortiGate firewall, adding 1.62 TB of recording capacity. Troubleshot NFS compatibility, RPC traffic, routing, firewall policies, and iSCSI connectivity using tcpdump, rpcinfo, and FortiGate debug flow.

