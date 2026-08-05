# FortiGate Debug Commands

## Routing

```text
get router info routing-table details 192.168.1.24
get router info routing-table details 192.168.101.139
```

Expected:

```text
192.168.1.0/24   directly connected, internal1
192.168.101.0/24 directly connected, internal3
```

## Debug Flow

```text
diagnose debug reset
diagnose debug flow filter clear
diagnose debug flow filter saddr 192.168.101.139
diagnose debug flow filter daddr 192.168.1.24
diagnose debug flow show function-name enable
diagnose debug flow trace start 200
diagnose debug enable
```

Stop debug:

```text
diagnose debug disable
diagnose debug flow trace stop
diagnose debug flow filter clear
```

## Check Sessions

```text
diagnose sys session filter clear
diagnose sys session filter src 192.168.101.139
diagnose sys session filter dst 192.168.1.24
diagnose sys session list
diagnose sys session filter clear
```

Look for:

```text
policy_id=15
```

For the hardened policy, verify that it uses TCP 3260 and NAT is disabled.

