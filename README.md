# gaia-archtree

Ignition skill for GAIA archtree: two adversarial binary trees (陽 BUILD /
陰 ADVERSARY) spawned via `gaia summon`, rooted at the calling agent, up to
depth 4 (16 nodes/side). Trigger phrase (as documented in the skill):
"starte gaia archtree" -- no questions, ignite.

This package ships only the **ignition skill** (`skills/gaia-archtree/
SKILL.md`, copied verbatim from `~/.gaia/skills/gaia-archtree/SKILL.md`):
root routing, ward-design topology, node specs, adversary-pass mandate. The
actual `/archtree` mechanism (room-tree registration, root-lane lifecycle,
`addArchtreeRoot`) is daemon-side (`src/services/archtree.ts`) and stays in
gaia-daemon -- it is not part of this package.

## Requires

gaia-daemon >= `4da2981` for `/archtree` to be reachable at all.

## Install

```
pi install git:github.com/pascaldisse/gaia-archtree
```
