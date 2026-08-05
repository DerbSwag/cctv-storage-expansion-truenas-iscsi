# Implementation Guide

## Phase 1: Infrastructure Preparation

1. Create a TrueNAS VM in Microsoft Hyper-V.
2. Use Generation 2.
3. Disable Secure Boot.
4. Allocate 2 vCPU and 8 GB RAM.
5. Disable Dynamic Memory.
6. Disable Hyper-V checkpoints.
7. Attach the VM to an External Virtual Switch.
8. Set the TrueNAS static IP to `192.168.1.24/24`.
9. Set the default gateway to `192.168.1.1`.
10. Attach Dynamic VHDX data disks for the storage pools.

## Phase 2: NFS Proof of Concept

NFS was tested first with these exports:

```text
/mnt/CCTV/NVR_Record_800
/mnt/NVR_CCTV/NVR_Record_460
```

The NVR was authorized as:

```text
192.168.101.139
```

Verification commands:

```bash
ping -c 4 192.168.101.139
showmount -e localhost
rpcinfo -p localhost
```

Packet capture showed:

```text
UDP 111   RPCBind passed
UDP 20048 mountd passed
NVR requested NFS program 100003 version 3 over UDP
TrueNAS exposed NFSv3 on TCP 2049
```

Because the protocol behavior did not match, NFS was not selected as the production storage protocol.

## Phase 3: iSCSI Implementation

For each pool:

1. Create a thick zvol.
2. Set block size to 16 KiB.
3. Set sync to Standard.
4. Keep compression as LZ4.
5. Keep deduplication off.
6. Create an iSCSI Device Extent.
7. Select `Legacy OS` as the sharing platform.
8. Create an iSCSI Target.
9. Use the shared iSCSI portal on TCP 3260.
10. Restrict Authorized Network to `192.168.101.139/32`.
11. Use Authentication Method `None`.
12. Associate the Target with the Extent.
13. Use LUN ID 0 per target.

## Phase 4: Hikvision Integration

On the Hikvision NVR:

1. Open `Storage -> Storage Management -> Network HDD`.
2. Select Type `IP SAN`.
3. Set Server Address to `192.168.1.24`.
4. Use Search to discover the iSCSI targets.
5. Select the IQN target.
6. Save the Network HDD entry.
7. Open `HDD Management`.
8. Format the new IP SAN disk.
9. Confirm status is `Normal`.
10. Confirm property is `R/W`.

## Phase 5: Operational Validation

Validate from TrueNAS:

```bash
zpool status
sudo ss -lntp | grep 3260
```

Validate from Hikvision:

```text
Disk 17: 680 GB, IP SAN, Normal, R/W
Disk 18: 600 GB, IP SAN, Normal, R/W
Disk 19: 340 GB, IP SAN, Normal, R/W
```

