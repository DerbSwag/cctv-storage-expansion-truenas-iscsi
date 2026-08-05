# Runbook: iSCSI Target Offline

Use this when one or more Hikvision IP SAN disks show `Offline`.

## Checks

1. Confirm TrueNAS Web UI is reachable.
2. Confirm the related pool is `ONLINE`.
3. Confirm iSCSI service is running.
4. Confirm the target exists:

```bash
sudo midclt call iscsi.target.query
```

5. Confirm the extent exists:

```bash
sudo midclt call iscsi.extent.query
```

6. Confirm TCP 3260 traffic:

```bash
sudo tcpdump -ni eth0 'host 10.20.30.40 and port 3260'
```

7. On Hikvision, use Network HDD Search and select the IQN again if needed.

## Expected Targets

```text
iqn.2005-10.example.storage:target-800
iqn.2005-10.example.storage:target-460
iqn.2005-10.example.storage:target-900
```

