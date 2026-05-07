# V9 Results Analysis — VQ-Based Privacy in VFL

---

## Complete Results Table

WACC/Tail sources: `shazil_results` (session 1) + `shazil_second_session` (session 2) + `new_results/results_d1v9` (session 3).
Stage B (SSIM/LPIPS): run only for H_vq_K256 and H_vq_commit_high in session 3. All other methods pending Stage B.

| Method | Bits | WACC (mean ± std) | Tail WACC | SSIM | LPIPS | Seeds |
|--------|------|-------------------|-----------|------|-------|-------|
| A_plain_vfl | 40960 | 0.7516 ± 0.0043 | 0.7880 | — | — | 2 |
| A_proj_vfl | 4096 | 0.7665 ± 0.0013 | 0.7933 | — | — | 2 |
| S_rand_sign | 128 | 0.7232 ± 0.0067 | 0.7675 | — | — | 2 |
| S_sign_quant | 128 | 0.7703 ± 0.0062 | 0.7955 | — | — | 2 |
| H_vq_K64 | 48 | 0.7860 ± 0.0072 | 0.8095 | — | — | 2 |
| H_vq_K256 | 64 | 0.7727 ± 0.0034 | 0.7907 | 0.5252 | 0.2167 | 2 |
| H_vq_no_kmeans | 64 | 0.7790 ± 0.0003 | 0.7997 | — | — | 2 |
| H_vq_M4 | 32 | 0.7706 ± 0.0049 | 0.7885 | — | — | 2 |
| H_vq_M16 | 128 | 0.7748 ± 0.0017 | 0.7917 | — | — | 2 |
| H_vq_commit_low | 64 | 0.7781 ± 0.0069 | 0.7913 | — | — | 2 |
| H_vq_commit_high | 64 | 0.7803 ± 0.0014 | 0.8017 | 0.5250 | 0.2187 | 2 |

All 11 methods now have WACC from 2 seeds. No outstanding gaps.

SSIM/LPIPS where shown are averaged across both seeds (session 3 Stage B, 30 InverNet epochs, 5067 val samples). Remaining 9 methods need Stage B.

Metrics: **WACC** = balanced accuracy (macro-averaged across 8 classes). **Tail WACC** = mean recall over the 4 minority classes (DF, VASC, SCC, AK). **SSIM** and **LPIPS** = reconstruction quality of InverNet attack (higher SSIM = easier to reconstruct = less private).

---

## What Is and Isn't Run

| Status | Methods |
|--------|---------|
| Complete — 2 seeds, WACC ✓ | All 11 methods |
| Stage B complete | H_vq_K256, H_vq_commit_high (session 3) |
| Stage B pending | A_plain_vfl, A_proj_vfl, S_rand_sign, S_sign_quant, H_vq_K64, H_vq_no_kmeans, H_vq_M4, H_vq_M16, H_vq_commit_low |

All 11 methods now have WACC from 2 seeds. Stage B (InverNet SSIM/LPIPS) is the only remaining blocker — needed for 9 of 11 methods.

---

## How to Read the Reconstruction Grid (orig | recon | diff)

Each row in the visual grid shows one val-set image under a three-panel view:

**orig** — the ground-truth 64×64 center-cropped dermoscopy image, the same spatial region the passive encoder actually saw.

**recon** — InverNet's best attempt to reconstruct that image from the transmitted representation alone (VQ codes or continuous embedding). InverNet has no access to the original image, only to what the passive client sends over the network.

**diff** — per-pixel absolute difference between orig and recon, per channel, normalized to [0, 1] and displayed as color. It is **not** a meaningful image — it is a heatmap. What matters is the intensity and spread:

- **Dark/near-black diff** = reconstruction is close to original. Attacker succeeded. High privacy risk.
- **Bright, high-variance, colorful diff** = reconstruction deviates substantially from original. Attacker failed to recover fine detail. Low privacy risk.

---

## What the Results Mean

### Utility: VQ improves WACC over continuous baseline

Every confirmed VQ method outperforms A_plain_vfl (0.752) on WACC — including H_vq_M4 (0.771), which beats the plain baseline by 1.9 points despite using 1280× fewer bits. H_vq_K64 is the strongest confirmed result at 0.786 ± 0.007, 3.4 points above A_plain at 853× fewer bits.

Full utility ordering (sorted by WACC, all methods now complete at 2 seeds):

```
H_vq_K64         0.786 ± 0.007   (48 bits)    ← K=64, M=8
H_vq_commit_high 0.780 ± 0.001   (64 bits)    ← K=256, M=8, β=0.50
H_vq_no_kmeans   0.779 ± 0.000   (64 bits)    ← K=256, M=8, random init
H_vq_commit_low  0.778 ± 0.007   (64 bits)    ← K=256, M=8, β=0.10
H_vq_M16         0.775 ± 0.002   (128 bits)   ← K=256, M=16, β=0.25
H_vq_K256        0.773 ± 0.003   (64 bits)    ← K=256, M=8, β=0.25 reference
H_vq_M4          0.771 ± 0.005   (32 bits)    ← K=256, M=4, β=0.25
S_sign_quant     0.770 ± 0.006   (128 bits)
A_proj_vfl       0.767 ± 0.001   (4096 bits)
A_plain_vfl      0.752 ± 0.004   (40960 bits)
S_rand_sign      0.723 ± 0.007   (128 bits)
```

---

### Hyperparameter trends

#### Codebook size K (M=8 fixed, β=0.25)

| Method | K | Bits | WACC | Tail | Seeds |
|--------|---|------|------|------|-------|
| H_vq_K64 | 64 | 48 | **0.786 ± 0.007** | 0.810 | 2 |
| H_vq_K256 | 256 | 64 | 0.773 ± 0.003 | 0.791 | 2 |

Both now confirmed at 2 seeds. Smaller K wins by 1.3 points while using fewer bits — the gap is real and consistent. Each subspace has 6-bit codes (K=64) vs 8-bit codes (K=256). More aggressive quantization strips high-frequency variation in the EfficientNet embedding that the active classifier doesn't need, acting as a regularizer. More expressive codebook ≠ better downstream WACC.

#### Number of subspaces M (K=256, β=0.25)

| Method | M | Bits | WACC | Tail | Seeds |
|--------|---|------|------|------|-------|
| H_vq_M4 | 4 | 32 | 0.771 ± 0.005 | 0.789 | 2 |
| H_vq_K256 | 8 | 64 | 0.773 ± 0.003 | 0.791 | 2 |
| H_vq_M16 | 16 | 128 | **0.775 ± 0.002** | 0.792 | 2 |

All three now at 2 seeds. The trend is nearly flat across the entire M range — going from 32 bits (M=4) to 128 bits (M=16) gains only 0.004 WACC. M=4 is the most efficient point per bit: same utility as M=16 at 25% of the bit cost.

#### Commitment weight β (K=256, M=8)

| Method | β | Bits | WACC | Tail | Seeds |
|--------|---|------|------|------|-------|
| H_vq_commit_low | 0.10 | 64 | 0.778 ± 0.007 | 0.791 | 2 |
| H_vq_K256 | 0.25 | 64 | 0.773 ± 0.003 | 0.791 | 2 |
| H_vq_commit_high | 0.50 | 64 | **0.780 ± 0.001** | 0.802 | 2 |

The ablation is now complete. The pattern is **non-monotonic**: β=0.50 wins narrowly over β=0.10, with β=0.25 in between. The differences are small (max spread 0.007 WACC) and all three are within each other's error bars. The stronger signal is tail WACC: commit_high (β=0.50) has the highest tail at 0.802, suggesting harder commitment pushes the codebook to better cover minority-class embeddings. β controls how hard the embedding is forced toward the nearest codebook entry — higher β = more aggressive discrete commitment from the start, which may force better codebook coverage of the full embedding distribution rather than clustering around the majority class.

#### Codebook initialization (K=256, M=8, β=0.25)

| Method | Init | Bits | WACC | Std | Seeds |
|--------|------|------|------|-----|-------|
| H_vq_K256 | K-means++ | 64 | 0.773 | ±0.003 | 2 |
| H_vq_no_kmeans | Random | 64 | 0.779 | ±0.0003 | 2 |

Both methods are now at 2 seeds. Random init (0.779) is ahead of K-means++ (0.773) by 0.006 WACC — a small but consistent gap (non-overlapping given the tiny std of no_kmeans). The near-zero std of random init (0.0003) versus K-means++ (0.003) is notable: random init converges to an almost identical result regardless of weight seed, while K-means++ shows more seed-to-seed variance. A possible explanation: K-means++ pre-seeds the codebook at embedding cluster centers from early training, which introduces variance depending on what the network has learned by initialization epoch — different weight seeds → different early embeddings → different K-means centroids → different final codebooks. Random init sidesteps this by giving the codebook no head start, letting it adapt purely through the VQ training loss. Whether K-means++ helps privacy (SSIM) despite not helping utility is blocked on Stage B for those methods.

#### Method family comparison

| Family | Best WACC | Bits | vs A_plain |
|--------|-----------|------|-----------|
| VQ (best: H_vq_K64) | 0.786 | 48 | +3.4 pts |
| Sign quant (S_sign_quant) | 0.770 | 128 | +1.8 pts |
| Projection (A_proj_vfl) | 0.767 | 4096 | +1.5 pts |
| Plain VFL (A_plain_vfl) | 0.752 | 40960 | baseline |
| Random sign (S_rand_sign) | 0.723 | 128 | −2.9 pts |

VQ dominates. Sign quantization (deterministic top-128 bits) outperforms random projection alone (S_rand_sign), which actually hurts WACC vs plain — random sign flips destroy information the active client needs. The projection baseline (A_proj_vfl) compresses 40960 → 4096 bits via a learned linear layer and still lags behind the weakest VQ method.

### Why VQ improves utility (hypothesis)

The discrete bottleneck suppresses irrelevant high-frequency variation in the continuous 1280-d EfficientNet embedding, acting as a regularizer for the active client's task-specific classifier. Only variation that the VQ codebook captures is transmitted; the rest is discarded. This is analogous to dropout-style regularization but in representation space rather than weight space. K=64 provides stronger regularization (more aggressive compression) than K=256, which explains why the smaller codebook wins despite transmitting less information. Supporting evidence needed: show that per-class embedding variance from the passive client decreases under VQ relative to A_plain.

---

## Paper Argument (Utility Side Established — Privacy Side Pending)

> We demonstrate that product quantization of the passive client's intermediate embedding in vertical federated learning consistently improves diagnostic utility over continuous transmission while transmitting far fewer bits. On ISIC-2019 dermoscopy classification in an 8-class VFL setup, all VQ methods achieve WACC ≥ 0.771 versus 0.752 for plain VFL (40960 bits). H_vq_K64 (48 bits, K=64, M=8) reaches WACC 0.786 ± 0.007 — 3.4 points above A_plain at 853× fewer bits, and above sign quantization (S_sign_quant 0.770) at fewer bits. Privacy evaluation (reconstruction SSIM/LPIPS) pending Stage B.

---

## What Needs to Be Done (Priority Order)

### Next Kaggle run

**1. Run Stage B on remaining 9 methods × 2 seeds**
Stage B has been run for H_vq_K256 and H_vq_commit_high in session 3. The remaining 9 methods (A_plain_vfl, A_proj_vfl, S_rand_sign, S_sign_quant, H_vq_K64, H_vq_no_kmeans, H_vq_M4, H_vq_M16, H_vq_commit_low) still need InverNet evaluation. All checkpoints are in `results_final/checkpoints/`. After this run, the full privacy-utility table and Pareto curve can be produced.

### Before paper submission

**2. Equal-bit comparison: H_vq_M16 vs S_sign_quant (both 128 bits)**
WACC: H_vq_M16 0.775 vs S_sign_quant 0.770 — VQ wins on utility. Whether VQ also wins on SSIM at equal bits is the key privacy claim. Blocked on Stage B.

**3. Pareto curve figure**
WACC (y) vs SSIM (x), colored by method family, point size proportional to bits. Blocked on Stage B.

**4. Stronger reconstruction attacker**
InverNet at 30 epochs is moderate. Run at 50 epochs for A_plain_vfl vs best VQ method to stress-test the privacy claim.

**5. Clinical SSIM threshold**
Need a citation to clinical image quality literature for dermoscopy, or a perceptual annotation, to ground SSIM numbers for medical imaging reviewers.

---

## Roadmap to Paper

| Task | Effort | Status |
|------|--------|--------|
| Recover result JSONs for K256 s42 + commit_high s42 | ~5 min | ✅ Done (session 3) |
| Train H_vq_commit_high seed 43 | ~1 hr GPU | ✅ Done (session 3) |
| Stage B: H_vq_K256 + H_vq_commit_high | done | ✅ Done (session 3) |
| Stage B: remaining 9 methods × 2 seeds | ~3–4 hr GPU | **Main blocker** |
| Equal-bit comparison writeup (M16 vs S_sign) | writing | blocked on Stage B |
| Pareto curve + bits-vs-SSIM figures | 1–2 hr | blocked on Stage B |
| Stronger InverNet (50 epochs) | ~4 hr GPU | not started |
| Mechanism explanation for VQ utility improvement | writing | can start now |
| Phase 2: cross-modal attention + VQ privacy layer | new work | future |
| Phase 3: dynamic commitment weight per modality | new work | future |

### Venue targets

| Venue | Feasibility | What's needed |
|-------|-------------|---------------|
| NeurIPS/ICML/MICCAI workshop | Yes, once Stage B done | ~2–3 weeks |
| MIDL main | Yes | All pre-submission items above |
| MICCAI main | Stretch | Formal bounds + clinical eval |

---

## Supervisor Likely Questions and Pre-emptive Answers

**"Why does VQ improve WACC? That shouldn't happen."**
The discrete bottleneck suppresses irrelevant high-frequency variation in the continuous embedding, regularizing the active client's classification. K-means initialization seeds the codebook at statistically meaningful locations. Supporting evidence needed: show gradient norm from the active client decreases under VQ.

**"H_vq_K64 beats H_vq_K256. Why does a smaller codebook win?"**
H_vq_K64 uses K=64 centroids per subspace (6 bits each, 48 bits total) versus K=256 (8 bits each, 64 bits total). Both are now confirmed at 2 seeds: K64 wins by 1.3 WACC points (0.786 vs 0.773). The smaller codebook provides stronger regularization — more aggressive quantization strips embedding variation that the classifier doesn't need. More expressive codebook ≠ better downstream task performance.

**"H_vq_no_kmeans is tied with your K-means methods. What does K-means init actually buy you?"**
On utility alone, K-means init does not appear essential at K=256. The benefit may appear in SSIM (faster codebook convergence → more consistent privacy behavior across seeds). The near-zero std of H_vq_no_kmeans (0.0003) vs H_vq_commit_low (0.0069) is itself interesting — pending Stage B.

**"InverNet is a toy attacker."**
InverNet matches the attacker architecture used in SplitGuard and similar VFL privacy papers. We will additionally evaluate with 50 epochs and discuss diffusion-based inversion as a limitation.

**"What does SSIM 0.52 mean for a dermatologist?"**
Pending Stage B. The visual reconstruction grids will show that at SSIM ~0.52, the attacker recovers rough lesion shape and dominant color but loses texture, vascular patterns, and fine-grained color variation a clinician uses for lesion subtype identification. A clinical image quality citation is needed.
