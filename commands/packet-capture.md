# Packet Capture

## TrueNAS iSCSI Capture

```bash
sudo tcpdump -ni eth0 'host 192.168.101.139 and port 3260'
```

## TrueNAS Full NVR Capture

```bash
sudo tcpdump -ni eth0 host 192.168.101.139
```

## FortiGate iSCSI Sniffer

```text
diagnose sniffer packet any 'host 192.168.101.139 and host 192.168.1.24 and port 3260' 4 0 l
```

## FortiGate Full NVR-to-TrueNAS Sniffer

```text
diagnose sniffer packet any 'host 192.168.101.139 and host 192.168.1.24' 4 0 l
```

Expected iSCSI handshake:

```text
192.168.101.139.xxxxx -> 192.168.1.24.3260 SYN
192.168.1.24.3260 -> 192.168.101.139.xxxxx SYN,ACK
```

