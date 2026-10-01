# llama.cpp patch replay

`llama-kvmem-current.patch` is the cumulative diff cut against Prism `9a9394a`.
It still applies cleanly on the current upstream `prism` tip `bdc23b56`
(19 commits ahead; only two context offsets: `src/models/qwen35.cpp` hunk #4
by +14 lines, `tests/test-backend-ops.cpp` hunk #1 by +25 lines, checked with
`git apply --check` on 2026-10-01), and the CI workflow pins that commit.
It includes the existing KVMem hooks, multimodal batch, MTP, media
parser and mtmd helper extensions, plus FP32 GDN Record/Fold for ReplaySSM.
It also fixes reasoning-budget initialization from a template's generation prefix.
`scripts/apply-patches.sh` applies it
without creating commits and checks for an already applied tree.

The former `ci-patches/0001-qwen35-mtp-hadamard-inverse.patch` is retired:
upstream commit `518ad108` ("qwen35: apply the Hadamard inverse to the MTP
token-embedding lookup", PR #205) carries the same fix, so the local patch no
longer applies and must not be overlaid. It is kept as
`ci-patches/0001-qwen35-mtp-hadamard-inverse.patch.retired-20261001` for
reference. Upstream `288859a9` (PR #210) does the same for the dflash
borrowed embedding and head.

`reasoning-budget-upgrade.patch` upgrades the v0.15.0 ReplaySSM tree.
`replayssm-upgrade.patch` upgrades the preceding multimodal/query-replay tree.
`multimodal-upgrade.patch` upgrades the KVMem working tree recorded before
the 2026-09-14 implementation to the same current code. The script checks applicability before
changing files. Unrelated local changes are preserved; conflicting changes
require review.

The numbered `0001` through `0004` files are historical patches, retained for
reference. They are superseded by the cumulative diff: the old series did
not cleanly replay on the current base and must not be applied together with it.

To check a clean extraction without changing the active submodule:

```bash
mkdir -p /tmp/kvmem-llama-patch-check
git -C llama.cpp archive 9a9394a | tar -x -C /tmp/kvmem-llama-patch-check
KVMEM_LLAMA_DIR=/tmp/kvmem-llama-patch-check scripts/apply-patches.sh
KVMEM_LLAMA_DIR=/tmp/kvmem-llama-patch-check scripts/apply-patches.sh
```
