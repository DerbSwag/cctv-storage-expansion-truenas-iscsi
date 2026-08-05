# Packet Capture

## TrueNAS iSCSI Capture

```bash
sudo tcpdump -ni eth0 'host 10.20.30.40 and port 3260'
```

## TrueNAS Full NVR Capture

```bash
sudo tcpdump -ni eth0 host 10.20.30.40
```

## FortiGate iSCSI Sniffer

```text
diagnose sniffer packet any 'host 10.20.30.40 and host 10.10.10.20 and port 3260' 4 0 l
```

## FortiGate Full NVR-to-TrueNAS Sniffer

```text
diagnose sniffer packet any 'host 10.20.30.40 and host 10.10.10.20' 4 0 l
```

Expected iSCSI handshake:

```text
10.20.30.40.xxxxx -> 10.10.10.20.3260 SYN
10.10.10.20.3260 -> 10.20.30.40.xxxxx SYN,ACK
```

