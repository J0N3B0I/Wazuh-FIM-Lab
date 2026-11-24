# Operations and Cleanup

## Current lab state

The test VM remains available with 2 cores, 2 GiB RAM, and a 20 GiB virtual disk. Automatic startup when the Proxmox host boots is disabled. The agent is enabled inside the guest and starts when the VM is started.

The private administrative key is stored on the hypervisor. The test VM uses key-based SSH administration through its dedicated bridge. Root login and password authentication are disabled; only the lab administrator account is allowed over SSH. Its lack of a default route prevents ordinary internet package retrieval. Security updates require a controlled maintenance path; the temporary Ubuntu-only proxy used for the security review has been stopped and removed.

The host's lab bridge configuration, forwarding socket units, and firewall rules persist across a host reboot. The event socket is enabled; the enrollment socket is disabled, and the VM can be started manually for the next exercise.

## Routine checks

Inside the test VM:

```sh
sudo systemctl is-active wazuh-agent
sudo tail -n 30 /var/ossec/logs/ossec.log
ls -la /opt/security-lab/fim
```

On the Wazuh manager:

```sh
sudo /var/ossec/bin/agent_control -l
sudo systemctl is-active wazuh-manager wazuh-indexer wazuh-dashboard filebeat
```

On the hypervisor:

```sh
systemctl is-active fim-lab-events.socket fim-lab-events.service
systemctl is-enabled fim-lab-enrollment.socket # expected: disabled
pve-firewall status
```

Use the actual lab VM identifier with `qm status` and `qm start` when needed. Keep private addressing and key paths in the local administration record.

## Diagnose a missing event

1. Confirm the agent is active at both the endpoint and manager.
2. Confirm the watch directory existed when the agent loaded the real-time configuration.
3. Check configuration validation and the real-time startup message.
4. Confirm the event is present in manager JSON alerts.
5. Check Filebeat and search the indexer with the intended time range, agent, and unique filename.
6. Confirm the manager and indexed event identifiers agree.

The enrolled agent initially used a short connection delay after receiving its key. Allow enrollment and the first manager connection to complete before treating startup timing as a fault.

## Re-run the exercise

Choose a new file name and follow the build-and-test guide. Confirm each event is indexed before advancing to the next operation. Keep test content synthetic because change reporting can collect its text.

## Backup and resource scope

This VM is outside the current guest backup job's explicit list. Backup capacity work is deferred by the owner. The repository records a rebuild procedure and configuration, while recovery of the live guest requires its own backup strategy.

The Proxmox thin pool was approximately 54% allocated after provisioning. The 20 GiB virtual disk is thin provisioned. Monitor actual pool usage as more labs are added.

## Teardown plan

When the lab is retired:

1. Record any final evidence and remove the lab's agent enrollment through the supported Wazuh workflow.
2. Shut down and remove the dedicated lab VM and its disks after confirming its identifier.
3. Disable and remove the two lab forwarding socket/service pairs.
4. Remove only the lab-specific host firewall rule and bridge configuration after checking their use by any later projects.
5. Remove the lab's private SSH identity and access alias when they are no longer required.
6. Re-check the original media endpoints, existing agents, and firewall state.

These steps are an operations plan. The live lab remains deployed for subsequent exercises.
