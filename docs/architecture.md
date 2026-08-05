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
| NVR | Hikvision Hikvision NVR |
| NVR firmware | production firmware release |
| Server network | 10.10.10.0/24 |
| CCTV network | 10.20.30.0/24 |
| TrueNAS IP | 10.10.10.20 |
| NVR IP | 10.20.30.40 |
| iSCSI port | TCP 3260 |
| Virtual storage | Dynamic VHDX |
| iSCSI extent type | Device Zvol |
| Compatibility profile | Legacy OS |

## Logical Flow

```text
Hikvision NVR
  IP: 10.20.30.40
  Storage Type: IP SAN
      â”‚
      â”‚ TCP 3260
      â–¼
FortiGate
  CCTV_VLAN: CCTV network
  SERVER_VLAN: server network
  NAT: disabled for local traffic
      â”‚
      â–¼
TrueNAS VM
  IP: 10.10.10.20
  iSCSI Portal: 0.0.0.0:3260
      â”‚
      â”œâ”€â”€ target-800 -> CCTV/NVR_IPSAN_800 -> 600 GiB
      â”œâ”€â”€ target-460 -> NVR_CCTV/NVR_IPSAN_460 -> 340 GiB
      â””â”€â”€ target-900 -> NVR_CCTV_3/NVR_IPSAN_900 -> 680 GiB
```

## FortiGate Routing

The FortiGate routes both networks directly:

```text
10.10.10.0/24   directly connected via SERVER_VLAN
10.20.30.0/24 directly connected via CCTV_VLAN
```

The working local policy was:

```text
Policy ID: example-local-policy
Name: LOCAL_LINK
Source Interface: CCTV_VLAN
Destination Interface: SERVER_VLAN
Action: Accept
Service: ALL
NAT: Disabled
```

For production hardening, replace `ALL` with a dedicated policy that allows only:

```text
Source:      10.20.30.40
Destination: 10.10.10.20
Service:     TCP 3260
NAT:         Disabled
```

