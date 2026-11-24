# Security review — November 24, 2026

A subsequent [kernel findings review](kernel-findings-review.md) records per-CVE vendor reconciliation and additional exposure restrictions. The results below preserve the initial review history.

## Scope and conclusion

Reviewed the dedicated FIM endpoint, its host forwarding/firewall rules, and targeted continuity of the existing media and Wazuh services. This was not a full homelab penetration test, external exposure audit, or zero-vulnerability certification.

Wazuh currently reports **440 package/CVE records** for this lab after the update review. These are scanner findings, not a count of successful attacks. The refreshed records all concern the current kernel, including 2 Critical and 111 High ratings. The installed kernel matches the available Ubuntu repository candidate. Remaining findings require vendor fixes and applicability review. The lab cannot be described as vulnerability-free.

## Verified hardening

- Effective SSH settings: root login disabled, password and keyboard-interactive authentication disabled, public-key authentication enabled, SSH user restricted to the lab administrator. Fresh key-based login succeeded.
- Runtime guest/host allow rules now permit only TCP 1514 for lab event forwarding. The TCP 1515 enrollment socket is stopped and disabled. Provisioning evidence retains its historical two-port state.
- Guest has no default route. Event connection succeeded; enrollment, host administration, selected media access, and public HTTPS probes failed as expected.
- Ubuntu metadata was refreshed through a temporary source-restricted proxy to official Ubuntu archives using verified upstream HTTPS and signed repository metadata. No unrestricted route was added. The proxy was stopped and its temporary script removed.
- 67 packages upgraded, 6 installed. New kernel `6.8.0-151-generic` booted successfully. Four obsolete image/module/header packages for `6.8.0-147` were purged after confirming the new boot.
- One update remains phased: `libsqlite3-0`, installed `3.45.1-1ubuntu2.8` from noble-security, candidate `3.45.1-1ubuntu2.11` from noble-updates at 10% rollout. Ubuntu's phasing was respected.
- Agent configuration checks and `dpkg --audit` passed; no failed systemd units were listed.
- Create/modify/delete FIM events passed again in manager and indexer, with matching event identifiers and file hashes. Observed indexed times: 9.83 s, 10.76 s, 10.77 s.
- All 13 original media container identities were unchanged; nine HTTP endpoint checks ended at 200 after redirects. Four central Wazuh services were active. These checks do not establish full playback or application security.

## Vulnerability inventory

The pre-update lab query reported 3,335 package/CVE documents, including 29 Critical and 866 High severity records. Most records referenced the old kernel package. Those counts describe the scanner inventory, not confirmed exploitation.

Latest reported total: **440**. Severity counts: `{"Medium": 232, "High": 111, "-": 64, "Low": 31, "Critical": 2}`. See the evidence file for scope and current state. Package installation and removal were verified directly on the endpoint. Wazuh correlates synchronized inventory with its vulnerability feed; a refreshed operating system alone is not proof that all inventory findings are resolved.

Do not delete findings to obtain a clean dashboard. Reconcile package versions, supported vendor fixes, and applicability. Maintain isolation and periodic patching. Review broader host and manager updates in a planned maintenance window with a recovery plan.

## Critical finding review

- [CVE-2021-3773](https://ubuntu.com/security/CVE-2021-3773): Ubuntu lists Noble as ignored because no upstream fix is available. The description concerns OpenVPN endpoint information; OpenVPN was not installed on this guest. This limits the described exposure, but does not erase the package-level finding.
- [CVE-2026-99287](https://ubuntu.com/security/CVE-2026-99287): Ubuntu lists Noble as vulnerable. The affected Xen network backend is a loadable module in this kernel. It was not loaded and this is a KVM guest with no `/proc/xen`. These observations reduce applicability to the current use, rather than proving the whole kernel safe.
- `linux-image-virtual` was installed at the repository candidate, `6.8.0-151.151`. No newer candidate was offered by the refreshed configured Ubuntu archives at review time. No findings were suppressed or deleted.

## Operational limits

The live guest is outside the current backup job while additional storage is pending. A rebuild guide is not a tested backup. FIM change reporting can include synthetic file text; do not put secrets in the training directory. The host remains a shared failure domain. Negative probes cover selected destinations, not every possible path.

## Evidence and references

- [Sanitized security checks](../evidence/security-review.json)
- [Plain-English summary](../PLAIN-ENGLISH-SUMMARY.md) / [PDF](../plain-english-summary.pdf)
- [Initial exercise evidence](../evidence/results.json)
- [Wazuh inventory and vulnerability correlation](https://documentation.wazuh.com/current/user-manual/capabilities/vulnerability-detection/how-it-works.html)
- [Ubuntu phased updates](https://documentation.ubuntu.com/server/explanation/software/about-apt-upgrade-and-phased-updates/)
