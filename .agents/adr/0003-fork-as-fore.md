# Fork as `fore`: own marketplace, own plugin name, version lineage reset

This repo was forked from [mattpocock/skills](https://github.com/mattpocock/skills) to carry the same skill set under an independent identity, installed through its own Claude Code marketplace rather than the upstream official listing. The fork point is upstream 1.2.3.

## Decision

- **Marketplace name `wxitzxg`**, declared in `.claude-plugin/marketplace.json`. The marketplace *is* this repo; adding `wxitzxg/fore` registers it.
- **Plugin name `fore`**, declared in both `.claude-plugin/plugin.json` and `package.json`. The install reference is `fore@wxitzxg`, and command prefixes become `/fore:<skill>`.
- **Version lineage resets at 0.1.0.** Fore versions and upstream 1.x versions number different things, and equal numbers must never mean equal contents.
- **Two meta-skills renamed**: `ask-matt` becomes `guide` (the router), `setup-matt-pocock-skills` becomes `setup`. Skill *behaviour* and every other skill directory are unchanged.
- **Repo renamed** from `wxitzxg/skills` to `wxitzxg/fore`, so marketplace name, plugin name and repo name all coincide. GitHub redirects the old URL.
- **History preserved**: the upstream `CHANGELOG.md`, `.changeset/`, and upstream ADRs (0001, 0002) stay as the historical record; only a fork notice is prepended. The MIT license keeps the upstream copyright line and adds our own.

## Why these names

The method the skills implement has two phases: relentless interview and specification up front (grilling, `to-spec`, domain modeling), then unhesitating execution (`implement`, `tdd`). "fore" carries both: *forethought* before, and moving *fore* after. The tagline is "Forethought first, then move fore." The marketplace keeps the owner identity (`wxitzxg`) because a marketplace can host multiple plugins; the product inside it gets the short name. Plugins in the local registry (context7, superpowers, claude-mem) carry no `-skills` suffix, so neither does this one.

Alternatives rejected: keeping `mattpocock-skills` as the plugin name (daily command prefix stays someone else's brand, and a second marketplace named `mattpocock` could collide with the upstream one); descriptive names like `sdd` or `specdriven` (accurate but generic, no philosophy); Chinese-philosophy names (`gewu`, `gezhi`) and Greek (`logos`, `elenchus`) considered for the philosophy brief, set aside in favour of an English word with the right command feel.

## Trade-off: upstream sync

The cost is paid in merge conflicts when pulling `upstream` (the `mattpocock/skills` remote):

- **Cheap and predictable**: `.claude-plugin/plugin.json`, `.claude-plugin/marketplace.json`, `package.json` conflict on identity lines (names, version, author). Resolve by keeping fore identity and taking upstream content changes.
- **Structurally annoying but small**: `guide/` and `setup/` were renamed from upstream directories. Upstream edits there arrive as modify/delete conflicts and get hand-ported. Both are meta-skills upstream changes rarely, so the surface is two directories, not the repo.
- **Untouched**: every other skill body is byte-for-byte upstream, so merges there are automatic. This is why skill bodies were *not* de-branded and why third-party links inside skill files were left alone. Deeper rebrand of skill bodies would forfeit automatic merges across the whole tree, which option C in the fork interview would have cost.

## Invariants this creates

- Identity lives only in the packaging layer (manifests, README, install block, docs pages) plus the two renamed meta-skills. When upstream changes a skill body, the merge should still apply cleanly.
- Fore version numbers are independent. Never claim parity with an upstream 1.x version; the fork point (1.2.3) and subsequent upstream SHAs merged in are recorded in git history, not faked through version numbers.
- Attribution stays visible: LICENSE keeps `Copyright (c) 2026 Matt Pocock`, README carries a fork attribution section. Removing either would breach the MIT license.
- Nothing fore ships may present itself as an official Matt Pocock product or imply his endorsement.
