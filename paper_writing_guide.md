# Paper Writing Guide

---

## Template Structure (TA vs. Our Outline)

The TA template (ICML 2021) uses: Abstract → Introduction → Methodology → Results → Discussion → Conclusion.

Map our 7-section outline to this:

| Our outline | Template section |
|---|---|
| Introduction | Introduction |
| Related Work | Add as subsection at end of Introduction, OR as a standalone section before Methodology |
| Method + Experiments | Methodology |
| Results | Results |
| Discussion + Ablations | Discussion |
| Conclusion | Conclusion |

The TA instructions say to structure by **task progression**: Baseline → First Improvement → Second Improvement. This maps to: A_plain_vfl (baseline) → A_proj_vfl / S_sign_quant / S_rand_sign (first improvement) → H_vq_* (second improvement / main contribution).

---

## Missing Citations (Add to main.bib Before Writing)

Three citations are missing from the citation bank in `paper_novelty_source_of_truth.md` section 14. They will be needed in Methodology.

| What | Why needed | Paper |
|---|---|---|
| **EfficientNet-B0** | Passive encoder backbone | Tan & Le, "EfficientNet: Rethinking Model Scaling for Convolutional Neural Networks," ICML 2019 |
| **Straight-through estimator** | Gradient flow through VQ discrete lookup | Bengio et al., "Estimating or Propagating Gradients Through Stochastic Neurons for Conditional Computation," arXiv 2013 |
| **ISIC-2019 dataset** | Citing the dataset formally | Combalia et al. 2019 (BCN20000) + Tschandl et al. 2018 (HAM10000) — or the ISIC challenge descriptor |

---

## Writing Order

Write in this order. Abstract is last.

1. Methodology
2. Results
3. Discussion
4. Introduction + Related Work
5. Abstract
6. Conclusion

---

## Section-by-Section Prompting Guide

Use this base prompt for every section:

> "Write the [Section Name] section of our VFL paper. Use ICML 2021 two-column LaTeX format. No AI writing tells: no 'delve', 'crucial', 'leverage', 'robust', 'furthermore', 'underscore', 'tapestry'. Direct academic prose, no fluff. Here is the context: [paste relevant block]. Constraints: [list framing rules]."

---

### Methodology

**Context to paste:** Sections 2 + 4 of `paper_novelty_source_of_truth.md`.

**Structure to request:**
1. VFL setup and notation (passive/active party, no raw data sharing)
2. Passive encoder pipeline (image → EfficientNet-B0 → 1280-d → learned 128-d projection)
3. Product-quantized bottleneck (M subspaces, K codebook, bits = M·log₂K) — request this as a LaTeX equation
4. Baselines (A_plain_vfl, A_proj_vfl, S_sign_quant, S_rand_sign) with bit counts
5. InverNetV9 attacker (FC → ResBlock×2 → ConvTranspose×4 → Refine, MSE + 0.1·LPIPS, 50 epochs)
6. Dataset: ISIC-2019, 8 classes, WACC as primary metric, passive=image / active=metadata+label
7. Training details and evaluation protocol

**Key constraint:** VQ quantizes the 128-d projected representation, NOT the 1280-d EfficientNet embedding. Flag this explicitly.

---

### Results

**Context to paste:** Section 3 results table + section 8 from `paper_novelty_source_of_truth.md`. Also paste the Key Results table from `paper_notes.md`.

**Structure to request:**
1. Utility + communication table (all 11 methods, WACC ± std, bits)
2. Privacy table (all 11 methods, SSIM, LPIPS)
3. Pareto dominance of H_vq_K64: best WACC (0.786) and lowest SSIM (0.503) at only 48 bits (853× fewer than A_plain_vfl)
4. Equal-bit comparison: H_vq_M16 vs S_sign_quant at 128 bits

**Critical framing constraint:** The equal-bit SSIM gap is 0.002 (0.540 vs 0.538) — too small to claim VQ is "more private." Frame this as: "VQ achieves better utility (+0.5pp WACC) at equivalent privacy." Do not say VQ wins on privacy in the 128-bit comparison.

**Figures to reference:** Pareto curve (`figures/pareto_wacc_vs_ssim.pdf`), reconstruction grids (from invernet_grid_regen.ipynb), bits vs WACC/SSIM plots.

---

### Discussion

**Context to paste:** Hyperparameter findings + argument arc from `paper_notes.md`. Sections 9 + 12 + 13 from `paper_novelty_source_of_truth.md`.

**Structure to request:**
1. Why VQ improves utility (regularization hypothesis: VQ strips high-frequency embedding variation the classifier doesn't need; silhouette score evidence; t-SNE visual)
2. Why A_proj_vfl is an essential control (proves the 128-d projection alone is insufficient — SSIM 0.585 vs 0.503 for H_vq_K64)
3. Ablation findings: K (smaller K = stronger regularizer, K=64 beats K=256 by 1.3pp WACC), M (nearly flat 0.771→0.775), β (non-monotonic at utility; β=0.50 leads on tail WACC)
4. Limitations: URVFL out of scope (different threat model — active/malicious), one dataset (ISIC-2019 only), no formal DP bounds, InverNetV9 is empirical not adversarially optimal

**Silhouette numbers to include verbatim** (from `embedding_analysis/silhouette_table.json`):

| Method | Silhouette | Codebook Util | Bits |
|---|---|---|---|
| A_plain_vfl | 0.015 | — | 40960 |
| A_proj_vfl | −0.015 | — | 4096 |
| S_sign_quant | 0.058 | — | 128 |
| H_vq_K64 | 0.074 | 98.8% | 48 |
| H_vq_K256 | 0.074 | 97.9% | 64 |
| H_vq_M4 | 0.136 | 92.1% | 32 |
| H_vq_M16 | 0.053 | 99.5% | 128 |

Key framing: A_proj_vfl being negative (−0.015) is the strongest evidence that regularization comes from VQ discretization, not the projection layer. M controls regularization strength monotonically (M4 highest). Codebook utilization 92–99% rules out collapse. Reference figures: `embedding_analysis/tsne_comparison.png` and `embedding_analysis/silhouette_by_method.png`.

---

### Introduction + Related Work

**Context to paste:** Sections 5 + 6 + 11 intro points from `paper_novelty_source_of_truth.md`. Paper contributions list from `paper_notes.md`.

**Structure to request:**

*Introduction:*
1. Clinical motivation: multi-institutional skin lesion diagnosis, image and metadata at different parties
2. VFL as the natural setup; intermediate representations are the attack surface
3. Continuous embeddings are high-bandwidth and reconstructable (UIFV threat)
4. Our approach: learned discrete bottleneck
5. Contributions list (4 bullet points — see paper_notes.md method section notes)

*Related Work subsections:*
- Vertical federated learning + communication efficiency (Compressed-VFL, SparseVFL, LESS-VFL)
- Privacy attacks and defenses in VFL (UIFV — threat model citation; URVFL — out-of-scope mention; FedPass)
- VQ in FL (FedVQCS, FedMPQ — both are communication compression, no reconstruction privacy eval)
- Multimodal VFL fusion (HybridVFL — orthogonal: their novelty is fusion, ours is transmission)

**Key framing for HybridVFL:** "HybridVFL demonstrates that cross-modal Transformer fusion outperforms concatenation; our work is orthogonal — we fix fusion (concatenation) and vary the passive transmission mechanism."

---

### Abstract

Write this last. Provide Claude with the final results table and the one-sentence claim.

**Target:** ≤150 words. Must include: problem (VFL embedding leakage), method (learned product quantization bottleneck), key result (H_vq_K64: +3.4pp WACC over continuous VFL, SSIM 0.503 vs 0.622, 853× fewer bits), and one-line implication.

---

### Conclusion

**Context to paste:** Section 9 research questions (answered) from `paper_novelty_source_of_truth.md`.

**Structure to request:**
1. Restate what was shown (Pareto improvement: utility + compression + privacy together)
2. Key finding: smaller K is a stronger regularizer, not worse
3. Limitations that are also future work: URVFL-class attacks, diffusion-based inversion, cross-domain validation, formal privacy bounds

---

## Checklist Before Sending to Claude

- [ ] Add 3 missing BibTeX entries to main.bib (EfficientNet, STE, ISIC-2019 dataset)
- [ ] Paste section-specific context blocks from paper_novelty_source_of_truth.md (do not rely on Claude knowing the file)
- [ ] Always include the framing constraint for the equal-bit comparison
- [ ] Always include the reviewer trap: VQ quantizes 128-d projection, not 1280-d embedding
- [ ] Do not mention PAD-UFES-20 anywhere in the paper
- [ ] Do not claim formal DP or URVFL robustness
- [ ] GitHub repo link goes at the top of the paper (per TA instructions)
- [ ] File name format: GroupNumber_RollNum1_RollNum2_Report.pdf
