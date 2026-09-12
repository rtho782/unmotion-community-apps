# unMotion Community Applications template

This repository contains the Community Applications plugin template for [unMotion](https://github.com/rtho782/unmotion).

- Author: Richard Skinner
- Template: `unmotion.xml`
- Current channel: stable `0.4.0` (`0.4.0-stable` in Unraid's Plugins tab for update ordering)
- Plugin feed: [stable unmotion.plg](https://raw.githubusercontent.com/rtho782/unmotion/plugin-stable/unmotion.plg)
- CA installation manifest: pinned to the verified feed commit in `unmotion.xml`, with the stable filename `unmotion.plg`. Its embedded update URL follows `plugin-stable`. Advance the template pin after publishing and verifying each new stable release. The initial branch rename changes only feed metadata, not the 0.4.0 package or version; historical GitHub release assets remain unchanged.
- Support: [GitHub issues](https://github.com/rtho782/unmotion/issues)

The repository is intentionally separate from the plugin source so Community Applications can consume its XML without encountering unrelated XML files. It has not yet been submitted to the Community Applications feed.

## Licence

Copyright (C) 2026 Richard Skinner. This repository's templates, metadata and documentation are licensed under **GPL-3.0-only**, not GPLv3-or-later. See [LICENSE](LICENSE). unMotion's source and installed package carry the same licence. No warranty is provided under the licence terms.

## Submission

The root `ca_profile.xml` supplies maintainer information. Submit this public repository through https://ca.unraid.net/submit/new, run Validate and Scan, resolve review findings, then submit for manual plugin review. Publication of this repository is not Community Applications approval. The current support destination is GitHub issues; add an Unraid forum support topic when available.

Stable release notes explicitly exclude live migration, automatic failover, unreachable-source recovery and failback transfer. The minimum declared Unraid version remains 7.0.0; matching plugin versions on both peers are recommended.
