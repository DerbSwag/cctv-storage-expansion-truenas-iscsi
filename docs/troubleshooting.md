# Troubleshooting

## NFS Findings

TrueNAS exported the NFS paths and the FortiGate allowed RPC traffic, but the NVR did not mount the NFS storage successfully.

Useful commands:

```bash
showmount -e localhost
rpcinfo -p localhost
sudo tcpdump -ni eth0 host 192.168.101.139
```

Observed behavior:

```text
NVR -> TrueNAS UDP 111    passed
NVR -> TrueNAS UDP 20048  passed
NVR requested NFSv3 over UDP
TrueNAS exposed NFS on TCP 2049
```

The production design moved to iSCSI/IP SAN.

## iSCSI Checks

Check that the service listens on TCP 3260:

```bash
sudo ss -lntp | grep 3260
```

Expected:

```text
LISTEN ... 0.0.0.0:3260
```

Check iSCSI global configuration:

```bash
sudo midclt call iscsi.global.config
```

Check targets and extents:

```bash
sudo midclt call iscsi.target.query
sudo midclt call iscsi.extent.query
```

## Packet Capture

TrueNAS:

```bash
sudo tcpdump -ni eth0 'host 192.168.101.139 and port 3260'
```

FortiGate:

```text
diagnose sniffer packet any 'host 192.168.101.139 and host 192.168.1.24 and port 3260' 4 0 l
```

Expected TCP handshake:

```text
192.168.101.139.xxxxx -> 192.168.1.24.3260 SYN
192.168.1.24.3260 -> 192.168.101.139.xxxxx SYN,ACK
```

## If a Hikvision IP SAN Disk Is Offline

1. Confirm TrueNAS VM is running.
2. Confirm pool is `ONLINE`.
3. Confirm iSCSI service is running.
4. Confirm TCP 3260 is listening.
5. Confirm target, extent, and associated target still exist.
6. In Hikvision, refresh HDD Management.
7. If needed, use Network HDD Search again.

Do not format a disk simply because it is temporarily offline.

