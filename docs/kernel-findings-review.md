# Remaining kernel findings: review and protections

Reviewed November 24, 2026. Scope: the dedicated Ubuntu FIM practice guest, not the Proxmox host kernel or existing Wazuh/media guests.

## What was reviewed

Reconciled all 440 remaining package/CVE records against the official Ubuntu CVE JSON service, selecting the `linux` source package and `noble` release. The installed kernel package is `6.8.0-151.151`. The evidence retains a vendor reference and result per CVE.

This is a complete vendor-status lookup of the captured inventory, combined with targeted checks of actual guest usage. It is **not** a demonstration of exploitability or a complete individual applicability analysis for every CVE. Text-based review groups help organize the work; they are not automatic dismissals.

| Ubuntu status | Records |
| --- | ---: |
| deferred | 27 |
| ignored | 13 |
| needed | 372 |
| needs-triage | 21 |
| pending | 7 |

Ubuntu status meanings matter: `needed` indicates an outstanding fix; `needs-triage` indicates vendor analysis is pending; `pending` and `deferred` do not establish that an installed package is patched; `ignored` requires reading the reason and does not automatically mean safe. A scanner severity can differ from Ubuntu's priority.

All seven `pending` records name `6.8.0-152.152` in the vendor version note. That is a future package to check for in the supported repositories; it is not installed here and does not resolve the other findings. Two `ignored` records carry a `6.8.0-44.44` version note, which predates the installed package; these ambiguous status/version combinations need clarification rather than automatic dismissal. No finding in this captured set was marked `released` by the vendor.

## Protections applied to this VM

1. Added `/etc/modprobe.d/fim-lab-unused-xen.conf` with `blacklist xen_netback` and `install xen_netback /bin/false`. The guest uses KVM and did not have this module loaded. This blocks normal modprobe loading of the unused backend; it does not remove vulnerable kernel code or prevent an administrator from bypassing the policy.
2. Added `/etc/sysctl.d/99-fim-lab-kernel-hardening.conf` with `kernel.unprivileged_userns_clone=0`. This restricts ordinary users from creating user namespaces and reaching kernel features through them. Rootless containers and some desktop sandboxes would need a review before being added to this dedicated lab.
3. Set `kernel.kptr_restrict=2` to restrict kernel pointer disclosure. Existing `kernel.unprivileged_bpf_disabled=2` remained enabled.
4. Regenerated this guest's initramfs and restarted **only the practice VM**. Confirmed settings persisted, dry-run modprobe selected `/bin/false`, and an ordinary-user namespace creation attempt was denied.
5. Retained the isolated bridge, default-deny firewall, no default internet route, closed enrollment listener, and key-only SSH administration.

These controls reduce exposure. They do not patch all remaining CVEs or make their inventory counts disappear.

## Actual guest usage checked

The guest runs x86_64 under KVM. No Xen environment, OpenVPN package, Bluetooth/Wi-Fi class devices, or loaded Xen backend was observed. The selected network/filesystem modules were absent, and no NFS, SMB, Ceph, F2FS, NTFS, Btrfs, JFS, or virtiofs mount was present. Some unused filesystem modules, including Btrfs, were loaded; lack of a mount is not proof that vulnerable code cannot be reached. General networking, ext4, virtual hardware, and other kernel paths still require normal vendor fixes and assessment.

The two Critical records remain documented:

- [CVE-2021-3773](https://ubuntu.com/security/CVE-2021-3773): vendor reports no upstream fix; OpenVPN is not installed.
- [CVE-2026-99287](https://ubuntu.com/security/CVE-2026-99287): vendor reports Noble vulnerable; unused Xen backend is now denied through normal module loading.

## Verification and update process

After the reboot, repeat file creation, modification, and deletion and confirm manager/indexer evidence. Recheck existing media endpoints, container identities, central Wazuh service state, and positive/negative lab network probes. The [post-reboot validation evidence](../evidence/kernel-hardening-validation.json) records actual results: all three real-time file operations were indexed with matching event identifiers and hashes, all 13 original container identities were unchanged, nine HTTP endpoint checks ended at 200, four central Wazuh services were active, and all five positive/negative network probes matched policy.

For later vendor updates, refresh the guest's signed Ubuntu metadata through a controlled maintenance path, install applicable supported updates, restart only the guest when needed, verify the running kernel, and let Wazuh inventory synchronize. Then review the new findings against vendor status again. This session did not install an unattended updater, scheduled reboot, or recurring advisory monitor. The lab has no permanent internet route.

No scanner findings were suppressed or deleted. Vendor-state lookups are point-in-time records; verify current advisories again before acting on a later update. Backup capacity work remains deferred by the owner; no backup settings were changed.

## Reversing these specific protections

Keep a Proxmox console session available. Remove only the two dedicated files named above, restore the previous live values (`kernel.unprivileged_userns_clone=1`, `kernel.kptr_restrict=1`), regenerate initramfs for the installed kernel, and restart the practice VM. Review why reversal is necessary before reopening these paths. Do not change the host or another guest by mistake.

## Download the record

- [Per-CVE review spreadsheet (CSV)](../evidence/kernel-triage.csv)
- [Structured review evidence (JSON)](../evidence/kernel-triage.json)
- [Original security review](security-review.md)
- [Ubuntu status source example](https://ubuntu.com/security/cves/CVE-2021-3773.json)
