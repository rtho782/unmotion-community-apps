# unMotion for Unraid

Clone a VM on your server, move it to another Unraid host, or keep scheduled ZFS copies on a second host. unMotion provides these workflows in the Unraid web interface, with transfer progress, preflight checks and control over what happens to the source VM.

**Warm Move reduces downtime; it is not live migration.** The VM shuts down for the final transfer. Replicas do not start automatically when a host fails.

[Plugin source](https://github.com/rtho782/unmotion) · [Releases](https://github.com/rtho782/unmotion/releases) · [Help and bug reports](https://github.com/rtho782/unmotion/issues)

## Features

- **Cold Move:** transfer a powered-off VM to a paired host and start it there.
- **Warm Move:** prepare a copy while the VM runs, update it, then transfer the final changes at cutover.
- **Flexible storage:** native ZFS transfers for zvols and dedicated VM datasets, with sparse file transfers for supported raw/qcow2 images, including images in shared datasets. Interrupted ZFS transfers can resume where supported.
- **VM state and media:** migrate UEFI variables and software TPM state, map compatible firmware by content, and copy, retain paths for, or detach attached ISOs.
- **Local cloning:** independent disks, new VM identifiers and NIC MAC addresses, with optional Ubuntu identity and DHCP customization.
- **Scheduled replication:** incremental ZFS recovery points with configurable frequency and retention, plus opportunistic TPM capture when the source VM shuts down.
- **Coordinated manual recovery:** activate an eligible replica after confirming that the reachable source is stopped and fenced. Native libvirt hooks reject unauthorized starts of recovery-managed VMs before QEMU starts.
- **Concurrent transfers:** independent Warm Move preparations and cutovers between compatible peers, with cancellable waiting for operations needing exclusive access.
- **Problem reports:** reviewable diagnostics containing selected job logs, VM/storage metadata and both hosts' available technical specifications.

## Install and set up

Requires Unraid **7.3.2 or later**, VM Manager enabled, sufficient storage and destination RAM, and SSH connectivity between hosts. Install matching unMotion versions on both hosts. Local cloning needs only one host and no pairing.

The application and GitHub release use version **0.4.3**. Unraid's Plugins tab shows **2026.10.04.01-0.4.3**, combining a fixed release identifier with the familiar version so updates sort correctly.

Current release availability:

| Channel | Version | Installation |
| --- | --- | --- |
| Stable / Community Apps template | [0.4.3](https://github.com/rtho782/unmotion/releases/tag/0.4.3) | [Stable installer](https://raw.githubusercontent.com/rtho782/unmotion/plugin-stable/unmotion.plg) |
| Beta feed | 0.4.3 stable graduation; no newer beta | [Graduation installer](https://raw.githubusercontent.com/rtho782/unmotion/plugin-beta/unmotion.plg) |

1. Install through the **Apps** listing when available, or paste the chosen installer link into **Plugins → Install Plugin**.
2. Open **Settings → unMotion** on each host. Check the destination image directory, optional zvol parent dataset and ISO directory. These describe where that host receives data; pool names do not have to match.
3. Pair the hosts using discovery or the other host's address, then test the connection. Pairing uses the Unraid **root** account, but plugin-managed SSH keys are restricted to authenticated unMotion operations, not a general root shell. Pair only hosts you fully trust: permitted storage and VM operations are still privileged.
4. Open **unMotion**, select a destination and review the VM inventory. Check the proposed paths, capacity, resources and choices in preflight.
5. Start with a disposable VM and verify its boot, networking and applications after migration.

Keep verified backups. Preflight checks prerequisites, not every possible guest or hardware dependency.

The installer downloads a versioned TXZ package from GitHub Releases and Unraid verifies its SHA-256 before installing it. The verified current package stays cached on the boot device for subsequent boots. After successful installation and registration, older recognized unMotion TXZ downloads are removed; settings, logs, VM storage and staged newer packages remain untouched. Initial installation and updates to uncached packages require access to GitHub.

### Upgrading from an older version

**0.4.3 uses protocol 7, unchanged from 0.4.2; these versions remain compatible.** For a 0.4.2 update, pause scheduled replication, finish active jobs and shut down recovery-managed VMs before installing. Existing prepared copies can remain. Check pairing/native-hook readiness afterwards, then resume scheduling and use unMotion to start managed VMs.

**From 0.4.1 or earlier**, transfers to protocol-7 peers are blocked; there is no unrestricted SSH fallback. Plan a coordinated update:

1. Using the old version, finish or explicitly remove any prepared or partial Warm Moves you no longer need. Do not delete partial receive data by hand.
2. Pause scheduled replication and let active migration, clone, replication, recovery and cleanup jobs finish.
3. Shut down recovery-managed VMs before updating. Their authority records and native-autostart restrictions must remain intact.
4. Update both hosts through **Plugins**, refresh unMotion, then test the pairing on each host.
5. Check recovery's native-hook readiness before arming recovery or starting a managed VM. If verification requires a VM Manager restart, stop all VMs first, restart VM Manager, then allow automatic verification to finish and refresh recovery. unMotion never restarts VM Manager automatically.
6. Resume scheduled replication and use unMotion's recovery controls to start managed VMs.

Updates preserve settings, pairing identities and recovery records. Old receive state without provable peer ownership is retained, not silently adopted or deleted. Resolve a reported compatibility or ownership problem before continuing; do not remove SSH key restrictions or enable native VM autostart to bypass it.

Restricted transport rejects arbitrary host paths and custom QEMU command-line extensions. Libvirt NVRAM files must use UUID-scoped names, or a supported layout beside the VM disk. If preflight reports an older unsupported NVRAM path, shut down the VM before correcting its file location and XML.

## Moving a VM

For **Cold Move**, shut down the source, choose **Cold migrate**, review destination RAM, vCPU, CPU-pinning and media options, then start the migration.

For **Warm Move**, choose **Prepare** while the source runs or is stopped. Wait for **READY**; **Update** refreshes the copy without cutting over. Choose **Cut over** when ready: unMotion requests a graceful shutdown, transfers the final disk changes and TPM/UEFI state, then starts the destination. A prepared copy is not a standalone bootable backup.

Independent preparations and cutovers can run together between matching peers. Cold moves, destination overwrite and copying attached ISOs wait cancellably for exclusive access. Concurrent operations on the same VM or overlapping storage remain blocked. Update, Cut over and Remove are unavailable while a migration job owns that VM; use the job's available controls or resolve an attention-required result first. Media choices appear only when an ISO is attached.

Preparation checks storage and pairing requirements; destination CPU/RAM startup availability is guidance until cutover, when it is checked again and enforced. Parallel transfers compete for storage, network and RAM resources; they do not bypass capacity checks.

Choose the source policy before migration:

- **Unregister and retain storage:** remove the source VM definition but keep its files.
- **Retain and rename:** keep the definition and disks, disable autostart and append the destination to its name.
- **Delete after validation:** remove the source VM and its owned storage after five minutes of continuous destination runtime.

If destination startup fails, fix the reported problem and start it manually, then use **Resolve migration**. unMotion verifies the result and completes the original source policy, including a requested rename, without repeating the transfer or overwriting your repaired configuration. Do not start a retained source while its destination copy is running.

## Cloning

Shut down the source, choose **Clone locally**, enter a unique name and select guest customization. The independent clone finishes stopped with autostart disabled.

Without customization, its NICs remain disconnected for console-based review. **Ubuntu DHCP** customization requires a working QEMU Guest Agent. It boots the clone with NICs disconnected, resets its hostname, machine ID and SSH host keys, replaces Netplan networking with DHCP, then shuts it down. NICs are enabled only after successful customization.

Review application identities, licences and service configuration yourself. Customization is not universal across operating systems. Software-TPM and PCI-passthrough guests cannot be cloned; USB passthrough is removed from clones.

## Replication and manual recovery

Choose **Replicate**, select the destination and set the recovery-point interval (RPO) and retention sliders. Intervals range from **5 minutes to 24 hours**. Historical points are distributed across the last 24 hours while retaining the newest verified point; retention choices depend on the interval. Slow transfers can cause the actual recovery-point age to exceed the target.

Every writable disk must be a dedicated ZFS zvol or an image in an isolated ZFS dataset. Shared datasets, non-ZFS storage, native ZFS encryption and qcow2 backing chains are unsupported for replication. Use **ZFS Master** or another storage tool to arrange datasets; unMotion does not convert the layout.

Replica storage stays read-only and inactive. Install QEMU Guest Agent for consistency checks and recovery-eligibility metadata. Without eligible agent evidence, copies can remain replication-only. Failed or unsupported filesystem freezing may produce a crash-consistent copy rather than an application-consistent one.

TPM checkpoints show their capture quality. Powered-off captures are safer than best-effort running captures, and the last verified safe checkpoint is retained. Test TPM-backed recovery with your guest and keep its recovery keys available.

Shut down the source VM before explicitly arming recovery, then use the destination's **Recovery** controls. Arming requires a verified native libvirt hook, gives unMotion control of source autostart and keeps native Unraid autostart disabled until ownership is safely reconciled. Existing Unraid hooks are preserved. Only one destination may be armed per VM. Activation requires an eligible point and a reachable, stopped, fenced source; separate writable copies leave retained points intact.

Start recovery-managed VMs through unMotion, not the native VM page or `virsh`: unauthorized starts are rejected before QEMU starts. **Start source VM** requests one authorized start without enabling autostart; **Start recovered VM** rechecks authority before starting an existing activation. Direct restore, attach and live-migration routes are also blocked for managed VMs. Disarm and reconcile recovery before removing the plugin.

**Automatic failover and recovery from an unreachable source are not available.** Neither are quorum/witness voting, alternate TPM-backup selection or failback transfer. Cold failback currently provides prerequisite checks only.

## Other limitations

- GPU/PCIe passthrough blocks migration. USB configuration can be retained or removed, but hardware must be physically attached to the destination before guest startup.
- Destination network bridges, firmware and CPU settings must suit the VM. Migration does not recreate networks or change guest VLAN/static-IP settings.
- File-copy migration supports shared image datasets, not concurrent shared writable guest disks. Unsupported layouts and external qcow2 backing chains are rejected.
- Native ZFS transfers use individual dataset or zvol streams and explicit destination settings; arbitrary source dataset properties are not imported.
- Warm Move still needs shutdown, final-transfer and startup time. Busy disks or slow storage/networking increase downtime; parallel transfers compete for resources.
- Replication is not a replacement for separate backups and tested recovery procedures.

## Updates and support

Use **Plugins → Check for Updates → Update**, following the coordinated upgrade steps above, then refresh unMotion. Both feeds currently offer 0.4.2 stable; installing it selects future stable updates. Future betas require explicit opt-in. Keep the installed descriptor named **unmotion.plg** rather than creating another plugin entry. The application reports **0.4.2** and the Plugins tab reports **0.4.2-stable** for update ordering.

Choose **Report a problem** on a migration or clone job, including a failed or stuck job. It collects recent logs, relevant VM/storage metadata and available specifications for both hosts. Review and edit the anonymised preview, optionally retain original paths, then download or copy it. Redaction is best-effort: check filenames, paths and logs before sharing publicly.

For a failed preparation with verified evidence that storage copying never began, **Archive failed record** removes the entry from the active list while retaining its logs locally. Removing a preparation with copied or partial storage is a separate, confirmed cleanup operation.

**Open GitHub issue draft** does not upload the report. Sign in to GitHub, attach the reviewed file or paste it, describe the problem and submit it yourself. Without a GitHub account, you can still save a report and share it with someone helping you.

## About

This repository contains the Community Applications template and user guide. The [main repository](https://github.com/rtho782/unmotion) contains the plugin source, technical documentation and downloads. The Apps template installs stable independently of beta development.

Author: **Richard Skinner**. Copyright (C) 2026 Richard Skinner. unMotion and this repository are licensed under [GPL-3.0-only](LICENSE), with no warranty under the licence terms.
