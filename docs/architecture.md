# Architecture

## Components

| Component | Technology |
|---|---|
| Hypervisor | Microsoft Hyper-V |
| Storage operating system | TrueNAS Community Edition |
| Storage filesystem | ZFS |
| Block storage protocol | iSCSI |
| Hikvision storage type | IP SAN |
| Firewall and router | FortiGate |
| NVR | Hikvision DS-7732NI-I4(B) |
| NVR firmware | V4.61.030 build 240123 |
| Server network | 192.168.1.0/24 |
| CCTV network | 192.168.101.0/24 |
| TrueNAS IP | 192.168.1.24 |
| NVR IP | 192.168.101.139 |
| iSCSI port | TCP 3260 |
| Virtual storage | Dynamic VHDX |
| iSCSI extent type | Device Zvol |
| Compatibility profile | Legacy OS |

## Logical Flow

```text
Hikvision NVR
  IP: 192.168.101.139
  Storage Type: IP SAN
      │
      │ TCP 3260
      ▼
FortiGate
  internal3: CCTV network
  internal1: server network
  NAT: disabled for local traffic
      │
      ▼
TrueNAS VM
  IP: 192.168.1.24
  iSCSI Portal: 0.0.0.0:3260
      │
      ├── hikvision-800 -> CCTV/NVR_IPSAN_800 -> 600 GiB
      ├── hikvision-460 -> NVR_CCTV/NVR_IPSAN_460 -> 340 GiB
      └── hikvision-900 -> NVR_CCTV_3/NVR_IPSAN_900 -> 680 GiB
```

## FortiGate Routing

The FortiGate routes both networks directly:

```text
192.168.1.0/24   directly connected via internal1
192.168.101.0/24 directly connected via internal3
```

The working local policy was:

```text
Policy ID: 15
Name: LOCAL_LINK
Source Interface: internal3
Destination Interface: internal1
Action: Accept
Service: ALL
NAT: Disabled
```

For production hardening, replace `ALL` with a dedicated policy that allows only:

```text
Source:      192.168.101.139
Destination: 192.168.1.24
Service:     TCP 3260
NAT:         Disabled
```

