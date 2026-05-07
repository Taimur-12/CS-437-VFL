# V9 Notebook Plan — VQ-Only VFL Privacy Study

---

## Key Decisions Going In

- **No GRL.** Drop `V8LabelAdversary`, `GradReverse`, `grad_reverse`, `dann_lambda` entirely.
- **No label inference attack.** Only privacy metric is reconstruction: SSIM + LPIPS.
- **Train everything from scratch.** No checkpoint loading from v6/v7/v8. Fully self-contained notebook.
- **LPIPS assumed available.** No fallback needed.
- **Dynamic commitment weight deferred** to Phase 3 (year-long project). Ablated at 3 fixed values here to establish the tradeoff direction.
- **Cross-attention fusion deferred** to Phase 2. VQ story first.

---

## S_sign_quant Status

Was trained in v7 (not v6, not v8). v8 listed it as a baseline loaded from v7 JSON artefacts — but v7 artefacts were missing at Kaggle run time, so SSIM/LR-AUC were N/A in the final table. It has never had a privacy eval. The new notebook trains it fresh.

---

## What Gets Dropped vs. v8

- `V8LabelAdversary`, `GradReverse`, `grad_reverse`, `dann_lambda` — all GRL plumbing
- `usage_entropy_loss` still **kept** (prevents codebook collapse, label-free, free compute)
- All baseline loading from v6/v7 JSON artefacts — replaced by training everything fresh
- Label inference evaluation (LR + MLP attack) — dropped entirely

---

## Section-by-Section Plan

### §1 — Setup & Config

Reuse `V8Config` dataclass from v8. Remove `grl_*` and `adv_*` fields. Add:
- `vq_commitment_weight` (swept in ablations)
- `vq_num_subspaces` (swept in ablations)

Kaggle paths reused verbatim (same dataset slugs):
```
IMG_DIR    = "/kaggle/input/datasets/andrewmvd/isic-2019/ISIC_2019_Training_Input/..."
GT_CSV     = "/kaggle/input/datasets/andrewmvd/isic-2019/ISIC_2019_Training_GroundTruth.csv"
META_CSV   = "/kaggle/input/datasets/andrewmvd/isic-2019/ISIC_2019_Training_Metadata.csv"
CACHE_PATH = "/kaggle/input/datasets/taimurjahanzeb/isic-2019-256-cache/images_256.npy"
FOLD_PATH  = "/kaggle/input/datasets/taimurjahanzeb/isic-2019-256-cache/fold_indices.npz"
RESULTS_DIR = "/kaggle/working/results_d1v9"
```

### §2 — Data Loading

**Reuse entirely from v6-b:** `ISICDataset`, `make_loaders`, all transforms (train augmentation + val center crop), meta processing (age bins, site one-hot, sex), class weights, fold loading. Most stable code in both notebooks.

### §3 — Model Definitions

**Reuse from v8 (stable V8* names, strict=True loading):**
- `V8Backbone` — EfficientNet-B0, no change
- `V8Projection` — linear 1280→128, no change
- `V8ProductQuantizer` — keep K-means++ init, `reset_usage()`, dead-code tracking. Drop nothing.
- `V8ActiveClient` — meta MLP + classifier, no change

**New in v9:**
- `VQPassiveClient`: backbone + projection + quantizer, no adversary. Simpler than v8.
- `SignQuantPassiveClient`: backbone → projection → `sign(z)` → transmitted as `{±1}^128`. STE for backward pass:
  ```python
  transmitted = z.sign()
  transmitted_ste = z + (transmitted - z).detach()  # STE
  # comm_bits = 128
  ```
  Expected WACC ~0.7579 (validates against known number).
- `ContinuousPassiveClient` variants: 1280-d (A_plain_vfl) and 128-d projection-only (A_proj_vfl). Reused from v6-b.

### §4 — Training Loop

Reuse `train_v8_main` structure from v8, strip all adversary paths. Loss:

```
L = L_task + λ_vq · L_vq + λ_u · L_usage_entropy
```

No `opt_adv`, no `adversary.train()`. ~30% fewer lines than v8's train function.

**Keep from v8:**
- K-means++ init warmup
- VQ curriculum (continuous → discrete warmup)
- Cosine LR schedule
- Early stopping on WACC
- Per-epoch history JSON
- Checkpoint saving with `strict=True` verify
- Dead-code count logging per epoch (% of codebook used)

### §5 — Ablation Grid

**9 methods × 2 seeds = 18 training runs. Everything trains from scratch.**

| Method | K | M | β_commit | Bits | What it tests |
|---|---|---|---|---|---|
| A_plain_vfl | — | — | — | 40960 | continuous baseline |
| A_proj_vfl | — | — | — | 4096 | projection-only (is privacy from dim reduction alone?) |
| S_sign_quant | — | 8 | — | 128 | trivial 1-bit coding at 128 bits |
| H_vq_K64 | 64 | 8 | 0.25 | 48 | K sweep, low end |
| H_vq_K256 | 256 | 8 | 0.25 | 64 | **main VQ method** |
| H_vq_M4 | 256 | 4 | 0.25 | 32 | fewer subspaces, fewer bits |
| H_vq_M16 | 256 | 16 | 0.25 | 128 | **same bits as S_sign — apples-to-apples** |
| H_vq_commit_low | 256 | 8 | 0.1 | 64 | soft commitment (less discrete) |
| H_vq_commit_high | 256 | 8 | 0.5 | 64 | hard commitment (more discrete) |

**H_vq_K1024 dropped** — v6 results showed K=256→1024 was negligible (WACC 0.7567 vs 0.7598, SSIM barely changed). Saves 2 seeds of compute.

**The money comparison: H_vq_M16 vs S_sign_quant** — both exactly 128 bits. One learned VQ coding, one naive sign. If VQ wins on SSIM at the same bit budget, that's the core privacy claim proven cleanly. If it doesn't, that's a clear negative result that needs explaining.

**What was missing from prior notebooks that this grid adds:**
1. `A_proj_vfl` SSIM — never measured. Critical control: if projection alone gives low SSIM, dim reduction is doing the privacy work and VQ isn't adding much.
2. Same-bit VQ vs sign comparison — H_vq_M16 (128 bits, learned) vs S_sign (128 bits, naive).
3. Commitment weight ablation — β ∈ {0.1, 0.25, 0.5} establishes the tradeoff direction for dynamic temperature (Phase 3).

### §6 — Reconstruction Privacy Evaluation

**Run on all 9 ablations** (v8 only ran reconstruction on V8_main — this was the critical gap).

**Improved InverNet (V9):**

Current v8 InverNet: fc → 4× ConvTranspose+BN+ReLU → Tanh. 20 epochs.

Improvements:
- Add 2 residual blocks at bottleneck (256 channels, before upsampling)
- Add final refinement conv layer at 64×64 (smooths artifacts)
- Loss: `L = L_MSE + 0.1 · L_LPIPS` (LPIPS assumed available)
- Increase to 30 epochs
- Adam LR 1e-3 with cosine decay

```python
class InverNetV9(nn.Module):
    def __init__(self, in_dim, out_res=64):
        super().__init__()
        self.fc = nn.Linear(in_dim, 256 * 4 * 4)
        self.res_blocks = nn.Sequential(ResBlock(256), ResBlock(256))
        self.upsampler = nn.Sequential(
            nn.ConvTranspose2d(256, 128, 4, 2, 1), nn.BatchNorm2d(128), nn.ReLU(True),  # 8x8
            nn.ConvTranspose2d(128,  64, 4, 2, 1), nn.BatchNorm2d(64),  nn.ReLU(True),  # 16x16
            nn.ConvTranspose2d( 64,  32, 4, 2, 1), nn.BatchNorm2d(32),  nn.ReLU(True),  # 32x32
            nn.ConvTranspose2d( 32,   3, 4, 2, 1),                                       # 64x64
        )
        self.refine = nn.Sequential(
            nn.Conv2d(3, 16, 3, 1, 1), nn.ReLU(True),
            nn.Conv2d(16, 3, 3, 1, 1), nn.Tanh()
        )
    def forward(self, x):
        h = self.fc(x).view(-1, 256, 4, 4)
        h = self.res_blocks(h)
        h = self.upsampler(h)
        return self.refine(h)
```

**Codebook utilization:** Keep `reset_usage()` and log dead-code % per epoch from `V8ProductQuantizer`. Include in per-epoch history JSON.

### §7 — Visual Reconstruction Output

After InverNet training for each method, save a visualization:
- **6 sample images** chosen to cover at least 2 tail classes (DF, VASC)
- For each sample: `[original 64×64 | reconstructed | pixel difference map]`
- Save one PNG per method, then combine into a single multi-row comparison figure
- Makes privacy argument tangible — supervisor and reviewers can see whether lesion type is recoverable from the reconstruction

### §8 — Results & Plotting

- Full combined table: WACC, tail WACC, SSIM (mean±std across seeds), LPIPS, bits
- **Pareto curve**: WACC vs SSIM across all methods, colored by method type — this is the headline figure
- Bits vs SSIM and bits vs WACC plots separately
- Save all as PNG

---

## Code Reuse Summary

| What | From which notebook |
|---|---|
| Data loading, ISICDataset, transforms, meta processing | v6-b |
| Fold loading, class weights | v6-b |
| V8Backbone, V8Projection, V8ProductQuantizer | v8 |
| V8ActiveClient | v8 |
| K-means++ init, reset_usage, dead-code tracking | v8 (V8ProductQuantizer) |
| usage_entropy_loss | v8 |
| save_metrics, verify_checkpoint_load | v8 |
| reconstruction_attack structure | v8 (modified: all methods, not just main) |
| Kaggle dataset paths | v6-b / v8 (identical) |

| What | New in v9 |
|---|---|
| VQPassiveClient (no GRL) | new |
| SignQuantPassiveClient | new |
| ContinuousPassiveClient variants | adapted from v6-b |
| InverNetV9 with residual blocks | new |
| Visual reconstruction grid output | new |
| M sweep ablations (H_vq_M4, H_vq_M16) | new |
| Commitment weight ablations | new |
| Reconstruction eval on all methods | new |
| Pareto curve plotting | new |

---

## What the Results Need to Show (Paper Story)

**Chain to establish:**
```
A_plain_vfl (SSIM ~0.75) → A_proj_vfl (SSIM ?) → S_sign_quant (SSIM ?) → H_vq_K256 (SSIM ~0.45)
```
- If A_proj SSIM ≈ A_plain: projection doesn't help privacy → discretization is what matters
- If S_sign SSIM ≈ H_vq at same bits: VQ's learned codebook adds nothing over naive sign → negative result
- If H_vq_M16 SSIM < S_sign SSIM at same 128 bits: VQ coding is better than sign coding → positive result

**Commitment story (for Phase 3 pitch):**
```
H_vq_commit_low (β=0.1, soft) → H_vq_K256 (β=0.25) → H_vq_commit_high (β=0.5, hard)
```
Shows SSIM goes down (better privacy) as β goes up, with WACC cost. Motivates: "the optimal β is a function of the modality's data distribution — Phase 3 makes it dynamic."

---

## Venue Target with These Results

| Venue | Realistic with v9 results |
|---|---|
| MIDL | Yes — empirical + clean framing sufficient |
| MICCAI workshop | Yes — current scope matches |
| NeurIPS/ICML workshop | Yes |
| MICCAI main | Needs stronger attacker + clinical SSIM threshold |
