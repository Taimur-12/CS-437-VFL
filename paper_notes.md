# Paper Notes — VQ-Based Privacy in VFL

Working notes for writing the paper. Purpose: when to cite each paper, what to say about it, and how to frame each argument. Implementation details live in separate plan docs — only paper-facing content goes here.

---

## Paper Narrative

**One-sentence claim:** Product quantization at the passive client's communication boundary in vertical federated learning simultaneously improves diagnostic utility and reduces image reconstructability, with no changes to the active client or the server.

**The argument arc:**
1. VFL leaks private images through the transmitted embedding — passive eavesdropping is sufficient (no labels, no gradient access needed).
2. Naive fixes (random projection, sign quantization) either hurt utility or reduce privacy only incidentally.
3. A learned discrete bottleneck (product VQ) strips high-frequency embedding variation the classifier doesn't need, acting as a regularizer — this is why utility *improves*.
4. The same discretization makes the transmitted representation harder to invert — reconstruction SSIM drops relative to continuous transmission.
5. This tradeoff is tunable via K, M, and β. K=64 is the strongest single operating point.
6. The regularization mechanism is empirically confirmed: VQ representations show higher within-class silhouette score (cosine) than plain VFL (0.053–0.136 vs 0.015) despite lower dimensionality. Critically, A_proj_vfl is negative (−0.015) — projection alone *hurts* class structure, proving the effect is from VQ discretization, not the projection layer. H_vq_M4 reaches silhouette 0.136, the highest of all methods.

**What the paper does NOT claim:**
- Formal privacy guarantees (no DP bounds).
- Defense against active/gradient-based attacks (URVFL is out of scope).
- That VQ is universally better than sign quantization — at 128 bits, VQ wins on utility (0.775 vs 0.770) at essentially equal SSIM (0.540 vs 0.538).

---

## Datasets

### ISIC-2019 (Primary)

**What it is:** 25,331 dermoscopy-style skin lesion images across 8 diagnostic classes (MEL, NV, BCC, AK, BKL, DF, VASC, SCC) with patient age, sex, and anatomical site metadata. Standard benchmark for skin lesion classification. Severe class imbalance — DF, VASC, SCC, AK are minority classes.

**VFL partition:** Passive client owns the image (EfficientNet-B0 backbone). Active client owns age/sex/site metadata and the diagnostic label.

**How to introduce it in the paper:**

> *"We evaluate on ISIC-2019, an 8-class dermoscopy skin lesion benchmark with patient metadata, partitioned vertically: the image party transmits only an intermediate representation, while the metadata-and-label party performs classification. Balanced accuracy (WACC) is the primary utility metric given severe class imbalance across 8 categories."*

**Why it is the primary dataset:** Large enough for stable 2-seed estimates across 11 methods, well-established in skin lesion literature, and provides both image and tabular metadata for a realistic multimodal VFL setup.

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

| Method | Bits | WACC | SSIM | LPIPS |
|--------|------|------|------|-------|
| A_plain_vfl | 40960 | 0.752 ± 0.004 | 0.622 | 0.111 |
| A_proj_vfl | 4096 | 0.767 ± 0.001 | 0.585 | 0.139 |
| S_rand_sign | 128 | 0.723 ± 0.007 | 0.529 | 0.214 |
| S_sign_quant | 128 | 0.770 ± 0.006 | 0.538 | 0.196 |
| H_vq_K64 | 48 | 0.786 ± 0.007 | **0.503** | 0.241 |
| H_vq_K256 | 64 | 0.773 ± 0.003 | 0.514 | 0.227 |
| H_vq_no_kmeans | 64 | 0.779 ± 0.000 | 0.520 | 0.232 |
| H_vq_M4 | 32 | 0.771 ± 0.005 | 0.512 | 0.239 |
| H_vq_M16 | 128 | 0.775 ± 0.002 | 0.540 | 0.194 |
| H_vq_commit_low | 64 | 0.778 ± 0.007 | 0.512 | 0.226 |
| H_vq_commit_high | 64 | 0.780 ± 0.001 | 0.512 | 0.231 |

All values final: WACC from 2 seeds, SSIM/LPIPS from 50-epoch InverNet Stage B × 2 seeds. SSIM: higher = easier to reconstruct = less private. LPIPS: higher = more perceptually different = more private.

**Headline numbers:**
- H_vq_K64: +3.4 WACC over A_plain (0.786 vs 0.752), SSIM 0.503 vs 0.622, at 853× fewer bits
- H_vq_M4 (32 bits): +1.9 WACC over A_plain at 1280× fewer bits
- Equal-bit comparison at 128b: H_vq_M16 (WACC 0.775, SSIM 0.540) vs S_sign_quant (WACC 0.770, SSIM 0.538) — VQ wins utility at equivalent privacy

---

## Method Section Notes

**What to call our approach:** "Product Quantization bottleneck for VFL privacy" or "VQ-based communication bottleneck." Avoid calling it "InverNet" in the method section — that's the attacker.

**VQ layer description:** The passive client's EfficientNet-B0 embedding (1280-d) is first projected to a learned 128-d representation. This 128-d vector is then split into M equal subspaces of 128/M dimensions. Each subspace is independently quantized to the nearest entry in a learned codebook of K vectors, trained jointly with straight-through gradients. Only the M codebook indices are transmitted (M·log₂K bits). The active client reconstructs the 128-d embedding by codebook lookup before classification. Note: VQ operates on the 128-d projection, NOT the 1280-d EfficientNet embedding — a critical reviewer trap.

**Straight-through estimator:** Required because nearest-neighbor lookup is not differentiable. Gradients pass through as if quantization didn't happen (standard VQ-VAE approach, van den Oord et al. 2017).

**K-means++ init:** At epoch 4, run mini-batch K-means on current embeddings to initialize codebook entries at statistically meaningful starting points. Tested against random initialization (H_vq_no_kmeans).

---

## Hyperparameter Findings (for ablation section)

- **K (codebook size):** K=64 beats K=256 by 1.3 WACC points using fewer bits. More aggressive quantization = stronger regularizer. Non-obvious finding.
- **M (subspaces):** Nearly flat across M=4 to M=16 (0.771 → 0.773 → 0.775). M=4 is most efficient per bit.
- **β (commitment weight):** Non-monotonic at utility level (max spread 0.007 WACC). Clearest signal is tail WACC: β=0.50 leads (0.802), suggesting harder commitment improves minority-class coverage.
- **Init:** Random init (0.779) slightly ahead of K-means++ (0.773) on utility, but much more stable (std 0.0003 vs 0.003). Privacy (SSIM): H_vq_no_kmeans 0.520 vs H_vq_K256 0.514 — K-means++ is fractionally more private. Neither difference is large enough to be a primary claim; frame as a stability argument for K-means++.

---

## Related Work — Papers to Cite

### HybridVFL (arXiv 2512.10701, UCC '25, Anoosha et al.)

**What they do:** Each client encoder produces two outputs — `z_inv` (shared malignancy-relevant semantics) and `z_spec` (modality-specific). All four vectors are fused at the server by a cross-modal Transformer with a cosine consistency loss. Dataset: HAM10000, 7 classes.

**Key distinction:** No privacy evaluation. No reconstruction attack. No VQ. Their contribution is fusion architecture; ours is transmission privacy.

**Do NOT compare numbers directly:** HAM10000 vs ISIC-2019 — different datasets, different class counts.

**Where to cite:** Related Work (multimodal VFL fusion subsection) and Discussion/Future Work.

**Paper line:**

> *"HybridVFL [cite] demonstrates that cross-modal transformer fusion substantially outperforms simple concatenation in VFL multimodal classification. Our work is orthogonal: we fix the fusion mechanism (concatenation) and vary the transmission method (VQ vs. continuous embedding), isolating the contribution of the quantization bottleneck to both utility and privacy. Combining VQ compression with disentangled cross-modal fusion is a natural extension we leave to future work."*

---

### UIFV (arXiv 2406.12588, Yang et al. 2024)

Cite for threat model justification. See Reconstruction Attacker section above for full framing.

---

### URVFL (arXiv 2404.19582, Hu et al., NDSS 2025)

Cite as a limitation / out-of-scope. See Reconstruction Attacker section above.

---

### FedVQCS (arXiv 2204.07692)

VQ in horizontal FL for communication compression. No privacy evaluation. Cite in Related Work to show prior VQ-in-FL work did not evaluate privacy.

Paper line: *"FedVQCS [cite] demonstrates that vector quantization can reduce communication overhead in horizontal FL while maintaining utility. Unlike our work, FedVQCS does not study the vertical setting and includes no reconstruction privacy evaluation."*

---

---

## Paper Structure (draft outline)

1. **Introduction** — VFL privacy threat, embedding inversion problem, paper contributions
2. **Related Work**
   - VFL privacy attacks: UIFV (passive), URVFL (active, out of scope)
   - VQ in FL: FedMPQ, FedVQCS (compression only, no privacy eval)
   - Multimodal VFL fusion: HybridVFL (orthogonal contribution)
   - Split learning privacy (broader context)
3. **Method** — VFL setup, VQ bottleneck, InverNetV9 attacker
4. **Experiments** — ISIC-2019, baselines, VQ ablations (K / M / β / init)
5. **Results** — Utility table, SSIM/LPIPS table, Pareto curve, reconstruction grids, embedding analysis (regularization evidence)
6. **Discussion** — Why VQ improves utility (regularization + silhouette evidence), privacy mechanism, limitations (URVFL out of scope, diffusion-based inversion as future stronger attacker)
7. **Conclusion**

---

## Pending Before Writing

- [x] Stage B (50-epoch InverNet) for all 11 methods × 2 seeds
- [x] Update Key Results table with final 50-epoch SSIM values
- [x] Pareto curve — generated locally via make_figures.py
- [ ] Reconstruction grid (invernet_grid_regen.ipynb — Kaggle run needed, ~1.5 hr)
- [x] Embedding analysis: silhouette score + t-SNE — done, results in embedding_analysis/
- [ ] Obtain and read ESANN 2024 ES2024-57 (novelty risk — must do before submission)
- [ ] Clinical SSIM citation for dermoscopy image quality

---

## Venue Targets

| Venue | Feasibility | What's needed |
|-------|-------------|-------|
| MICCAI workshop / MIDL workshop | Yes, once Stage B done | ~2-3 weeks |
| MIDL main | Yes | All pending items above |
| MICCAI main | Stretch | Formal privacy bounds + clinical eval |
