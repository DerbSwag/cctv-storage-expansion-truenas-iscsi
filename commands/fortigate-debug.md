# FortiGate Debug Commands

## Routing

```text
get router info routing-table details 10.10.10.20
get router info routing-table details 10.20.30.40
```

Expected:

```text
10.10.10.0/24   directly connected, SERVER_VLAN
10.20.30.0/24 directly connected, CCTV_VLAN
```

## Debug Flow

```text
diagnose debug reset
diagnose debug flow filter clear
diagnose debug flow filter saddr 10.20.30.40
diagnose debug flow filter daddr 10.10.10.20
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
diagnose sys session filter src 10.20.30.40
diagnose sys session filter dst 10.10.10.20
diagnose sys session list
diagnose sys session filter clear
```

Look for:

```text
policy_id=<local-policy-id>
```

For the hardened policy, verify that it uses TCP 3260 and NAT is disabled.

