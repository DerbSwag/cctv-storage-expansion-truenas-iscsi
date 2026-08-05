# Lessons Learned

## NFS Was Not the Best Protocol for This NVR and TrueNAS Version

The NVR successfully reached RPCBind and mountd, but the packet captures showed it requesting NFSv3 over UDP. The deployed TrueNAS system exposed NFS on TCP 2049. This mismatch made NFS unsuitable for the production design.

## iSCSI Was the Correct Hikvision Integration Path

Hikvision exposes iSCSI under the `IP SAN` storage type. Once the target was created in TrueNAS and discovered through Hikvision Search, the NVR attached the LUN and formatted it successfully.

## Use Search Instead of Typing Target Names Manually

The NVR worked reliably after using the built-in Search function to discover full IQNs:

```text
iqn.2005-10.example.storage:target-800
iqn.2005-10.example.storage:target-460
iqn.2005-10.example.storage:target-900
```

## Keep ZFS Pool Usage Below About 80%

Leaving free capacity helps ZFS handle copy-on-write, metadata, fragmentation, and continuous CCTV writes.

## Thick Zvols Make TrueNAS Usage Look High

TrueNAS reserves thick zvol capacity immediately. Hikvision free space and TrueNAS used space are different views of the same storage.

## Do Not Trust VM Storage as Backup

The design expands recording capacity. It does not provide redundancy, offsite backup, or retention protection.

