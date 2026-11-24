# Your cybersecurity lab, explained simply

Reviewed November 24, 2026. This report covers the new Wazuh file-monitoring project and its security review. It does not certify every device in the homelab.

## What we built

We created a separate Ubuntu practice computer inside Proxmox. A virtual machine is a computer made in software that uses some of your server's resources. This one has 2 CPU cores, 2 GiB of memory, and a 20 GiB virtual disk. The disk uses physical storage as data is written. Automatic startup after a host reboot is off.

We installed a Wazuh agent: a small program that reports security information to your existing Wazuh server. It watches `/opt/security-lab/fim` for file changes. This is called file integrity monitoring, or FIM. Think of it as an alarm for files being added, changed, or removed. It reports changes; it does not decide that every change is malicious or automatically block it.

The practice machine uses a separate virtual network with no physical uplink and no default internet route. Your existing Wazuh server receives its reports through a restricted connection on the Proxmox host.

## What we tested

We created a harmless text file, changed it, and deleted it. Wazuh recorded all three actions and made the alerts searchable. We also checked each file's digital fingerprint, called a checksum, to confirm that the recorded changes matched the test.

The first test appeared in searchable results in about 10–16 seconds. After security updates, all three tests passed again in about 10–11 seconds. These timings describe the observed tests, not a speed guarantee.

The original 13 media containers kept their existing identities. Nine application endpoint checks passed after following login redirects, and the four main Wazuh services were active. This checks service availability; it is not a full playback or application audit.

## What we secured and updated

- Refreshed Ubuntu's package lists, upgraded 67 packages, and installed 6 new packages, including the newer kernel.
- Restarted only the lab VM and confirmed it is running kernel `6.8.0-151-generic`. Removed the old `6.8.0-147` kernel packages after the new boot worked.
- Disabled root SSH login and password login. Administration requires the lab administrator account and its dedicated SSH key.
- Closed the temporary enrollment connection after Wazuh registration. Normal operation keeps only the event connection open from the lab to its forwarding address.
- Verified that the lab could send Wazuh events while connections to Proxmox administration, a media service, and a public HTTPS destination failed.
- Removed the temporary Ubuntu-only update proxy after patching. The practice machine still has no general internet route.
- Checked the agent configuration and package database; no failed systemd units or unfinished package operations were reported.

## What the security findings mean

Wazuh currently reports **440 package/CVE records** for this lab after the update review. These are scanner findings, not a count of successful attacks. The refreshed records all concern the current kernel, including 2 Critical and 111 High ratings. The installed kernel matches the available Ubuntu repository candidate. Remaining findings require vendor fixes and applicability review. The lab cannot be described as vulnerability-free.

One SQLite update is deferred by Ubuntu's phased rollout. The installed version comes from the security repository; the newer candidate is in the updates repository. We left the publisher's rollout policy in place.

No responsible review can promise “no vulnerabilities.” These checks reduce specific risks and document what worked. They do not replace future patching, vulnerability review, or a full audit of the host, router, Wazuh server, and other applications.

## How to use the project

1. In Proxmox, find the VM named `fim-lab-01` and make sure it is running. It does not start automatically after a host reboot.
2. Open the Proxmox host's shell and run `ssh fim-lab`. This uses the private key already stored on that host. Keep that key private.
3. Create a harmless practice file in `/opt/security-lab/fim`, change its text, then delete it. Use invented test text because change reports can include file contents.
4. In Wazuh, select the `fim-lab-01` agent and look for file-change alerts: rule 554 means added, 550 means changed, and 553 means deleted.
5. Investigate whether each change was expected. An alert is evidence to review, not proof of an attack.

## What still needs attention

Backups remain limited by available storage, as previously agreed. This practice VM is outside the current backup job. The GitHub instructions help rebuild it, but documentation is not a backup of the live system. Extra storage and a tested restore remain important next steps when you are ready.

The VM needs periodic controlled updates because it has no permanent internet connection. The physical Proxmox host is still shared with the existing services, so it remains a common point of failure.

## Where the documentation lives

- Project and evidence: https://github.com/J0N3B0I/Wazuh-FIM-Lab
- Wazuh setup: https://github.com/J0N3B0I/Wazuh-Homelab
- Main homelab documentation: https://github.com/J0N3B0I/Proxmox-homelab

Public files omit private addresses, credentials, and SSH keys.
