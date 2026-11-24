# Project Summary — Real-time File Integrity Monitoring

## Objective

Demonstrate a complete detection workflow using a dedicated Linux endpoint and the existing Wazuh deployment. The acceptance criteria were three verified file-change events, matching manager/indexer evidence, verified hashes, restricted connectivity, and healthy existing services after the exercise.

## Implementation

Provisioned an Ubuntu 24.04.5 LTS virtual machine in Proxmox with 2 CPU cores, 2 GiB RAM, and a 20 GiB virtual disk. The image checksum matched the publisher's SHA-256 manifest.

A dedicated virtual bridge provides the lab network. The guest has no default route, and its firewall limits connectivity to the required Wazuh forwarding ports and host-originated administration. Host socket-activated forwarding carries event and enrollment traffic to the existing manager. Private SSH credentials support administration from the hypervisor.

Installed a Wazuh agent matching the manager's 4.14.8 version. Configured `/opt/security-lab/fim` for real-time checks, new-file alerts, file checksums, and change reporting. The existing agent configuration was preserved privately before the lab-specific FIM changes.

## Experiment

Created a synthetic text file, waited for its manager alert and indexed document, changed the contents, waited again, and deleted the file. Each operation was verified before advancing to the next operation.

| Event | Built-in rule | Observed index visibility |
| --- | --- | --- |
| Added | 554 | 9.76 seconds |
| Modified | 550 | 10.80 seconds |
| Deleted | 553 | 16.46 seconds |

The manager's event identifiers matched the indexed documents. All events reported real-time mode. The creation and modification hashes matched the file contents, and the modification event linked the original hash to the new hash.

## Validation

Positive connection checks confirmed the two Wazuh forwarding ports. Negative checks confirmed that the guest could not reach the Proxmox administration port, a selected media endpoint, or a public HTTPS destination. These were targeted TCP checks on the lab's actual network configuration.

All existing media container IDs remained the same. Nine application endpoints responded with HTTP 200, four Wazuh central services were active, and the newly enrolled agent reported active status. The final test file was deleted and its watched directory was empty.

## Interpretation

The exercise demonstrates endpoint collection, built-in rule matching, alert forwarding, and indexed evidence using controlled file changes. A higher-severity integrity alert reflects the matching rule and affected attributes. The exercise's file changes were authorized training actions.

The timing measurements include polling and query overhead and represent a single run. Further experiments can assess repeatability, authorization context, and detection behavior under load.

## Deliverables and follow-up

The project includes an architecture diagram, configuration excerpt, reproduction procedure, investigation notes, operations guidance, and sanitized machine-readable evidence.

The lab VM remains available for future exercises. Its automatic startup is disabled, and its backup coverage follows the homelab's current storage limitation. Future work can add attribution of file changes, a controlled authentication investigation, or a hardening assessment.
