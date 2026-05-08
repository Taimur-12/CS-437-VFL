# Paper Novelty Source of Truth

Last verified: 2026-05-08.

Purpose: this file is the working source of truth for the paper idea before drafting. It separates the current defensible paper claim from older/stale directions, records the exact novelty position against related work, and lists what still must be proven before submission.

---

## 1. Current Paper Idea

### Working title

**Product-Quantized Passive Representations for Communication-Efficient Reconstruction Privacy in Medical Vertical Federated Learning**

Shorter alternatives:

- **VQ Bottlenecks for Reconstruction Privacy in Medical VFL**
- **Learned Discrete Communication Bottlenecks for Private Medical VFL**
- **Product-Quantized VFL for Skin Lesion Diagnosis**

### One-sentence idea

In vertical federated learning (VFL), replace the passive image client's continuous transmitted representation with a learned product-quantized representation, then evaluate whether this bottleneck preserves or improves diagnostic utility while reducing communication cost and passive image reconstructability.

### Current strongest claim

**Established:** On ISIC-2019, learned product quantization at the passive communication boundary improves balanced accuracy over continuous VFL while reducing transmitted representation size from 40960 bits to 32-128 bits, and simultaneously reduces reconstruction SSIM (0.503 for H_vq_K64 vs 0.622 for A_plain_vfl). The paper can claim an empirical privacy-utility-communication Pareto improvement. H_vq_K64 is Pareto-dominant: best WACC and lowest SSIM of all 11 methods.

### What changed from older versions

The paper is no longer about:

- GRL/adversarial label obfuscation.
- Label inference AUC as the main privacy metric.
- Cross-modal attention as the main novelty.
- A broad "medical VFL SOTA" claim.

The paper is now about:

- Passive image-party representation leakage.
- Reconstruction privacy from transmitted embeddings.
- Learned product quantization as a communication/privacy bottleneck.
- Empirical utility/compression/privacy results on ISIC-2019. PAD-UFES-20 external validation was attempted but results were poor and that direction is dropped.

---

## 2. Exact Method Being Claimed

### VFL partition

Primary setting:

- Passive party: lesion image.
- Active party: tabular metadata plus diagnostic label.
- Server/active-side classifier: receives only the passive representation plus active metadata.
- Task: skin lesion classification.

This matches VFL because parties share sample IDs but hold different feature sets. A standard description of VFL is that parties have the same samples but disjoint features, often with the label held by one party; see the VFL background in Khan et al. 2022 and FedPass 2023 ([MDPI Algorithms](https://www.mdpi.com/1999-4893/15/8/273), [IJCAI FedPass](https://www.ijcai.org/proceedings/2023/418)).

### Passive representation pipeline

The actual v9/v10 code does this:

```text
image -> EfficientNet-B0 backbone -> 1280-d embedding -> learned 128-d projection -> optional communication transform
```

Communication transforms:

- `A_plain_vfl`: transmit 1280-d continuous embedding, 40960 bits.
- `A_proj_vfl`: transmit 128-d continuous projection, 4096 bits.
- `S_sign_quant`: transmit sign of the learned 128-d projection, 128 bits.
- `S_rand_sign`: frozen random projection plus sign, 128 bits.
- `H_vq_*`: product quantize the learned 128-d projection, transmit `M * log2(K)` bits.

Important correction: older notes sometimes say the 1280-d EfficientNet embedding is split directly into product-quantized subspaces. That is not what the current notebook implements. The current method quantizes the **128-d projected representation**.

### Product quantization details

For VQ methods, the 128-d projected vector is split into `M` subspaces. Each subspace is assigned to the nearest entry in a learned codebook of size `K`. The number of transmitted bits is:

```text
bits = M * log2(K)
```

Examples:

- `H_vq_M4`: `M=4`, `K=256`, 32 bits.
- `H_vq_K64`: `M=8`, `K=64`, 48 bits.
- `H_vq_K256`: `M=8`, `K=256`, 64 bits.
- `H_vq_M16`: `M=16`, `K=256`, 128 bits.

The conceptual basis is product quantization, originally used to represent high-dimensional vectors by subspace code indices for compact approximate search ([Jegou et al. 2011, IEEE TPAMI](https://doi.org/10.1109/TPAMI.2010.57)). The differentiable training style follows the broader neural VQ/VQ-VAE tradition, where discrete codes are learned using vector quantization ideas and straight-through-style optimization ([van den Oord et al. 2017, NeurIPS](https://papers.neurips.cc/paper/7210-neural-discrete-representation-learning)).

---

## 3. Current Local Evidence

Local primary artifacts:

- `shazil_v9.ipynb`: actual ISIC-2019 v9 implementation.
- `results_final/result_*.json`: utility results for 11 methods x 2 seeds.
- `results_final/recon_*.json`: SSIM/LPIPS results for 11 methods x 2 seeds (50-epoch InverNet).
- `results_final/checkpoints/*.pt`: saved checkpoints for all 11 methods x 2 seeds.
- `results_analysis.md`: complete analysis of utility, privacy, and regularization evidence.
- `figures/`: Pareto curve and supporting bits-vs-WACC/SSIM plots (from make_figures.py).
- `embedding_analysis/`: silhouette table JSON, silhouette bar chart, t-SNE comparison, individual t-SNE PNGs.

### ISIC-2019 utility/compression results

All numbers below are from the local `results_final/result_*.json` files and `results_analysis.md`.

| Method | Bits | WACC mean +/- std | Tail WACC | SSIM | LPIPS |
|---|---:|---:|---:|---:|---:|
| `A_plain_vfl` | 40960 | 0.7516 +/- 0.0043 | 0.7880 | 0.622 | 0.111 |
| `A_proj_vfl` | 4096 | 0.7665 +/- 0.0013 | 0.7933 | 0.585 | 0.139 |
| `S_rand_sign` | 128 | 0.7232 +/- 0.0067 | 0.7675 | 0.529 | 0.214 |
| `S_sign_quant` | 128 | 0.7703 +/- 0.0062 | 0.7955 | 0.538 | 0.196 |
| `H_vq_K64` | 48 | **0.7860 +/- 0.0072** | **0.8095** | **0.503** | 0.241 |
| `H_vq_K256` | 64 | 0.7727 +/- 0.0034 | 0.7907 | 0.514 | 0.227 |
| `H_vq_no_kmeans` | 64 | 0.7790 +/- 0.0003 | 0.7997 | 0.520 | 0.232 |
| `H_vq_M4` | 32 | 0.7706 +/- 0.0049 | 0.7885 | 0.512 | 0.239 |
| `H_vq_M16` | 128 | 0.7748 +/- 0.0017 | 0.7917 | 0.540 | 0.194 |
| `H_vq_commit_low` | 64 | 0.7781 +/- 0.0069 | 0.7913 | 0.512 | 0.226 |
| `H_vq_commit_high` | 64 | 0.7803 +/- 0.0014 | 0.8017 | 0.512 | 0.231 |

All values final. WACC: 2 seeds. SSIM/LPIPS: 50-epoch InverNet × 2 seeds. SSIM higher = easier to reconstruct = less private.

### Utility observations that are currently defensible

1. Every VQ method beats `A_plain_vfl` on WACC.
2. `H_vq_K64` is the best current utility point: 0.7860 WACC at 48 bits.
3. `H_vq_K64` beats `A_plain_vfl` by about 3.4 WACC points while using about 853x fewer bits.
4. `H_vq_M4` uses only 32 bits and still beats `A_plain_vfl` by about 1.9 WACC points.
5. At equal 128-bit communication, `H_vq_M16` beats `S_sign_quant` on utility: 0.7748 vs 0.7703 WACC.
6. `A_proj_vfl` is a critical control: it tests whether the 128-d projection alone, not VQ, explains any privacy or utility changes.

### Privacy evidence status

Stage B complete for all 11 methods × 2 seeds at 50 InverNet epochs. The privacy claim is now defensible. Key comparisons confirmed:

- `A_plain_vfl` SSIM 0.622 vs `H_vq_K64` SSIM 0.503: −0.119 absolute, largest drop.
- `A_proj_vfl` SSIM 0.585 vs `H_vq_K64` SSIM 0.503: projection alone is insufficient.
- Equal-bit (128b): `H_vq_M16` SSIM 0.540 vs `S_sign_quant` SSIM 0.538: essentially tied on privacy, VQ wins utility (+0.5pp WACC). Frame as "VQ achieves better utility at equivalent privacy," not "VQ is more private."

---

## 4. Threat Model and Privacy Metric

### Threat model

The primary threat model is passive inversion from the transmitted representation:

- Attacker observes representations transmitted by the passive image party.
- Attacker trains an image decoder to reconstruct the passive image/crop.
- Attacker does not manipulate gradients.
- Attacker does not need labels.
- Attacker does not need the passive client's raw images at inference time.

This is aligned with the passive feature-inversion framing of UIFV, which uses intermediate feature data exchanged during VFL inference to reconstruct original data ([UIFV arXiv:2406.12588](https://arxiv.org/abs/2406.12588)).

### Attacker implementation

Your implementation uses `InverNetV9`:

```text
representation -> FC -> 2 residual blocks -> 4 ConvTranspose upsampling blocks -> refinement conv -> 64x64 RGB image
loss = MSE + 0.1 * LPIPS
metrics = SSIM, LPIPS
```

The target image is the same validation spatial extent seen by the encoder: center crop to 224, then resize to 64. This avoids making the attacker reconstruct image regions the encoder never observed.

### Metrics

- SSIM: higher means the reconstruction is more structurally similar, hence less private in this setting. SSIM was introduced as an image quality metric based on structural similarity ([Wang et al. 2004](https://pubmed.ncbi.nlm.nih.gov/15376593/)).
- LPIPS: lower means perceptually more similar, higher means less similar. LPIPS was proposed as a deep-feature perceptual similarity metric ([Zhang et al. 2018](https://arxiv.org/abs/1801.03924)).

### What is out of scope

URVFL is an active malicious VFL attack that uses malicious gradient behavior and label information to improve reconstruction while evading detection ([URVFL arXiv:2404.19582](https://arxiv.org/abs/2404.19582), [NDSS 2025 page](https://www.ndss-symposium.org/ndss-paper/urvfl-undetectable-data-reconstruction-attack-on-vertical-federated-learning/)). Your current defense is not designed for that threat model. Mention URVFL as a limitation, not as a baseline you have already handled.

---

## 5. Novelty Claim

### Primary novelty claim

The defensible novelty is:

> A learned product-quantized communication bottleneck at the passive-client representation boundary in medical VFL, evaluated jointly for diagnostic utility, communication cost, and passive image reconstruction leakage.

This is not just "VQ in FL" and not just "communication-efficient VFL." The novelty is the **intersection**:

1. VFL, not horizontal FL.
2. Passive image representation, not model-update compression.
3. Learned product quantization, not scalar quantization/top-k sparsification.
4. Reconstruction privacy evaluation, not only communication savings.
5. Medical multimodal skin-lesion setting, with image/metadata vertical partition.

### Strongest final paper claim if Stage B succeeds

If full Stage B confirms the expected pattern, the paper can say:

> Product quantization creates a tunable discrete bottleneck that improves the utility/communication tradeoff and empirically reduces reconstruction leakage from the passive image representation compared with continuous VFL and projection-only VFL.

### Current full claim (Stage B complete)

> Product quantization creates a tunable discrete bottleneck that simultaneously improves diagnostic utility, reduces communication by 300–1280×, and empirically reduces image reconstructability relative to continuous and projection-only VFL. H_vq_K64 is Pareto-dominant across all 11 methods on ISIC-2019.

### Claims not to make

Do not claim:

- Formal privacy guarantees.
- Differential privacy.
- Security against malicious gradient attacks.
- Security against diffusion-based inversion.
- Universal medical imaging generalization.
- That VQ always beats sign quantization on privacy.
- That PAD or ISIC-2024 results exist before they are run.
- That HybridVFL missed the field; HybridVFL exists and is directly relevant to multimodal VFL fusion.

---

## 6. Related Work Map

### VFL basics and communication bottleneck

VFL handles parties with overlapping samples but different feature spaces. This creates repeated exchange of intermediate embeddings and gradients in split/deep VFL, making communication cost and representation leakage central issues.

Relevant work:

- **Compressed-VFL** proposes communication-efficient VFL with compressed intermediate results and gives convergence analysis for compression such as quantization and top-k sparsification ([ICML 2022](https://icml.cc/virtual/2022/poster/16395)). Difference: communication/theory focus, not learned product VQ at the passive medical image boundary, and not reconstruction privacy as the central empirical endpoint.
- **SparseVFL** sparsifies embeddings and gradients using ReLU/L1/masked-gradient/run-length coding to reduce communication ([OpenReview 2023](https://openreview.net/forum?id=BVH3-XCRoN3)). Difference: sparsification, not learned product quantization; communication, not image reconstruction privacy.
- **LESS-VFL** performs communication-efficient feature selection in VFL ([IBM Research / ICML 2023](https://research.ibm.com/publications/less-vfl-communication-efficient-feature-selection-for-vertical-federated-learning)). Difference: feature selection/generalization/efficiency, not passive image reconstruction leakage.

### VFL privacy defenses

Relevant work:

- **FedPass** uses adaptive obfuscation to protect private features and labels in vertical federated deep learning, with theoretical privacy claims and empirical utility/privacy tradeoffs ([IJCAI 2023](https://www.ijcai.org/proceedings/2023/418)). Difference: adaptive obfuscation/privacy defense, not learned product quantization; not a medical image reconstruction Pareto study.
- **UIFV** is especially relevant as a threat-model citation because it reconstructs private data from VFL intermediate feature data rather than relying only on gradients or full model details ([arXiv:2406.12588](https://arxiv.org/abs/2406.12588)). Difference: attack framework, not a defense.
- **URVFL** is relevant as an out-of-scope stronger attack because it is malicious/active and gradient-spoofing based ([arXiv:2404.19582](https://arxiv.org/abs/2404.19582)). Difference: different threat model.

### VQ and product quantization in FL

Relevant work:

- **FedVQCS** compresses local model updates in horizontal FL using dimensionality reduction plus vector quantization/compressed sensing ([arXiv:2204.07692](https://arxiv.org/abs/2204.07692)). Difference: horizontal FL model-update compression, not VFL passive embedding transmission, and not reconstruction privacy.
- **FedMPQ** uses multi-codebook product quantization for communication-efficient secure aggregation in cross-device FL ([arXiv:2404.13575](https://arxiv.org/abs/2404.13575)). Difference: FL uplink/model-update compression under secure aggregation, not VFL image embedding privacy.
- **ESANN 2024: About Vector Quantization and its Privacy in Federated Learning** considers how privacy of VQ models can be broken in an FL setting by exposing data from prototype updates ([ESANN proceedings](https://www.esann.org/proceedings/2024), [PDF](https://www.esann.org/sites/default/files/proceedings/2024/ES2024-57.pdf)). Difference based on abstract/PDF snippet: it studies leakage from prototype updates in VQ models, not using a learned product quantizer as a passive representation bottleneck in VFL. This is still the closest novelty-risk paper and must be read carefully before submission.

### Multimodal VFL fusion

Relevant work:

- **HybridVFL** uses client-side disentangled encodings plus server-side cross-modal Transformer fusion for edge-enabled multimodal healthcare classification on HAM10000 ([arXiv:2512.10701](https://arxiv.org/abs/2512.10701)). Difference: fusion architecture and utility, not VQ bottlenecks or reconstruction privacy evaluation. HybridVFL means cross-modal attention/fusion is not your novelty. Your work is orthogonal: keep fusion simple and vary the passive communication mechanism.

### Metrics and reconstruction

Relevant work:

- SSIM is a structural image similarity metric ([Wang et al. 2004](https://pubmed.ncbi.nlm.nih.gov/15376593/)).
- LPIPS measures perceptual similarity using deep features ([Zhang et al. 2018](https://arxiv.org/abs/1801.03924)).
- UIFV motivates intermediate-feature inversion in VFL ([arXiv:2406.12588](https://arxiv.org/abs/2406.12588)).

---

## 7. Dataset Positioning

### ISIC-2019: primary dataset

ISIC-2019 is the current primary dataset in v9. The official challenge page describes 25,331 training images across 8 training categories, with a task using images and optional metadata, and uses balanced multiclass accuracy as the goal metric ([ISIC 2019 challenge](https://challenge.isic-archive.com/landing/2019/)).

Why this fits the paper:

- It has lesion images and metadata.
- It naturally supports an image-party / metadata-party VFL simulation.
- It is large enough for the current 11-method, 2-seed grid.
- Balanced accuracy is aligned with the official challenge metric and class imbalance.

Current limitation:

- It is one dermoscopy-style skin-lesion benchmark. It does not prove cross-domain medical generality.

### PAD-UFES-20: clean external validation

PAD-UFES-20 contains 2,298 smartphone clinical skin lesion images, 1,373 patients, 1,641 lesions, and structured clinical metadata; the Mendeley dataset page confirms the six-class structure and smartphone collection ([Mendeley Data](https://data.mendeley.com/datasets/zr7vgbcyr2/1)). The paper/data article reports smartphone clinical images and patient clinical data, with biopsy status highly entangled with cancer classes ([PAD-UFES-20 article](https://pmc.ncbi.nlm.nih.gov/articles/PMC7479321/)).

Why this fits:

- Same conceptual VFL setup: image party plus metadata/label party.
- Different acquisition regime: smartphone clinical images instead of dermoscopy-style images.
- Good external validation once v10 is run.

Important leakage rule:

- Exclude biopsy status from model inputs because BCC/SCC/MEL are biopsy-proven and biopsy status is a diagnostic pathway proxy.
- Use patient-grouped splits to avoid patient/lesion leakage.

Status:

- `shazil_v10_pad_ufes.ipynb` implements the PAD loader, leakage-safe features, patient-grouped splits, and the minimum method grid.
- PAD results are not yet local evidence unless a completed `results_d1v10_pad` table is produced.

### ISIC-2024 / SLICE-3D: latest-dataset extension, not current core

SLICE-3D/ISIC-2024 is a strong future/latest validation target. Scientific Data describes SLICE-3D as over 400,000 lesion crops from 3D total-body photography across seven dermatologic centers, with image quality comparable to smartphone crops ([Scientific Data 2024](https://www.nature.com/articles/s41597-024-03743-w)). A 2025 npj Digital Medicine paper analyzes the ISIC-2024 challenge and notes the competition collected over 900,000 lesion crops off 3D total-body photos ([npj Digital Medicine 2025](https://www.nature.com/articles/s41746-025-02070-7)).

Why this matters:

- It is newer than PAD and has active 2025 literature.
- It supports a contemporary skin-cancer triage framing.
- Existing work focuses on centralized/multimodal triage and explainability; the vertical privacy framing is still distinct.

Status:

- Future extension. Do not claim results.

---

## 8. What Makes the Paper Potentially Publishable

The paper is strongest if it produces the following chain:

```text
A_plain_vfl:
  high communication, continuous representation, likely high reconstructability

A_proj_vfl:
  tests whether projection alone explains privacy/utility

S_sign_quant:
  tests whether naive 128-bit discreteness is enough

H_vq_*:
  learned discrete bottleneck, 32-128 bits
```

The central figure should be a Pareto plot:

```text
x-axis: SSIM, lower is more private
y-axis: WACC, higher is better
point label/size: bits
color: method family
```

The paper-positive outcome: **confirmed.**

```text
H_vq_K64:
  WACC 0.786 >= A_plain_vfl 0.752  ✓
  SSIM 0.503 < A_proj_vfl 0.585    ✓
  LPIPS 0.241 > A_proj_vfl 0.139   ✓
  bits 48 <= 64                    ✓
```

Equal-bit learned-VQ outcome: **partially confirmed.**

```text
H_vq_M16 vs S_sign_quant, both 128 bits:
  WACC 0.775 >= 0.770              ✓ (+0.5pp)
  SSIM 0.540 vs 0.538              ~ (essentially tied, 0.002 gap)
  LPIPS 0.194 vs 0.196             ~ (essentially tied)
```

Frame the equal-bit result as: VQ achieves better utility at equivalent privacy, not "VQ is more private." The SSIM gap (0.002) is too small to claim a privacy advantage.

---

## 9. Research Questions

### Main research questions

1. Does learned product quantization preserve or improve diagnostic utility compared with continuous VFL? **Yes — all VQ methods beat A_plain_vfl.**
2. Does learned product quantization reduce communication by orders of magnitude? **Yes — 300–1280× reduction.**
3. Does learned product quantization reduce image reconstructability beyond 128-d projection alone? **Yes — SSIM 0.503 vs 0.585.**
4. Does learned VQ beat naive sign quantization at the same 128-bit budget? **On utility yes (+0.5pp); on privacy essentially tied (0.002 SSIM gap).**

### Secondary questions

1. Is smaller `K` better because it regularizes the passive representation?
2. Does `M` matter much once the projection dimension is fixed at 128?
3. Does commitment weight affect tail-class recall and reconstruction privacy?
4. Does K-means++ initialization help privacy even though random initialization slightly wins on utility?

---

## 10. Required Experiments Before Writing the Full Paper

Priority order:

1. ~~Run Stage B for all remaining ISIC methods~~ — **Done. All 11 methods × 2 seeds at 50 epochs.**
2. ~~Export final results table~~ — **Done. results_final/ contains all result_*.json and recon_*.json.**
3. ~~Generate Pareto plot~~ — **Done. figures/pareto_wacc_vs_ssim.pdf via make_figures.py.**
4. ~~Generate bits vs WACC and bits vs SSIM plots~~ — **Done.**
5. Generate aligned reconstruction grids — **In progress. Run invernet_grid_regen.ipynb on Kaggle (~1.5 hr).**
6. ~~Run 50-epoch attacker~~ — **Done. All Stage B ran at 50 epochs.**
7. ~~PAD-UFES-20~~ — **Dropped. Results were poor.**
8. ~~Run embedding analysis (silhouette + t-SNE)~~ — **Done. Results in `embedding_analysis/`. Silhouette confirms regularization hypothesis; A_proj_vfl is negative (−0.015), H_vq_M4 leads at 0.136.**
9. Read the full ESANN 2024 VQ privacy PDF — **Still required before submission.**

---

## 11. Paper Outline

### Introduction

Key points:

- VFL is attractive for medical multimodal data because image and metadata may live at different parties.
- Raw data are not shared, but intermediate representations can still leak passive images.
- Continuous embeddings are high-bandwidth and reconstructable.
- We test whether a learned discrete bottleneck can improve utility, communication, and reconstruction privacy together.

Contributions:

1. Product-quantized passive representation transmission for medical VFL.
2. Full utility/compression ablation against continuous, projection-only, sign, and random-sign baselines.
3. Reconstruction-privacy evaluation using InverNetV9 with SSIM and LPIPS across all 11 methods.
4. Empirical evidence that VQ acts as a representation regularizer (silhouette score + t-SNE).

### Related Work

Subsections:

- Vertical federated learning and split representation exchange.
- Communication-efficient VFL.
- Privacy attacks and defenses in VFL.
- Vector/product quantization in FL.
- Multimodal VFL fusion and HybridVFL.
- Skin-lesion datasets and multimodal dermatology.

### Method

Subsections:

- VFL setup and notation.
- Passive image encoder and active metadata classifier.
- Product-quantized passive bottleneck.
- Baselines.
- Reconstruction attacker and metrics.

### Experiments

Subsections:

- ISIC-2019 setup.
- Method grid (11 methods, 2 seeds).
- Utility metrics (WACC, tail WACC).
- Privacy metrics (SSIM, LPIPS via InverNetV9).

### Results

Subsections:

- Utility and communication.
- Reconstruction privacy.
- Equal-bit comparison (H_vq_M16 vs S_sign_quant at 128 bits).
- K/M/commitment/init ablations.
- Regularization evidence (silhouette + t-SNE).

### Discussion

Subsections:

- Why VQ may improve utility.
- Why projection-only is an essential control.
- Privacy limits and threat-model limits.
- Relationship to HybridVFL and future private fusion.

---

## 12. Reviewer-Trap Checklist

Before writing claims, verify these:

- [ ] Do not say VQ quantizes the 1280-d embedding; it quantizes the 128-d projection.
- [ ] Do not cite old SOTA notes as if current; HybridVFL exists.
- [x] ~~Do not claim privacy improvement until `A_plain_vfl` and `A_proj_vfl` Stage B are complete.~~ — Stage B done.
- [x] ~~Do not claim learned VQ beats sign on privacy until `H_vq_M16` and `S_sign_quant` Stage B are complete.~~ — Done. Equal-bit SSIM gap is 0.002; frame as utility win, not privacy win.
- [ ] Do not compare HybridVFL numbers directly to your ISIC-2019 numbers because datasets and protocols differ.
- [ ] Do not claim formal privacy or DP.
- [ ] Do not claim robustness to URVFL.
- [ ] Do not claim ISIC-2024/SLICE-3D results until implemented.
- [ ] Do not reference PAD results — those experiments are dropped.

---

## 13. Second-Pass Accuracy Audit

### Checked against local artifacts

- `shazil_v9.ipynb` defines five passive-client families: plain, projection, sign, random sign, VQ.
- `shazil_v9.ipynb` confirms VQ is applied after a learned 128-d projection.
- `shazil_v9.ipynb` defines 11 ISIC methods and 2 seeds.
- `results_final/result_*.json` exists for all 11 methods x 2 seeds.
- `results_final/checkpoints/*.pt` exists for all 11 methods x 2 seeds.
- `results_analysis.md` shows Stage B complete for all 11 methods × 2 seeds at 50 InverNet epochs.
- `results_final/` contains all 22 recon_*.json (SSIM/LPIPS) alongside the 22 result_*.json (WACC).
- `embedding_analysis/` contains silhouette_table.json, silhouette_by_method.pdf/png, tsne_comparison.pdf/png, and individual t-SNE PNGs. Regularization hypothesis confirmed.
- `figures/` contains pareto_wacc_vs_ssim.pdf/png and supporting bits-vs-metric plots.
- PAD external validation dropped; results were poor.

### Checked against primary/external sources

- ISIC-2019 official page confirms 25,331 training images across 8 training categories and balanced multiclass accuracy as the goal metric.
- PAD-UFES-20: dropped from paper — results were poor. Do not cite or reference in the paper.
- SLICE-3D Scientific Data page confirms over 400,000 lesion crops from 3D total-body photography and seven dermatologic centers.
- UIFV arXiv page confirms intermediate feature data inversion in VFL.
- URVFL arXiv/NDSS pages confirm malicious gradient-based reconstruction and NDSS 2025 acceptance.
- HybridVFL arXiv page confirms VFL multimodal classification with client-side disentanglement and server-side cross-modal Transformer fusion on HAM10000.
- FedVQCS and FedMPQ are FL update/uplink compression works, not VFL passive image embedding reconstruction studies.
- ESANN 2024 VQ privacy paper is genuinely adjacent and must be read in full before submission.

### Remaining uncertainty

1. The ESANN 2024 paper may contain details beyond the abstract that require stronger positioning.
2. Clinical meaning of SSIM/LPIPS for dermatology reconstruction still needs either visual grids, expert interpretation, or a carefully worded limitation.
3. A stronger inversion attacker could raise reconstruction quality; the current paper should frame InverNetV9 as an empirical attacker, not as a final privacy proof.
4. ISIC-2024/SLICE-3D is not current evidence — future extension only. PAD-UFES-20 is dropped.

---

## 14. Citation Bank

- VFL background / communication-efficient VFL: [Khan et al. 2022, Algorithms](https://www.mdpi.com/1999-4893/15/8/273)
- Compressed-VFL: [Castiglia et al. 2022, ICML](https://icml.cc/virtual/2022/poster/16395)
- SparseVFL: [Inoue et al. 2023, OpenReview](https://openreview.net/forum?id=BVH3-XCRoN3)
- LESS-VFL: [Castiglia et al. 2023, IBM/ICML](https://research.ibm.com/publications/less-vfl-communication-efficient-feature-selection-for-vertical-federated-learning)
- FedPass: [Gu et al. 2023, IJCAI](https://www.ijcai.org/proceedings/2023/418)
- UIFV: [Yang et al. 2024/2025, arXiv:2406.12588](https://arxiv.org/abs/2406.12588)
- URVFL: [Yao et al. 2024/2025, arXiv:2404.19582](https://arxiv.org/abs/2404.19582)
- Product quantization: [Jegou et al. 2011, IEEE TPAMI](https://doi.org/10.1109/TPAMI.2010.57)
- VQ-VAE / neural discrete representation learning: [van den Oord et al. 2017, NeurIPS](https://papers.neurips.cc/paper/7210-neural-discrete-representation-learning)
- FedVQCS: [Oh et al. 2022/2023, arXiv:2204.07692](https://arxiv.org/abs/2204.07692)
- FedMPQ: [Yang et al. 2024, arXiv:2404.13575](https://arxiv.org/abs/2404.13575)
- ESANN VQ privacy risk: [Schubert and Villmann 2024, ESANN](https://www.esann.org/proceedings/2024), [PDF](https://www.esann.org/sites/default/files/proceedings/2024/ES2024-57.pdf)
- HybridVFL: [Anoosha et al. 2025, arXiv:2512.10701](https://arxiv.org/abs/2512.10701)
- SSIM: [Wang et al. 2004, IEEE TIP / PubMed](https://pubmed.ncbi.nlm.nih.gov/15376593/)
- LPIPS: [Zhang et al. 2018, arXiv:1801.03924](https://arxiv.org/abs/1801.03924)
- ISIC-2019: [Official ISIC 2019 challenge page](https://challenge.isic-archive.com/landing/2019/)
- PAD-UFES-20: [Mendeley Data](https://data.mendeley.com/datasets/zr7vgbcyr2/1), [Data in Brief article](https://pmc.ncbi.nlm.nih.gov/articles/PMC7479321/)
- SLICE-3D / ISIC-2024: [Scientific Data 2024](https://www.nature.com/articles/s41597-024-03743-w), [ISIC 2024 challenge page](https://challenge2024.isic-archive.com/), [npj Digital Medicine 2025 challenge analysis](https://www.nature.com/articles/s41746-025-02070-7)

