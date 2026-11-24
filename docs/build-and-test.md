# Build and Test Procedure

## Provision the endpoint

1. Obtain an official Ubuntu cloud image and compare its SHA-256 checksum with the publisher's manifest.
2. Create a dedicated Proxmox VM with 2 cores, 2 GiB memory, and a 20 GiB system disk.
3. Attach a Cloud-Init drive and provision a dedicated administrative SSH key.
4. Attach the guest to a separate virtual bridge with no physical interface.
5. Configure its lab address, omit a default gateway, and apply the scoped firewall policy described in the architecture guide.
6. Verify SSH administration from the host and check the guest's operating system, resource allocation, and routes.

The deployed guest uses Ubuntu 24.04.5 LTS. The image's verified SHA-256 was `04e8f2461a16a44e78a370284471f29616cdd9baf1893df25807b71f985b6c94`. The publisher's `current` image changes over time, so a later download requires checking its current manifest. [Ubuntu cloud images](https://cloud-images.ubuntu.com/noble/current/) · [Proxmox Cloud-Init](https://github.com/proxmox/pve-docs/blob/master/qm-cloud-init.adoc)

## Connect and enroll the agent

Configure host forwarding for the event and enrollment ports, limited to the lab address and fixed manager destination. The [network templates](../config/network-policy.md) show the deployed bridge, firewall, and forwarding structure. Confirm connectivity from the endpoint before enrollment.

Transfer a compatible agent package through the host and install it on the test VM. This lab uses agent version 4.14.8 with a 4.14.8 manager. Configure the agent's manager and enrollment destinations to the host's lab-facing forwarding address.

Start the agent and confirm an active manager registration. Enrollment can precede the first event connection. In this run, the agent waited approximately 20 seconds before its initial manager connection.

## Configure real-time file monitoring

Create the directory before enabling its real-time watch:

```sh
sudo install -d /opt/security-lab/fim
```

Back up the test agent's existing configuration privately. Merge the settings in [fim-syscheck.xml](../config/fim-syscheck.xml) into its existing `<syscheck>` section. The lab-specific configuration replaces the test guest's directory list with the dedicated exercise directory.

The `realtime` setting enables live filesystem monitoring. `check_all` enables the file attribute checks, and `report_changes` supplies content-change information for the exercise's synthetic text. Use change reporting only on content suitable for collection into security evidence. [Official FIM configuration](https://documentation.wazuh.com/current/user-manual/capabilities/file-integrity/basic-settings.html)

Validate the test agent's configuration and apply it:

```sh
sudo /var/ossec/bin/wazuh-syscheckd -t
sudo /var/ossec/bin/wazuh-agentd -t
sudo systemctl enable --now wazuh-agent
```

When changing an already running test agent, restart that test agent to load its new FIM configuration. Check its log for completion of the initial scan and real-time monitoring startup before exercising the watch.

## Run the three operations

Use a new test filename for each run. The commands below use synthetic data and operate on the dedicated test guest.

Create:

```sh
printf 'project=wazuh-fim-lab\nphase=created\n' | sudo tee /opt/security-lab/fim/fim-demo.txt >/dev/null
sudo sha256sum /opt/security-lab/fim/fim-demo.txt
```

Find the added-file alert on the manager and in the indexer before proceeding.

Modify:

```sh
printf 'project=wazuh-fim-lab\nphase=modified\nchange=authorized-training-exercise\n' | sudo tee /opt/security-lab/fim/fim-demo.txt >/dev/null
sudo sha256sum /opt/security-lab/fim/fim-demo.txt
```

Compare the modification event's before/after hashes with the created and modified file hashes. Wait for its indexed document.

Delete:

```sh
sudo rm -- /opt/security-lab/fim/fim-demo.txt
```

Confirm the deletion event reaches both the manager and indexer and that the directory is empty.

The live run used a unique timestamped filename to distinguish its events from other activity. The published evidence normalizes host identifiers and omits the raw event IDs.

## Verify the outcome

Check each operation's rule ID, event type, real-time mode, timestamp, and file hashes. Match the manager event to its indexed counterpart using its event identifier privately. Record first index visibility and the polling method.

Finish by checking agent connectivity, the intended network boundary, existing application endpoints, and the watched directory's final state.
