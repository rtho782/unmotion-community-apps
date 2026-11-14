# unMotion for Unraid

Clone a VM on your server, move it to another Unraid host, or keep scheduled ZFS copies on a second host. unMotion provides these workflows in the Unraid web interface, with transfer progress, preflight checks and control over what happens to the source VM.

**Warm Move reduces downtime; it is not live migration.** The VM shuts down for the final transfer. Replicas do not start automatically when a host fails.

[Plugin source](https://github.com/rtho782/unmotion) · [Releases](https://github.com/rtho782/unmotion/releases) · [Help and bug reports](https://github.com/rtho782/unmotion/issues)

**Stable 0.5.0:** live migration with optional software TPM 2.0 and file/zvol disks, pooled capacity planning and disk/RAM progress, nested-aware CPU preparation/defaults, retained-copy cleanup and replication retirement/recreation are available through Community Apps and the stable feed. The opt-in beta feed offers **0.5.2-beta5**, described below. Live controls retain beta labels in the interface; software-TPM support remains experimental within the documented scope. Test disposable guests and keep verified backups.

**Retained-copy cleanup:** reclaim proven inactive live-migration disks and state through an explicit review on the non-owning host. Both paired hosts must be reachable and running 0.5.0. Current-owner data, native start fences and job history remain; legacy artifacts without proven identities stay protected. See [cleanup instructions and limits](https://github.com/rtho782/unmotion/blob/main/docs/LIVE-CLEANUP-BETA9.md).

## Features

### Opt-in beta: native live handover, SSH encryption and running-owner updates

**0.5.2-beta5** uses native OpenSSH/libvirt live handover without a custom payload forwarder or an extra application-controlled pause. On **both matching beta hosts**, enable **Settings → unMotion → Native live migration trust**. Temporary system-libvirt/forwarding access is administrator-equivalent, not a per-VM sandbox: use it only between trusted administrators. Permanent pairing keys stay restricted. Persistent start fences, authenticated ownership reconciliation and retained source disks remain. Ambiguous interruptions can require operator review; neither a missing host nor a retained copy grants permission to start another VM.

**Updating with a migrated VM running:** a settled live-migration owner may remain running when the target installer verifies identical native fencing code and its existing, exact QEMU admission. No VM shutdown, new ownership grant or process adoption is needed for that compatible update; retained old copies stay fenced. Finish/reconcile active work and pause replication first. Incompatible fencing changes, missing admission evidence, pending hand-back and active recovery-managed guests still block this exception. Read the specific refusal in the Plugins console rather than deleting ownership records. Linux file/zvol and Windows TPM/BitLocker running-owner upgrades passed on Unraid 7.3.4/7.4.0; see [upgrade behaviour and qualification](https://github.com/rtho782/unmotion/blob/main/docs/RUNNING-OWNER-UPGRADE-BETA5.md).

Choose **Settings → SSH encryption** for AES-128-GCM (default) or AES-256-GCM on outgoing live, Cold/Warm Move and replication connections. This may improve encryption-limited throughput, but hardware, storage and changed data still determine speed; AES-256 is not always slower. Live jobs pin their selection, existing sessions are unchanged, and no silent cipher fallback or plaintext option is offered.

The beta also allows same-vendor newer-to-older CPU migration when fresh checks validate the complete **active** guest CPU; changing only next-boot XML is insufficient. File/zvol round trips, Windows TPM/BitLocker checks, cached host reboots, receiver loss during copying and source loss after native resume passed bounded lab qualification. This is not automatic failover or exhaustive hardware certification. Read the [trust model](https://github.com/rtho782/unmotion/blob/main/docs/NATIVE-TRUSTED-MIGRATION.md) and [host-loss/reboot results and limits](https://github.com/rtho782/unmotion/blob/main/docs/NATIVE-HOST-LOSS-052-QUALIFICATION.md). The remaining feature instructions use the stable 0.5.0 baseline unless marked beta.

**Storage and progress:** live zvols stay zvols under the destination's configured zvol parent, with full reservations; file images remain sparse in its direct-pool image directory. Mixed file/zvol guests are supported without conversion or shared storage. Capacity checks combine requirements sharing a pool, with allocation overhead and free-space headroom. Disk-copy and RAM-convergence statistics are separate observations, not an overall percentage or promised ETA. Source and partial copies keep consuming storage until explicitly removed.

**Live readiness:** review independent CPU, storage, TPM, device and network blockers before starting. Submission remains disabled until fresh full preflight passes; Start checks both hosts again. Native TPM transfer needs no shared storage. Linux sealed-secret persistence and Windows Server 2025 Core with active BitLocker passed lab tests in both live directions and after protected cold boots. See the [qualification scope and limits](https://github.com/rtho782/unmotion/blob/main/docs/BETA7-QUALIFICATION.md).

**CPU defaults:** select a validated mode in **Settings → unMotion**, then choose **Use for new VMs**. The native VM form offers **unMotion default** and keeps existing custom CPU profiles unchanged. Saved form templates retain their own fixed profiles. **Clear default** restores Unraid's ordinary new-VM selection without changing existing VMs. Integration supports reviewed editor sources, qualified on Unraid 7.3.2 and 7.3.4; an unknown or externally changed editor is not overwritten. Check integration status after an Unraid update.

Online Warm Move preparation and Update tolerate growing file images while checking the received transfer length. Prepared copies remain provisional. Cutover still shuts down the source and forces the final image delta, even when file size/timestamps match, before starting the destination.

**Live migration — 0.5.0:** live migration moves an eligible running guest without a shutdown/reboot, with a handover pause. Both hosts need matching 0.5.0 and qualified QEMU/libvirt versions. The scope is BIOS or supported UEFI, optional software TPM 2.0, one to sixteen independent raw/qcow2 files on direct pool paths or whole raw ZFS volumes, and ordinary bridge NICs. Passthrough, attached ISOs, native ZFS-encrypted volumes, unrelated block devices and recovery/replication-managed guests are excluded; empty read-only CD-ROM drives may remain. UEFI requires matching reviewed OVMF and UUID-scoped raw variables; TPM requires matching swtpm/libtpms builds and ordinary local emulator state. See [live-migration instructions and limits](https://github.com/rtho782/unmotion/blob/main/docs/LIVE-MIGRATION-BETA.md).

Plan for **transfer time as well as RAM convergence**: requests abort at 30 minutes, and a fast NIC does not guarantee sufficient encrypted end-to-end throughput or a subsecond pause. 0.5.0 uses a packaged native disk/RAM connector and disposable job-restricted SSH keys; PHP admits/supervises endpoints and validates control RPCs, but does not relay VM payload. Keys and proven job-owned sessions are revoked on completion/abort. Fencing remains unchanged. See [final native qualification](https://github.com/rtho782/unmotion/blob/main/docs/NATIVE-TRANSPORT-QUALIFICATION.md) for scope, measurements and limits.

**CPU compatibility:** **Settings → unMotion** compares persistent local observations with paired hosts' cached masks and suggests an exact shared mode. One preferred revision per family is shown, with **Show all revisions** for older definitions; Q35/i440fx reference checks are automatic. A reviewed bulk action backs up and applies the selected mode to eligible persistent VM XMLs, preserving topology, pinning and other settings. It takes effect at the next full stop/start, not a guest reboot. Live migration also performs fresh VM-specific checks; a cached match alone is not permission. Cold/Warm Move do not require a shared CPU mode, including Intel/AMD where guest/VM settings are compatible. Older modes may reduce performance or omit required instructions, but can suit lightly loaded essential VMs. See [CPU compatibility](https://github.com/rtho782/unmotion/blob/main/docs/CPU-COMPATIBILITY.md).

CPU application preserves existing VMX/SVM exposure, including implicit host-passthrough nesting and explicit feature opt-outs. The augmented CPU is freshly validated locally; unsupported requirements are skipped, not silently removed. Secure XML reads/backups retain console credentials. Libvirt's harmless CPU-child reordering does not produce a false failure; real feature, topology and non-CPU differences still stop the batch. A common base mask does not itself prove nested workload compatibility on a peer.

- **Cold Move:** transfer a powered-off VM to a paired host and start it there.
- **Warm Move:** prepare a copy while the VM runs, update it, then transfer the final changes at cutover.
- **Flexible storage:** native ZFS transfers for zvols and dedicated VM datasets, with sparse file transfers for supported raw/qcow2 images, including images in shared datasets. Interrupted ZFS transfers can resume where supported.
- **VM state and media:** migrate UEFI variables and software TPM state, map compatible firmware by content, and copy, retain paths for, or detach attached ISOs.
- **Local cloning:** independent disks, new VM identifiers and NIC MAC addresses, with optional Ubuntu identity and DHCP customization.
- **Scheduled replication:** incremental ZFS recovery points with configurable frequency and retention, plus opportunistic TPM capture when the source VM shuts down.
- **Coordinated manual recovery:** activate an eligible replica after confirming that the reachable source is stopped and fenced. Native libvirt hooks reject unauthorized starts of recovery-managed VMs before QEMU starts.
- **Concurrent transfers:** independent Warm Move preparations and cutovers between compatible peers, with cancellable waiting for operations needing exclusive access.
- **Problem reports:** reviewable diagnostics containing selected job logs, VM/storage metadata and both hosts' available technical specifications.
- **Responsive inventory:** basic VM information appears before slower storage and guest-agent details. Browser tabs and authenticated peer display requests share a private RAM cache; operational safety checks always use fresh observations.

## Install and set up

Requires Unraid **7.3.2 or later**, VM Manager enabled, sufficient storage and destination RAM, and SSH connectivity between hosts. Install matching unMotion versions on both hosts. Local cloning needs only one host and no pairing.

Stable offers **0.5.0**, shown in Plugins as **2026.11.11.02-0.5.0**. Beta offers **0.5.2-beta5**, shown as **2026.11.14.01-0.5.2-beta5**. Fixed installer IDs are allocated during qualification, not derived from publication time. To opt in, install the beta installer below; subsequent Plugins-tab updates follow that channel. Existing stable installations do not switch automatically.

Current release availability:

**Replication controls:** end a schedule while keeping read-only replicas or deleting their exact owned data, then configure a separate new cycle for the same VM/destination. See [ending replication](#ending-replication) below. Both hosts need the applicable optional capabilities; these controls require the applicable capabilities on both hosts. Source point counts reflect acknowledged destination inventory and retention pruning, not recovery authority.

| Channel | Version | Installation |
| --- | --- | --- |
| Stable / Community Apps template | [0.5.0](https://github.com/rtho782/unmotion/releases/tag/0.5.0) | [Stable installer](https://raw.githubusercontent.com/rtho782/unmotion/plugin-stable/unmotion.plg) |
| Beta feed — opt-in prerelease | [0.5.2-beta5](https://github.com/rtho782/unmotion/releases/tag/0.5.2-beta5) | [Beta installer](https://raw.githubusercontent.com/rtho782/unmotion/plugin-beta/unmotion.plg) |

Migration/cutover and clone dialogs open immediately while fresh checks run, with submission disabled until checks finish. Background refreshes preserve unchanged VM details, controls, focus and tooltips. Logs preserve your reading position when you scroll back and follow output while at the bottom. A pairing test shows progress until it completes.

The VM list shows the latest available observation and its timestamp while refreshing in the background. Use **Refresh** to request another scan; existing rows stay visible. After a reboot, the first scan starts with basic information and fills in details. An unavailable VM Manager is reported as an error rather than an empty host. Cached display information never replaces the fresh safety checks performed before an action.

1. Install through the **Apps** listing when available, or paste the chosen installer link into **Plugins → Install Plugin**.
2. Open **Settings → unMotion** on each host. Check the destination image directory, optional zvol parent dataset and ISO directory. These describe where that host receives data; pool names do not have to match.
3. Pair using discovery or the other host's address, review its Ed25519 fingerprint and choose **Accept and pair**, then test the connection. Pairing uses the Unraid **root** account and keys restricted to authenticated unMotion operations. Pair only hosts you fully trust: permitted storage and VM operations are privileged.
4. Open **unMotion**, select a destination and review the VM inventory. Check the proposed paths, capacity, resources and choices in preflight.
5. Start with a disposable VM and verify its boot, networking and applications after migration.

Keep verified backups. Preflight checks prerequisites, not every possible guest or hardware dependency.

### Pairing hosts

Enter the other host's address, SSH port and root password, then choose **Pair**. The **Trust this host?** dialog displays the peer's Ed25519 fingerprint. Choose **Accept and pair** to trust that key and establish reciprocal pairing, or **Cancel** to return without pairing. New pairing requires 0.4.7 or later on both hosts; the initiating host needs 0.4.8 or later for this dialog.

Acceptance uses trust on first use, like an SSH client's first-connection prompt. For an independent check, expand **Verify manually (optional)** and compare the displayed fingerprint with this command on the peer's console or through a connection you already trust:

```bash
ssh-keygen -lf /etc/ssh/ssh_host_ed25519_key.pub -E sha256
```

The preview sends no password to the peer or installs any keys. After acceptance, unMotion checks the key again and pins only the accepted key; a change stops pairing before password authentication. Future connections enforce that saved pin, and the return host key travels through the authenticated pairing connection. Existing pairings and pins are preserved. For a deliberate host-key change, verify the replacement and resolve dependent replication/recovery policies before explicitly removing and recreating the pairing.

Automatic discovery uses Avahi/mDNS on the local network. Enable discovery on both hosts and allow multicast traffic; if a host is not listed, enter its address to pair directly. unMotion publishes only public connection metadata for discovery; private keys and settings remain restricted.

The installer downloads a versioned TXZ package from GitHub Releases and Unraid verifies its SHA-256 before installing it. The verified current package stays cached on the boot device for subsequent boots. After successful installation and registration, older recognized unMotion TXZ downloads are removed; settings, logs, VM storage and staged newer packages remain untouched. Initial installation and updates to uncached packages require access to GitHub.

### Upgrading from an older version

**Protocol 7 is unchanged since 0.4.2; existing pairings remain compatible.** New pairing requires 0.4.7 or later on both hosts. The supported OVMF override still requires its optional capability on both peers. Before updating, pause replication and finish/reconcile active work and cleanup. Stable 0.5.0 requires recovery/live-managed VMs stopped; the beta5 target installer permits settled live owners to remain running when their existing admission and unchanged fencing code pass compatibility checks. Active recovery-managed guests still require shutdown. Check pairing/native-hook readiness afterward before restarting managed guests or resuming schedules.

**Live migration needs exact matching application versions, live ABI capability 4 and native-data capability 1 on both peers.** Stable uses 0.5.0; the new native mode needs 0.5.2-beta5 plus native capability 1 and explicit trust on both hosts. Do not mix stable and beta for live moves. Otherwise-supported ordinary protocol-7 workflows retain their capability gates. Old queued guarded jobs must be resolved before creating a fresh native job; they are not converted silently. Pending hand-back or unresolved authority blocks upgrade/removal; do not bypass fences. Ordinary transfers after hand-back and encoded-zvol transfers retain their separate capability requirements.

**From 0.4.1 or earlier**, transfers to protocol-7 peers are blocked; there is no unrestricted SSH fallback. Plan a coordinated update:

1. Using the old version, finish or explicitly remove any prepared or partial Warm Moves you no longer need. Do not delete partial receive data by hand.
2. Pause scheduled replication and let active migration, clone, replication, recovery and cleanup jobs finish.
3. Shut down recovery-managed VMs before updating. Their authority records and native-autostart restrictions must remain intact.
4. Update both hosts through **Plugins**, refresh unMotion, then test the pairing on each host.
5. Check recovery's native-hook readiness before arming recovery or starting a managed VM. If verification requires a VM Manager restart, stop all VMs first, restart VM Manager, then allow automatic verification to finish and refresh recovery. unMotion never restarts VM Manager automatically.
6. Resume scheduled replication and use unMotion's recovery controls to start managed VMs.

Updates preserve settings, pairing identities and recovery records. Old receive state without provable peer ownership is retained, not silently adopted or deleted. Resolve a reported compatibility or ownership problem before continuing; do not remove SSH key restrictions or enable native VM autostart to bypass it.

Restricted transport rejects arbitrary host paths and custom QEMU command-line extensions except for the validated firmware setting below. Libvirt NVRAM files must use UUID-scoped names, or a supported layout beside the VM disk. If preflight reports an older unsupported NVRAM path, shut down the VM before correcting its file location and XML.

VMs using `-fw_cfg opt/ovmf/X-PciMmio64Mb,string=<number>` retain their OVMF PCI address-space setting during transfer. Both hosts require 0.4.9 or later; unsupported destinations are rejected before copying. Only this bounded decimal setting is permitted, not arbitrary QEMU options, extra arguments or file-backed firmware data. It does not allocate additional VM RAM or enable PCI/GPU passthrough migration. See [accepted XML syntax and limits](https://github.com/rtho782/unmotion/blob/main/docs/OVMF-FIRMWARE.md).

## Moving a VM

For **Cold Move**, shut down the source, choose **Cold migrate**, review destination RAM, vCPU, CPU-pinning and media options, then start the migration.

For **Warm Move**, choose **Prepare** while the source runs or is stopped. Wait for **READY**; **Update** refreshes the copy without cutting over. Choose **Cut over** when ready: unMotion requests a graceful shutdown, transfers the final disk changes and TPM/UEFI state, then starts the destination. A prepared copy is not a standalone bootable backup.

Independent preparations and cutovers can run together between matching peers. Cold moves, destination overwrite and copying attached ISOs wait cancellably for exclusive access. Concurrent operations on the same VM or overlapping storage remain blocked. Update, Cut over and Remove are unavailable while a migration job owns that VM; use the job's available controls or resolve an attention-required result first. Media choices appear only when an ISO is attached.

Preparation checks storage and pairing requirements; destination CPU/RAM startup availability is guidance until cutover, when it is checked again and enforced. Parallel transfers compete for storage, network and RAM resources; they do not bypass capacity checks.

For Cold/Warm Move, choose the source policy before migration:

- **Unregister and retain storage:** remove the source VM definition but keep its files.
- **Retain and rename:** keep the definition and disks, disable autostart and append the destination to its name.
- **Delete after validation:** remove the source VM and its owned storage after five minutes of continuous destination runtime.

Child datasets or unrelated contents select sparse file transfers of the VM disks. Automatic ZFS source cleanup removes only exact objects and this migration's snapshot. Other snapshots, holds, child datasets or changed ownership block cleanup and retain the source for inspection. Choose a retain policy when backups or other contents must remain. An interruption during the final dataset check can leave it read-only or unmounted; inspect the job before manually changing that state. See the [pairing and storage safety guidance](https://github.com/rtho782/unmotion/blob/main/docs/SECURITY-047.md).

If destination startup fails, fix the reported problem and start it manually, then use **Resolve migration**. unMotion verifies the result and completes the original source policy, including a requested rename, without repeating the transfer or overwriting your repaired configuration. Do not start a retained source while its destination copy is running.

### Live migration — beta only

Use disposable guests first. Both hosts need matching 0.5.0, reciprocal pairing and verified native hooks. The current adapter is qualified for **libvirt 12.2.0** with matching QEMU/libvirt, an exact versioned Q35/i440fx machine and a same/newer CPU generation from the same vendor. Apply an appropriate shared custom CPU mode and fully stop/start the guest first; changing next-boot XML alone is insufficient. The one to sixteen writable disks may mix independent raw/qcow2 files on direct local pool paths and whole raw ZFS volumes, with disk cache `none` or `directsync`. Zvols must be writable, device-visible and without native ZFS encryption; guest BitLocker is separate. A disk referenced by another VM is blocked. Equivalent physical networks remain your responsibility.

Review the destination capacity summary before starting. unMotion budgets full virtual sizes, combines requirements on the same pool and includes allocation overhead plus 20% free-space headroom. Zvol creation uses and verifies a full reservation, not thin overcommit; destination dedup policy is preserved. File images stay sparse, but sparse source usage is not a promise of future capacity or network transfer size. Other workloads can still consume free space. Source and failed/cancelled partial copies are retained, so repeat attempts can require additional storage.

Software TPM 2.0 (`tpm-tis` or `tpm-crb`) transfers through the native migration stream into a separate job/UUID-bound directory. No shared TPM disk or running-file copy is required. Both hosts need freshly matching swtpm/libtpms builds. External/passthrough TPMs, TPM 1.2 and encrypted/custom-profile TPM backends are excluded; guest disk encryption such as BitLocker is separate. Keep guest recovery keys available and validate protected boot on the destination. Old source and partial TPM state are retained, not overwritten.

Intel nested hypervisors require KVM nesting enabled on both hosts and validation of the complete active CPU contract, including nested features. AMD SVM CPU configuration is supported, but SVM-enabled guests cannot migrate live in 0.5.0 because the kernel warns against migrating active nested AMD workloads. Use otherwise-supported Cold/Warm Move instead. unMotion does not change host module settings. Test the nested guests' own CPU requirements after deliberately reducing a parent VM's CPU feature set. See [kernel guidance](https://docs.kernel.org/virt/kvm/x86/running-nested-guests.html).

Select the destination, then **Live migrate (beta)**. The readiness dialog lists independent blockers; use **Recheck** after correcting them. Submission stays disabled until fresh full preflight succeeds, and Start rechecks both hosts. The job samples encrypted throughput and RAM dirty rate, then copies disk and RAM while the guest runs; **4× headroom** is recommended, not guaranteed. Later workload changes can prevent convergence. Follow progress/logs on the source: disk copy and RAM pre-copy/convergence show separate remaining-byte observations and RAM passes when available, not a combined completion percentage or ETA. The destination shows received transport bytes, including overhead and retransmission. At handover, the receiver remains paused until the source is stopped and durably fenced. Guest UUID/MACs, CPU and resources are retained. Nested-lab packet gaps were approximately **1.85–2.13 seconds**, not a guaranteed 500 ms stun. CPU badges show active/next-boot observations, not migration permission. If browser login expires, reload and sign in again; server-side jobs continue.

**Request cancellation** uses libvirt abort, not a forced guest shutdown. After an interrupted job, use **Reconcile** on the source once its worker/native jobs have stopped; both hosts must be reachable. Reconciliation either confirms ownership at the destination or proves the receiver stopped before returning source ownership. If an attempt failed before a receiver record existed, fresh authenticated callbacks must prove that the target is absent and has no owning/recovery authority; a missing record alone grants nothing, and partial data is retained. Reconcile does not automatically boot a stopped source. Do not manually resume an incoming VM, start a retired source or delete authority files.

Only one live job may run per host. Source definitions are unregistered after confirmed success, while source/partial files are retained. Native owner starts and further live moves are supported. To return to ordinary workflows, fully shut down the owner, choose **Return to ordinary workflows** on its settled live-job row and confirm the paired checks. Both hosts must be reachable and the peer's copy absent/stopped and retired/aborted. Hand-back enables otherwise-supported Cold/Warm Move, cloning, CPU edits and replication-only configuration; TPM guests still cannot be cloned and non-owner copies remain fenced. Recovery arming after hand-back is not supported. A pending acknowledgement keeps the owner fenced until a successful retry.

An interrupted ordinary transfer of a formerly live-managed VM can offer **Finish guarded handover**. This rechecks the stopped source, destination definition and paired ownership without starting a VM or deleting storage. After it succeeds, start the destination when ready and use **Resolve migration** to finish the original source policy. Do not bypass start fences or boot a retained source.

Automatic disk reclamation and failover are not implemented. Final 0.5.0 native transport tests passed on Unraid 7.3.4/7.4.0, including Linux file/zvol interruption/retry, full-pattern mixed-disk round trips and Windows Server 2025 Core/active TPM-only BitLocker live/protected cold boots. Earlier physical-host and host-loss checks remain bounded inherited control/fencing evidence, not a fresh production-native transport test. Windows 11/TPM+PIN, AMD nested live migration and physical 10/25 GbE remain unqualified. See [requirements](https://github.com/rtho782/unmotion/blob/main/docs/LIVE-MIGRATION-BETA.md) and [final qualification](https://github.com/rtho782/unmotion/blob/main/docs/NATIVE-TRANSPORT-QUALIFICATION.md).

### Clean up an inactive live copy

On the host holding the inactive, non-owning copy, find its settled live job and choose **Clean up retained copy**. Finish or reconcile interrupted handovers first; cleanup does not stop or unregister VMs. Review the exact disks and state paths, then confirm **Delete reviewed data**. The current owner must be reachable and able to prove its ownership, but its VM can remain running.

The worker rechecks the whole plan and records per-object progress. Held snapshots, replaced files/volumes, VM or kernel references, open files and unproven legacy identities protect data from deletion. New ZFS file identities use dataset GUIDs and inodes rather than temporary import-order device numbers. Storage-allocation figures are estimates, not guaranteed reclaimed space. Shared parent directories are not recursively removed.

If cleanup is **BLOCKED**, correct the displayed reason and choose **Review/retry cleanup**. A host restart never resumes deletion automatically. **Cleanup receipt** records completion without removing job history or native start fences. Cleanup is not available for current-owner data or after responsibility has moved to ordinary hand-back/recovery authority. Keep those records intact; see the [complete workflow](https://github.com/rtho782/unmotion/blob/main/docs/LIVE-CLEANUP-BETA9.md).

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

Use **End replication** to retire stored replicas through the paired workflow described below. Do not delete replica data or ownership records manually.

### Ending replication

With **0.5.0 on both hosts**, use **Disable** for a reversible pause, or **End replication** to permanently end a cycle. Review the affected data and choose **Keep replicas read-only** or **Delete replica data**. The ordinary source VM can keep running; its writable disks are never deleted. Kept copies remain inert inventory, not runnable recovery VMs, and can be removed later with **Delete retained replicas**.

Partial receives are preserved unless you explicitly select **Also discard any partial receive**. An offline peer, unresolved recovery authority or foreign storage reference can leave retirement **BLOCKED** with scheduling stopped and data retained. Resolve the reason and choose **Retry retirement**; do not bypass it by deleting records or holds. Source bases are released only after paired destination acknowledgement. Warm Move removal follows the same acknowledgement-first ordering and retains logs.

After ending completes, choose **Replicate** beside the VM to configure a **new cycle** for that destination. It starts a full copy with separate storage/identity, not reuse of old points or partial receives. Retained old cycles remain visible and independently removable; allow space for both copies. An unfinished retirement cannot be bypassed, and only one current cycle per VM/destination is permitted. Retained replicas or unfinished cleanup block uninstall so ownership records are not orphaned. See the [workflow and limits](https://github.com/rtho782/unmotion/blob/main/docs/REPLICATION.md#ending-replication) and [Beta6 qualification](https://github.com/rtho782/unmotion/blob/main/docs/TESTING-RETIREMENT-BETA6.md).

## Other limitations

- GPU/PCIe passthrough blocks migration. USB configuration can be retained or removed, but hardware must be physically attached to the destination before guest startup.
- Destination network bridges, firmware and CPU settings must suit the VM. Migration does not recreate networks or change guest VLAN/static-IP settings.
- File-copy migration supports shared image datasets, not concurrent shared writable guest disks. Unsupported layouts and external qcow2 backing chains are rejected.
- Native ZFS transfers use individual dataset or zvol streams and explicit destination settings; arbitrary source dataset properties are not imported.
- Warm Move still needs shutdown, final-transfer and startup time. Busy disks or slow storage/networking increase downtime; parallel transfers compete for resources.
- Replication is not a replacement for separate backups and tested recovery procedures.

## Updates and support

Use **Plugins → Check for Updates → Update**, following the coordinated upgrade steps above, then refresh unMotion. Community Apps and both feeds offer 0.5.0; beta installations keep their opt-in URL for future prereleases. To change tracks, install the appropriate feed URL through **Plugins → Install Plugin**, keeping the descriptor named **unmotion.plg**. Both live peers need matching 0.5.0/native-data capability 1; older protocol-7 hosts retain otherwise-supported ordinary operations. The verified package stays cached for offline boot restoration.

Choose **Report a problem** on a migration or clone job, including a failed or stuck job. It collects recent logs, relevant VM/storage metadata and available specifications for both hosts. Review and edit the anonymised preview, optionally retain original paths, then download or copy it. Redaction is best-effort: check filenames, paths and logs before sharing publicly.

For a failed preparation with verified evidence that storage copying never began, **Archive failed record** removes the entry from the active list while retaining its logs locally. Removing a preparation with copied or partial storage is a separate, confirmed cleanup operation.

**Open GitHub issue draft** does not upload the report. Sign in to GitHub, attach the reviewed file or paste it, describe the problem and submit it yourself. Without a GitHub account, you can still save a report and share it with someone helping you.

## About

This repository contains the Community Applications template and user guide. The [main repository](https://github.com/rtho782/unmotion) contains the plugin source, technical documentation and downloads. The Apps template installs stable independently of beta development.

Author: **Richard Skinner**. Copyright (C) 2026 Richard Skinner. unMotion and this repository are licensed under [GPL-3.0-only](LICENSE), with no warranty under the licence terms.
