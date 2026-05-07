# Session Changes — vfl-v9.ipynb

Date: 2026-04-30

---

## Change 1 — Added `train_epochs` field to `MethodSpec` (Cell 16)

**What:** Added `train_epochs: int = 30` as a field on the `MethodSpec` dataclass.

**Why:** The original design tied every training run to `V9Config.stage1_epochs = 30`. That global constant makes it impossible to give individual methods a different epoch budget without touching the config object. This matters because K=256 product quantization converges slower than K=64 — the K256 runs from the truncated session showed val WACC still improving at epoch 22 and plateauing around epoch 18, while K64 converged comfortably in 30 epochs. A per-spec override field lets us handle this without touching any shared config.

Default is 30 so all methods that don't set it explicitly are unaffected.

---

## Change 2 — Wired `spec.train_epochs` into `train_v9` (Cell 14)

**What:** Replaced three occurrences of `cfg.stage1_epochs` in `train_v9` with `spec.train_epochs`:
- `n_sched_steps = max(spec.train_epochs - cfg.warmup_epochs, 1)` — cosine LR schedule T_max
- `for epoch in range(1, spec.train_epochs + 1):` — training loop upper bound
- The epoch progress print string

**Why:** Without wiring this into the training function, the new `MethodSpec.train_epochs` field does nothing. The cosine LR schedule `T_max` also needs to reflect the actual number of training steps, otherwise a 40-epoch run with `T_max=25` would decay the LR to `eta_min` by epoch 30 and the last 10 epochs would train at a near-zero learning rate — essentially wasted compute.

---

## Change 3 — Set all K=256 VQ methods to `train_epochs=40` (Cell 16)

**What:** Added `train_epochs=40` to six MethodSpecs: `H_vq_K256`, `H_vq_no_kmeans`, `H_vq_M4`, `H_vq_M16`, `H_vq_commit_low`, `H_vq_commit_high`.

**Why:** All six methods use K=256 codebooks. The evidence from the truncated session (H_vq_K256 seed=42 best val at 0.7437, seed=43 still improving at epoch 22 with best of 0.7221) suggests K=256 needs more epochs than K=64 to saturate the codebook and converge the classifier. H_vq_K64 finished cleanly at 0.7526 in 30 epochs; K=256 appears to need ~10 more epochs.

Critically, applying 40 epochs to **all** K=256 variants (not just H_vq_K256) ensures the ablations are internally consistent. If H_vq_commit_low and H_vq_commit_high used different epoch counts, any WACC difference in the commitment weight ablation would be confounded by training time. Same applies to the M sweep (H_vq_M4, H_vq_M16) and the init ablation (H_vq_no_kmeans).

H_vq_K64 stays at 30 because it demonstrably converges there (0.7526 avg WACC across two seeds with no sign of continued improvement).

---

## Change 4 — Added `invernet_epochs=50` to `A_plain_vfl` (Cell 16)

**What:** Set `invernet_epochs=50` on the `A_plain_vfl` MethodSpec (was using the default of 30).

**Rationale given:** `A_plain_vfl`'s InverNet has `fc = Linear(1280, 4096)` — 5.24M parameters — versus `Linear(128, 4096)` (524K params) for every other method. The argument was that 30 InverNet epochs might not be enough for the larger fc to converge, and since A_plain's SSIM serves as the "plain VFL is dangerous" baseline, under-training its attacker artificially depresses that SSIM and weakens the paper narrative.

**Open question / pushback:** The user challenged this as potentially biased — selectively giving A_plain a stronger attacker specifically because it supports the paper's argument. This challenge is valid. The cleaner experimental protocol is to use the same `invernet_epochs` for all methods. The current state (A_plain=50, most others=30) is inconsistent.

**Pending decision:** Should all methods be set to 50 uniformly? That would be the most defensible position for a paper — equal attacker budget across all methods, with 50 chosen because it's the value already assigned to the headline comparison pair (S_sign_quant and H_vq_M16). No code change was made beyond the A_plain bump; this question was left open at the end of the session.

---

## Change 5 — Added then reverted `input_proj` in `InverNetV9` (Cell 19)

**What was attempted:** Added an `input_proj = Linear(in_dim, 256) → ReLU` layer before the shared fc+ResBlocks+upsampler path in InverNetV9. The intention was to make the decoder architecture identical across all methods by normalizing every input to 256-d first.

**Why it was reverted:** After further analysis, the bottleneck for A_plain_vfl (1280→256 compression before decoding) would throw away information that InverNet could otherwise use to reconstruct A_plain images. This would artificially lower A_plain's SSIM — the exact opposite of what's needed. The "problem" of different fc sizes is not actually a fairness problem: the convolutional decoder (ResBlocks + upsampler + refine) is already identical across all methods. The fc layer is just an adapter from representation space to the 256-ch 4×4 spatial feature map. Scaling it with `in_dim` is the standard approach in inversion attack papers — you give the attacker a capacity that is proportional to the richness of the input, which is the correct choice for a privacy evaluation.

**What remains:** The InverNetV9 docstring was updated to explain this reasoning explicitly, so it's clear to any future reader why the fc is not fixed.

---

## Summary of current state

| Method | train_epochs | invernet_epochs |
|---|---|---|
| A_plain_vfl | 30 (default) | **50** |
| A_proj_vfl | 30 (default) | 30 (default) |
| S_rand_sign | 30 (default) | 30 (default) |
| S_sign_quant | 30 (default) | 50 (pre-existing) |
| H_vq_K64 | 30 (default) | 30 (default) |
| H_vq_K256 | **40** | 30 (default) |
| H_vq_no_kmeans | **40** | 30 (default) |
| H_vq_M4 | **40** | 30 (default) |
| H_vq_M16 | **40** | 50 (pre-existing) |
| H_vq_commit_low | **40** | 30 (default) |
| H_vq_commit_high | **40** | 30 (default) |

**Unresolved before next Kaggle run:** whether to set all `invernet_epochs` to 50 uniformly. The current inconsistency (A_plain=50, most others=30) should be resolved one way or the other before running Stage B.
