# Your Wazuh project: the complete plain-English guide
Recorded work: November 24, 2026.

## Your Wazuh project, from start to finish

A plain-English guide to what was built, why each step mattered, and what the results mean.

### What this document covers

This guide brings together the new file-monitoring lab, its security updates, and the follow-up review of the remaining kernel findings. It describes verified work recorded on November 24, 2026. It is not a new live audit or a history of every earlier homelab change.

**The goal**

Build a separate place to practice cybersecurity, prove that Wazuh notices file changes, and reduce the lab’s exposure while preserving existing services.

### The pieces in everyday language

| Name | What it does |
| --- | --- |
| Proxmox | Runs the virtual computers on your physical server. |
| Ubuntu practice VM | A separate software computer for harmless exercises. |
| Wazuh agent | The small reporter installed on the practice VM. |
| Wazuh manager | Receives reports and creates alerts. |
| Indexer and dashboard | Store searchable alerts and let you investigate them. |

### How to read the guide

Page 2 explains the build. Page 3 explains the tests. Page 4 covers updates and access controls. Page 5 explains the remaining findings. Page 6 shows how to practice and what maintenance still needs attention.

The lab is operating with the protections that were tested. Open kernel findings and the backup gap remain. Working properly does not mean completely risk-free.


## 1. Build a separate practice computer

The existing Wazuh server was reused; the practice machine was created separately.

### Step 1 — Create the virtual machine

We created fim-lab-01 using an official Ubuntu 24.04.5 LTS image and checked its published SHA-256 checksum. A checksum is a digital fingerprint: matching it helps confirm that the downloaded image matches the publisher’s file.

| Resource | Assigned amount |
| --- | --- |
| CPU | 2 virtual cores |
| Memory | 2 GiB |
| Virtual disk | 20 GiB; physical space is used as data is written |
| Automatic startup | Off after a Proxmox host reboot |

### Step 2 — Give it a restricted network

A separate virtual bridge connects the lab to a controlled address on the Proxmox host. The bridge has no physical uplink, and the guest has no default internet route. Firewall rules allow host-to-lab administration and a narrow path for Wazuh reports.

### Step 3 — Connect the Wazuh reporter

We installed Wazuh agent 4.14.8, matching the existing manager. Registration briefly used an enrollment connection on port 1515. Reports use port 1514. The host forwards that stream to the existing Wazuh server. After setup, the enrollment connection was closed.

### Step 4 — Choose what to watch

Real-time file integrity monitoring was enabled for /opt/security-lab/fim. Wazuh watches for files being added, changed, or deleted there. Built-in Wazuh rules handle these alerts; no custom manager rules were required.

**Why a separate folder?**

It gives you a predictable place for harmless exercises. Change reports can include file text, so practice with invented content and keep passwords and private documents out of this folder.


## 2. Prove the whole monitoring path works

We checked the reports at both the Wazuh manager and the searchable index.

**The path we verified**

Test file changes > agent reports the change > manager records an alert > Filebeat forwards it > indexer stores a searchable result.

| Test | Expected alert | Result |
| --- | --- | --- |
| Create a text file | Rule 554: file added | Passed |
| Change its contents | Rule 550: checksum changed | Passed |
| Delete the file | Rule 553: file deleted | Passed |

### What made this more than a simple screenshot

We matched event identifiers between the manager and indexer. We also compared SHA-256 file fingerprints with the known test contents, including the before-and-after values for the modification. Each operation reported real-time monitoring mode. The final test file was removed.

### We repeated the tests after making changes

The initial run appeared in searchable results in roughly 10–16 seconds. After the package updates, the three tests passed again in about 10–11 seconds. After the later kernel restrictions and restart, they passed in 13.40, 12.45, and 13.35 seconds. These are measured observations from these runs, not a performance guarantee.

### We also checked the existing services

All 13 original media containers kept their existing identities. Nine application endpoint checks ended at HTTP 200, meaning a successful web response, after following redirects. The Wazuh manager, indexer, dashboard, and Filebeat were active.

These checks support service availability and successful event delivery. They do not prove full movie playback, every application feature, or the security of every device. An alert records a change; it does not automatically mean an attack happened.


## 3. Update the lab and tighten access

These changes were limited to the practice VM and its dedicated forwarding rules.

### Refresh and install software updates

The original package lists were old. We refreshed them and upgraded 67 packages, installing 6 new packages including a newer kernel. A temporary Ubuntu-only update proxy provided a controlled maintenance connection without adding a default route. The proxy was stopped and its script removed afterward.

Only the practice VM was restarted. It booted kernel 6.8.0-151-generic, package version 6.8.0-151.151. The old 6.8.0-147 image, modules, and headers were removed after the new boot worked. One SQLite update remained deferred by Ubuntu’s phased rollout; the installed version came from the security repository.

### Restrict logins and close setup access

SSH is a remote command-line connection. Root SSH login, password login, and keyboard-interactive login were disabled. SSH administration is restricted to the lab administrator using a dedicated private key. The enrollment listener was stopped and disabled, and port 1515 was removed from the lab’s allowed firewall rules.

| Connection tested from the lab | Observed result |
| --- | --- |
| Wazuh event connection | Allowed |
| Wazuh enrollment connection | Blocked after registration |
| Proxmox administration | Blocked |
| Selected media service | Unreachable |
| Public HTTPS destination | Unreachable |

### Add protections for unused kernel features

We blocked normal loading of the unused Xen network backend, disabled ordinary-user namespace creation, and restricted kernel pointer visibility. The already-enabled restriction on unprivileged BPF remained in place. In plain terms, these limit ways ordinary users can reach certain kernel features or see kernel details.

The guest’s startup files were regenerated and it was restarted again. The restrictions persisted, namespace creation was denied, and the file-change tests passed. Rootless containers or some sandbox software would need review before being added. These restrictions reduce exposure; they do not patch all kernel flaws.


## 4. Understand the remaining findings

The kernel is the central part of Linux that manages hardware, memory, and other low-level work.

Wazuh compares installed software with a database of known weaknesses, called CVEs. A package/CVE record is a warning to assess. It is not a report that an attacker has successfully used that weakness.

**What changed**

The lab’s reported records fell from 3,335 before patching to 440 after the inventory refreshed. The remaining records all concern the current kernel. They include 2 Critical and 111 High severity ratings.

### Every remaining record was checked against Ubuntu

| Ubuntu status | Count | Plain-English meaning |
| --- | --- | --- |
| Needed | 372 | A fix is still marked as needed. |
| Pending | 7 | A fix is pending; not marked released. |
| Deferred | 27 | Vendor work has been deferred. |
| Needs triage | 21 | Vendor analysis is still needed. |
| Ignored | 13 | Read the stated reason; this does not mean safe. |

All seven pending records name version 6.8.0-152.152 in Ubuntu’s notes. That is a supported-repository update to watch for; it was not installed here. None of the captured records was marked released. Two ignored records have older version notes that need clarification.

### Why the two Critical findings still matter

CVE-2021-3773 concerns OpenVPN-related networking. OpenVPN was not installed; Ubuntu reports no upstream fix. CVE-2026-99287 concerns the Xen network backend. This VM uses KVM, the Xen module was not loaded, and normal loading is now blocked. Those facts reduce the described exposure, but do not erase the package findings.

The review checked all 440 vendor statuses and selected aspects of actual guest usage. It did not prove individual exploitability or non-applicability for every CVE. Some flaws affect general kernel paths that the VM may use. No findings were hidden or deleted, and no vulnerability-free claim was made.


## 5. Use the lab and keep the record useful

You can use the project to practice recognizing, investigating, and explaining file changes.

### A simple practice exercise

1. In Proxmox, start the VM named fim-lab-01 if it is stopped. Automatic startup after a host reboot is disabled.

2. Open the Proxmox host shell and run ssh fim-lab. That host already has the dedicated SSH key. This alias is not automatically available on your phone or another computer.

3. In /opt/security-lab/fim, create a harmless text file, change its contents, then delete it. Use a new filename each time so the exercise is easy to identify.

4. In Wazuh, filter for the fim-lab-01 agent. Look for rules 554, 550, and 553. Compare the filename, time, and operation with what you intentionally did.

### How to analyze the spreadsheet

Open evidence/kernel-triage.csv from the project repository. Start with the Critical and High records. Read the Ubuntu status and version note, then follow the CVE link. The review note describes relevant guest checks. A review group organizes findings; it is not a declaration that the flaw cannot affect this VM.

### What still needs maintenance

As supported fixes become available, use a controlled maintenance connection to refresh Ubuntu metadata and install them. Restart only the lab when needed, verify its running kernel and Wazuh tests, then let the inventory update. No recurring advisory monitor, unattended updater, or scheduled reboot was installed.

**The remaining recovery gap**

Extra backup storage is still pending, as agreed. This VM is outside the current backup job. GitHub documentation helps rebuild the lab, but it is not a backup of the live machine. The shared physical host remains a common point of failure.

Project, detailed evidence, and rollback instructions: https://github.com/J0N3B0I/Wazuh-FIM-Lab
Wazuh setup: https://github.com/J0N3B0I/Wazuh-Homelab
Main homelab record: https://github.com/J0N3B0I/Proxmox-homelab

Public documentation omits private addresses, credentials, and SSH keys. This guide records the completed work and its limits; it does not certify the entire homelab as secure.

