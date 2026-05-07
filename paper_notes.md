# Paper Notes — VQ-Based Privacy in VFL

Working notes for writing the paper. Add to this as new decisions are made.

---

## Reconstruction Attacker — Threat Model and Framing

### What we use: InverNetV9

Architecture:
```
FC(in_dim → 256×4×4) → ResBlock×2 → ConvTranspose×4 (4→8→16→32→64px) → Refine conv → Tanh
Loss: MSE + 0.1·LPIPS
Epochs: 50 (uniform across all methods)
Output: 64×64×3
```

The fc layer scales with in_dim — A_plain_vfl gets a larger attacker (Linear(1280,4096) = 5.24M params) than VQ methods (Linear(128,4096) = 524K params). This is intentional and correct: give the attacker capacity proportional to the richness of the input. Fixing fc size across methods would artificially disadvantage A_plain_vfl's InverNet, understating plain VFL's privacy risk.

### Threat model

Passive eavesdropping (honest-but-curious active client). Attacker observes all transmitted representations during inference. Does NOT:
- Participate in training
- Manipulate gradients
- Require label access
- Know the passive client's model weights

This is the standard threat model for evaluating intermediate representation privacy in VFL/split learning.

### Citation: UIFV not InverNet

Cite UIFV (arXiv 2406.12588, Yang et al. 2024) to justify the threat model. Do NOT call our implementation "UIFV" — the architectures differ:

| | UIFV image decoder | Our InverNetV9 |
|--|--|--|
| Architecture | 2-layer ConvTranspose | FC + 2 ResBlocks + 4 ConvTranspose + Refine |
| Loss | MSE only | MSE + 0.1·LPIPS |
| Output resolution | 32×32 (CIFAR-10) | 64×64 (ISIC-2019) |
| Epochs | Unspecified | 50 |

Our attacker is stronger than UIFV's published baseline. This makes our privacy evaluation more conservative — a reviewer asking "why not UIFV?" gets: our InverNetV9 is an enhanced version of the same paradigm, giving the attacker more capacity.

Paper line: *"Following the passive inversion threat model of UIFV [cite], we evaluate each method against InverNetV9 — a residual CNN decoder trained with MSE + LPIPS loss, a stronger architecture than UIFV's baseline decoder."*

### Why not URVFL

URVFL (Hu et al., NDSS 2025, arXiv 2404.19582) is a different threat model: active malicious participant, controls top model + labels, injects gradient-spoofed updates designed to evade detection (SplitGuard, Gradient Scrutinizer). Our defense (VQ bottleneck) targets information reduction in transmitted embeddings — a defense against passive reconstruction, not gradient-based attacks. Evaluating against URVFL would be testing a different threat than the one we defend against. Mention as a limitation / out-of-scope.

---

## Novelty Claims

Based on literature search (May 2026):

**Primary claim (defensible):** The specific combination of (1) product quantization at the passive client communication boundary in VFL, (2) empirical utility improvement over continuous transmission, and (3) SSIM/LPIPS reconstruction attack evaluation is not present in prior work.

**Supporting claims:**
- Prior VQ-in-FL work (FedMPQ, FedVQCS) treats VQ as communication compression only, not as a privacy mechanism. No reconstruction attack evaluation in those papers.
- FedVQCS (arXiv 2204.07692) shows VQ can improve utility in horizontal FL — parallel finding, different setting (horizontal FL, no privacy evaluation).
- Randomized quantization for DP (arXiv 2306.11913) uses discrete Gaussian sampling, different mechanism, no reconstruction attack eval.

**Risk: ESANN 2024 paper** — "About Vector Quantization and its Privacy in Federated Learning" (ES2024-57). PDF could not be fully retrieved. Must obtain and read before submission — this is the closest potential prior work.

**Strongest novelty angle:** VQ improves utility while being expected to reduce reconstruction fidelity. No prior work claims both simultaneously in VFL.

---

## Key Results (for paper tables/figures)

All 11 methods, 2 seeds each:

| Method | Bits | WACC | SSIM |
|--------|------|------|------|
| A_plain_vfl | 40960 | 0.752 ± 0.004 | pending |
| A_proj_vfl | 4096 | 0.767 ± 0.001 | pending |
| S_rand_sign | 128 | 0.723 ± 0.007 | pending |
| S_sign_quant | 128 | 0.770 ± 0.006 | pending |
| H_vq_K64 | 48 | 0.786 ± 0.007 | pending |
| H_vq_K256 | 64 | 0.773 ± 0.003 | 0.5252 |
| H_vq_no_kmeans | 64 | 0.779 ± 0.000 | pending |
| H_vq_M4 | 32 | 0.771 ± 0.005 | pending |
| H_vq_M16 | 128 | 0.775 ± 0.002 | pending |
| H_vq_commit_low | 64 | 0.778 ± 0.007 | pending |
| H_vq_commit_high | 64 | 0.780 ± 0.001 | 0.5250 |

SSIM values above are from 30-epoch runs and will be replaced with 50-epoch runs.

**Headline numbers:**
- H_vq_K64: +3.4 WACC over A_plain, 853× fewer bits
- H_vq_M4 (32 bits): +1.9 WACC over A_plain at 1280× fewer bits
- Equal-bit comparison: H_vq_M16 (0.775) vs S_sign_quant (0.770) at 128 bits — VQ wins on utility; pending SSIM comparison

---

## Method Section Notes

**What to call our approach:** "Product Quantization bottleneck for VFL privacy" or "VQ-based communication bottleneck." Avoid calling it "InverNet" in the method section — that's the attacker.

**VQ layer description:** The passive client's 1280-d EfficientNet-B0 embedding is split into M equal subspaces of 1280/M dimensions. Each subspace is independently quantized to the nearest entry in a learned codebook of K vectors, trained jointly with straight-through gradients. Only the M codebook indices are transmitted (M·log₂K bits). The active client reconstructs the embedding by codebook lookup before classification.

**Straight-through estimator:** Required because nearest-neighbor lookup is not differentiable. Gradients pass through as if quantization didn't happen (standard VQ-VAE approach, van den Oord et al. 2017).

**K-means++ init:** At epoch 4, run mini-batch K-means on current embeddings to initialize codebook entries at statistically meaningful starting points. Tested against random initialization (H_vq_no_kmeans).

---

## Hyperparameter Findings (for ablation section)

- **K (codebook size):** K=64 beats K=256 by 1.3 WACC points using fewer bits. More aggressive quantization = stronger regularizer. Non-obvious finding.
- **M (subspaces):** Nearly flat across M=4 to M=16 (0.771 → 0.773 → 0.775). M=4 is most efficient per bit.
- **β (commitment weight):** Non-monotonic at utility level (max spread 0.007 WACC). Clearest signal is tail WACC: β=0.50 leads (0.802), suggesting harder commitment improves minority-class coverage.
- **Init:** Random init (0.779) slightly ahead of K-means++ (0.773) on utility, but much more stable (std 0.0003 vs 0.003). May not matter for privacy — pending Stage B.

---

## Paper Structure (draft outline)

1. Introduction — VFL privacy threat, embedding inversion problem
2. Related Work — VFL privacy attacks (UIFV, URVFL), VQ in FL (FedMPQ, FedVQCS), split learning privacy
3. Method — VFL setup, VQ bottleneck, InverNetV9 attacker
4. Experiments — Dataset (ISIC-2019), baselines (A_plain, A_proj, sign methods), VQ ablations
5. Results — Utility table, SSIM table, Pareto curve, reconstruction grids
6. Discussion — Why VQ improves utility, privacy mechanism, limitations (URVFL out of scope, DRAG as stronger attacker)
7. Conclusion

---

## Pending Before Writing

- [ ] Stage B (50-epoch InverNet) for all 11 methods × 2 seeds
- [ ] Obtain and read ESANN 2024 ES2024-57 (novelty risk)
- [ ] Pareto curve (WACC vs SSIM, colored by family)
- [ ] Reconstruction grid figure (orig | recon | diff for best vs worst VQ method vs A_plain)
- [ ] Equal-bit comparison writeup (H_vq_M16 vs S_sign_quant at 128 bits)
- [ ] Clinical SSIM citation for dermoscopy image quality

---

## Venue Targets

| Venue | Feasibility | Notes |
|-------|-------------|-------|
| MICCAI workshop / MIDL workshop | Yes, once Stage B done | ~2-3 weeks |
| MIDL main | Yes | All above items |
| MICCAI main | Stretch | Formal privacy bounds + clinical eval needed |
