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

## HybridVFL — Related Work Positioning

**Paper:** "HybridVFL: Disentangled Feature Learning for Edge-Enabled Vertical Federated Multimodal Classification" (arXiv 2512.10701, UCC '25, Anoosha et al.)

**What they do:** Each client encoder produces two outputs — `z_inv` (malignancy-invariant, shared semantics) and `z_spec` (modality-specific). All four vectors are sent to the server and fused by a cross-modal Transformer with a cosine consistency loss aligning the two clients' invariant embeddings.

**Their dataset:** HAM10000 (10,015 images, 7 classes). Their Simple Concatenation VFL baseline: balanced accuracy 0.7766. HybridVFL: 0.9235 (+14.69 pp).

**Why you cannot compare their numbers to ours:** Different datasets (HAM10000 vs ISIC-2019), different class counts (7 vs 8), different image sources. Any numerical comparison would be invalid.

**How to use this in the paper (Related Work):**

> *"HybridVFL [cite] demonstrates that cross-modal transformer fusion substantially outperforms simple concatenation in VFL multimodal classification. Our work is orthogonal: we fix the fusion mechanism (concatenation, as in their baseline) and vary the transmission method (VQ vs. continuous embedding), isolating the contribution of the quantization bottleneck to both utility and privacy. Combining VQ compression with disentangled cross-modal fusion is a natural extension we leave to future work."*

**Key distinction to maintain:** HybridVFL has no privacy evaluation (no reconstruction attack). They do not claim VQ or any privacy mechanism. Our contribution — showing VQ simultaneously improves utility and reduces reconstruction fidelity — is not present in their paper.

**Future work hook:** Phase 2 of this project (cross-modal attention + VQ privacy layer) directly extends HybridVFL's fusion idea with our quantization bottleneck.

---

## PAD-UFES-20 External Validation — Paper-Ready Notes

**Role in the paper:** PAD-UFES-20 is the external validation dataset after ISIC-2019. It tests whether the VQ communication bottleneck transfers from ISIC dermoscopy-style imagery to smartphone clinical skin-lesion imagery with structured metadata.

**Dataset summary:** PAD-UFES-20 contains 2,298 smartphone clinical skin-lesion images from 1,373 patients and 1,641 lesions. The native task is six-class diagnosis: ACK, BCC, MEL, NEV, SCC, and SEK. The dataset includes structured patient/lesion metadata, but also contains fields that must be excluded from model inputs because they are identifiers, labels, or diagnostic-process proxies. Class imbalance is substantial, especially for MEL, so balanced accuracy, per-class recall, and tail recall should be reported.

**Why it belongs in this paper:** Recent 2025/2026 PAD-UFES-20 work studies multimodal skin-lesion diagnosis with centralized image+metadata fusion. Our experiment studies the vertical version of the same clinical data structure: the image party transmits only a representation, the metadata-and-label party performs classification, and reconstruction privacy is evaluated from the transmitted representation. This positions our contribution as orthogonal to fusion architecture papers.

### Recent PAD-UFES-20 Work to Cite

| Paper | Year | Relevance |
|---|---:|---|
| **DualRefNet: Multimodal dual-stage feature refinement for robust skin lesion classification**, Scientific Reports | 2025 | Uses PAD-UFES20 and ISIC-2019 for image+metadata fusion; supports PAD as an active multimodal benchmark. URL: https://www.nature.com/articles/s41598-025-14839-7 |
| **MetaBlock-SE: A Method to Deal With Missing Metadata in Multimodal Skin Cancer Classification**, IEEE JBHI | 2025 | Uses PAD-UFES-20 and extended PAD data for metadata-robust multimodal classification; centralized fusion, not VFL privacy. DOI: 10.1109/JBHI.2025.3612837 |
| **HCHS-Net: A Multimodal Handcrafted Feature and Metadata Framework for Interpretable Skin Lesion Classification**, Biomimetics | 2026 | Six-class PAD-UFES-20 study combining visual features with clinical metadata; useful evidence that PAD remains active in 2026. URL: https://www.mdpi.com/2313-7673/11/2/154 |
| **DermaCalibra: A Robust and Explainable Multimodal Framework for Skin Lesion Diagnosis via Bayesian Uncertainty and Dynamic Modulation**, Diagnostics | 2026 | Uses PAD-UFES-20 for uncertainty-aware multimodal diagnosis; relevant recent centralized multimodal comparison point. URL: https://www.mdpi.com/2075-4418/16/4/630 |
| **Cross-Attention Enables Context-Aware Multimodal Skin Lesion Diagnosis**, medRxiv preprint | 2026 | Uses PAD-UFES-20 with image+metadata and compares metadata-only, image-only, late fusion, and cross-attention. Cite only as a preprint. URL: https://www.medrxiv.org/content/10.64898/2026.03.10.26348046v1 |

### Related-Work Positioning

Paper line:

> *"Recent PAD-UFES-20 studies demonstrate renewed interest in multimodal skin-lesion diagnosis using smartphone images and structured clinical metadata. These works primarily study centralized fusion architectures. In contrast, we evaluate the vertical setting in which the image holder transmits only a compressed representation to the metadata-and-label holder, and we measure both diagnostic utility and reconstruction leakage from that representation."*

This distinction should be maintained throughout the paper:

| Recent PAD papers | This paper |
|---|---|
| Centralized multimodal fusion | Vertical split between image holder and metadata/label holder |
| Optimize diagnosis accuracy, calibration, or interpretability | Optimize privacy/utility tradeoff at the communication boundary |
| Raw image features available inside one training pipeline | Passive image representation is the privacy-sensitive transmitted object |
| No reconstruction privacy evaluation | InverNetV9 reconstruction attack with SSIM/LPIPS |

### PAD Experimental Protocol

**Task:** Six-class classification using native PAD labels: ACK, BCC, MEL, NEV, SCC, SEK. Keep this as the primary PAD task because it aligns with the existing v9 multiclass setup and recent six-class PAD work. A binary malignant/benign variant can be reported as an appendix if needed.

**VFL split:**
- Passive client: smartphone lesion image.
- Active client: non-leaky metadata and diagnosis label.
- Passive architecture: same EfficientNet-B0, projection, sign, and VQ variants as v9.
- Active architecture: same metadata MLP and classifier structure as v9, with `NUM_CLASSES = 6`.
- Privacy evaluation: same InverNetV9 reconstruction attack from transmitted representation.

**Primary metadata setting: `PAD-core+skin`**
- Include: `age`, `gender`, `region`, `fitspatrick`.
- Rationale: closest to ISIC age/sex/site metadata while adding Fitzpatrick skin type as legitimate clinical context.

**Secondary metadata setting: `PAD-clinical`**
- Include non-leaky symptoms/risk factors: lesion diameters, smoke/drink/pesticide, cancer history, skin cancer history, itch, grew, hurt, changed, bleed, elevation.
- Use as a secondary setting because richer clinical metadata may dominate the classifier and reduce the interpretability of image-side VQ effects.

**Excluded fields:**
- `patient_id`, `lesion_id`, `img_id`: identifiers; use only for splitting and audit.
- `diagnostic`: target label.
- `biopsed` / biopsy indicator: diagnostic-process proxy and likely label leakage.
- `background_father`, `background_mother`, `has_piped_water`, `has_sewage_system`: exclude from main experiments to avoid ancestry/socioeconomic confounding. These can be reserved for optional sensitivity analysis only.

**Splitting:** Use patient-grouped stratified splits. No patient may appear in both training and validation. Image-level random splitting is invalid because PAD contains repeated patients/lesions. Validation splits must contain MEL and SCC.

**Image preprocessing:** Keep the v9 architecture, but use lesion-safe PAD preprocessing. Build a square resize/pad cache and use conservative random crops, e.g. `RandomResizedCrop(scale=(0.70,1.0))`, rather than the aggressive ISIC crop. Save a crop-audit grid before final runs.

**Minimum PAD method grid:**

| Method | Purpose |
|---|---|
| `A_plain_vfl` | continuous transmission baseline |
| `A_proj_vfl` | projection-only control |
| `S_sign_quant` | naive 128-bit discrete baseline |
| `H_vq_K64` | strong ISIC utility point |
| `H_vq_K256` | reference VQ setting |
| `H_vq_M4` | 32-bit aggressive compression |
| `H_vq_M16` | equal-bit comparison against sign quantization |
| `H_vq_commit_high` | high-commitment VQ setting |

**Additional controls:**
- Metadata-only PAD-core+skin model.
- Image-only model.

These controls are needed to show that PAD results are not driven entirely by metadata or entirely by the image backbone.

### PAD Result Interpretation

Strongest positive result:

```text
H_vq_K64 or H_vq_M4 keeps WACC close to A_plain_vfl / A_proj_vfl,
reduces SSIM relative to continuous baselines,
increases LPIPS relative to continuous baselines,
and uses 32-64 bits instead of 4096-40960 bits.
```

Key same-bit comparison:

```text
H_vq_M16 vs S_sign_quant at 128 bits.
If H_vq_M16 matches or exceeds sign utility and reduces reconstruction quality,
the learned codebook claim is strengthened.
```

If VQ lowers reconstruction quality but costs utility, report it as a privacy/utility tradeoff. If sign quantization matches VQ at equal bits, soften the claim from "learned VQ is superior" to "discrete bottlenecks provide reconstruction resistance, with VQ offering a tunable learned variant."

### Paper Paragraph Draft

> *"We additionally evaluate on PAD-UFES-20, a public smartphone clinical skin-lesion dataset with structured patient metadata. Although introduced in 2020, PAD-UFES-20 remains an active benchmark in recent 2025/2026 multimodal skin-lesion work, including DualRefNet, MetaBlock-SE, HCHS-Net, and DermaCalibra. These studies focus on centralized image-metadata fusion. We instead use PAD-UFES-20 to evaluate a vertical version of the same clinical setting: the image holder transmits only an intermediate representation, the metadata-and-label holder performs classification, and reconstruction leakage is measured from the transmitted representation. This tests whether the VQ bottleneck behavior observed on ISIC-2019 transfers to smartphone clinical imagery under a recent multimodal benchmark context."*

---

## Paper Structure (draft outline)

1. Introduction — VFL privacy threat, embedding inversion problem
2. Related Work — VFL privacy attacks (UIFV, URVFL), VQ in FL (FedMPQ, FedVQCS), multimodal VFL fusion (HybridVFL), split learning privacy
3. Method — VFL setup, VQ bottleneck, InverNetV9 attacker
4. Experiments — Datasets (ISIC-2019 primary, PAD-UFES-20 external validation), baselines (A_plain, A_proj, sign methods), VQ ablations
5. Results — Utility table, SSIM table, PAD external-validation table, Pareto curve, reconstruction grids
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
- [ ] Implement PAD-UFES-20 loader with patient-grouped split and leakage-safe metadata
- [ ] Run PAD-core+skin minimum method grid + Stage B reconstruction
- [ ] Add metadata-only and image-only PAD controls

---

## Venue Targets

| Venue | Feasibility | Notes |
|-------|-------------|-------|
| MICCAI workshop / MIDL workshop | Yes, once Stage B done | ~2-3 weeks |
| MIDL main | Yes | All above items |
| MICCAI main | Stretch | Formal privacy bounds + clinical eval needed |
