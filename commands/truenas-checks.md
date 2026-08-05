# TrueNAS Checks

## Pool Health

```bash
zpool status
zpool list
zfs list
lsblk -o NAME,SIZE,TYPE,MODEL
```

Expected pools:

```text
CCTV
NVR_CCTV
NVR_CCTV_3
```

Expected zvols:

```text
CCTV/NVR_IPSAN_800
NVR_CCTV/NVR_IPSAN_460
NVR_CCTV_3/NVR_IPSAN_900
```

## iSCSI Service

```bash
sudo ss -lntp | grep 3260
sudo midclt call iscsi.global.config
sudo midclt call iscsi.target.query
sudo midclt call iscsi.extent.query
```

Expected listener:

```text
0.0.0.0:3260
```

## NFS Investigation Commands

These were used during the proof of concept:

```bash
showmount -e localhost
rpcinfo -p localhost
sudo tcpdump -ni eth0 host 10.20.30.40
```

