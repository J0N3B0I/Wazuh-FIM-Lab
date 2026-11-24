# Network Configuration Templates

These templates describe the configuration deployed for this exercise. Replace the address and interface placeholders with the intended local values. The actual host addresses and administrative key paths are retained privately.

## Isolated bridge

A dedicated file under the host's network-interface include directory defines the bridge:

```text
auto LAB_BRIDGE
iface LAB_BRIDGE inet static
    address LAB_GATEWAY_IP/24
    bridge-ports none
    bridge-stp off
    bridge-fd 0
```

The guest uses a static address in the same subnet and has no default gateway. Its virtual NIC has Proxmox firewall filtering enabled. The bridge is brought up individually, without restarting the host's existing network configuration.

## Guest firewall

```text
[OPTIONS]
enable: 1
policy_in: DROP
policy_out: DROP
dhcp: 0
ndp: 0

[RULES]
IN ACCEPT -source LAB_GATEWAY_IP -p tcp -dport 22
OUT ACCEPT -dest LAB_GATEWAY_IP -p tcp -dport 1514,1515
```

The host firewall receives one additional rule scoped to the test guest and the forwarding address:

```text
IN ACCEPT -source LAB_GUEST_IP -dest LAB_GATEWAY_IP -p tcp -dport 1514,1515
```

The rest of the host policy is preserved. Validate the resulting Proxmox firewall configuration before starting the exercise.

## Event forwarding socket

The event socket unit listens on the host's lab-facing address:

```ini
[Unit]
Description=FIM lab event forwarding socket
After=network.target

[Socket]
ListenStream=LAB_GATEWAY_IP:1514
FreeBind=true

[Install]
WantedBy=sockets.target
```

The matching service unit forwards the accepted stream to the manager:

```ini
[Unit]
Description=Forward FIM lab events to Wazuh

[Service]
ExecStart=/usr/lib/systemd/systemd-socket-proxyd WAZUH_MANAGER_IP:1514
DynamicUser=yes
NoNewPrivileges=yes
PrivateTmp=yes
ProtectSystem=strict
ProtectHome=yes
MemoryMax=64M
```

The enrollment socket and service use the same structure with port 1515. Matching service/socket unit names let systemd activate the correct proxy when a connection arrives. The deployed unit prefixes are `fim-lab-events` and `fim-lab-enrollment`.

## Verification

The recorded checks cover the two allowed forwarding ports and three denied destinations. Review [network-checks.json](../evidence/network-checks.json) for the observed results. The TCP forwarding service's accepted socket alone does not establish agent health. The active registration and indexed test events establish the working manager connection in this lab.
