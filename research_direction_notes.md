# Research Direction Notes — VFL Medical Privacy Project

---

## On HybridVFL and Multi-Modal Attention

HybridVFL (arXiv 2512.10701, December 2025) does cross-modal attention in VFL:
- Same VFL partition: Image Client (CNN) + Tabular Client (MLP) → Server holds labels
- Same domain: HAM10000 (dermoscopy + metadata, essentially same as ISIC-2019)
- Uses Transformer self-attention on server to fuse image and tabular embeddings
- Balanced accuracy: **0.9235** vs. concatenation baseline of 0.7766
- Explicitly leaves privacy open: "embedding exchanges remain susceptible to inversion attacks — differential privacy/secure computation not implemented"

This means: cross-modal attention in multimodal VFL is covered. The gap HybridVFL leaves, and where the current work fits, is the privacy side.

---

## Has VQ Been Done in VFL Before?

For **horizontal FL gradient compression**: FedVQCS (arXiv 2204.07692) compresses model weight updates via VQ — compressing gradients, not split-layer embeddings. Not VFL.

For **VFL intermediate embedding transmission** with a trained product quantizer codebook, used as a privacy mechanism: no direct prior work found. SparseVFL uses sparsification. Compressed-VFL (ICML 2022) uses scalar quantization (fixed-step rounding). Neither uses a learned VQ codebook.

The specific combination — trained VQ codebook + GRL adversary on the VFL embedding transmission in a medical imaging setting — appears novel.

---

## Full Results Table (from notebook outputs)

| Method | WACC | LR-AUC | MLP-AUC | SSIM | Bits | Notes |
|---|---|---|---|---|---|---|
| A_plain_vfl | 0.7452 | 0.9311 | 0.9500 | 0.7476 | 40960 | 1280-d continuous baseline |
| A_proj_vfl | 0.7586 | N/A | N/A | N/A | 4096 | 128-d projection |
| S_sign_quant | 0.7579 | N/A | N/A | N/A | 128 | 1 bit/dim sign |
| H_vq_small | 0.7698 | 0.9457 | 0.9458 | 0.4222 | 48 | PQ K=64, no GRL |
| H_vq_medium | 0.7567 | 0.9377 | 0.9369 | 0.4530 | 64 | PQ K=256, no GRL |
| H_vq_large | 0.7598 | 0.9428 | 0.9405 | 0.4501 | 80 | PQ K=1024, no GRL |
| I_vq_cc_medium | 0.7760 | 0.9406 | N/A | N/A | 64 | Label-aware (not fair) |
| **V8_main (s42)** | ? | **0.8870** | **0.8939** | **0.5663** | 64 | VQ + GRL |
| **V8_main (s43)** | ? | **0.8895** | **0.8992** | **0.5405** | 64 | VQ + GRL |
| V8_no_grl (s42) | ? | 0.9446 | 0.9430 | ? | 64 | VQ only, no GRL |
| V8_grl_low (s42) | ? | 0.9385 | 0.9371 | ? | 64 | GRL λ=0.1 |
| V8_grl_high (s42) | ? | 0.9004 | 0.8979 | ? | 64 | GRL λ=3.0 |
| V8_no_kmeans (s42) | ? | 0.9142 | 0.9218 | ? | 64 | Random codebook init |

---

## Is SSIM 0.7476 → 0.4 Enough for a Paper? Honest Assessment.

**Short answer: not as the only claim, but the actual finding is more interesting than that.**

### The LR AUC problem

LR-AUC in the 90s reflects a fundamental impossibility in adversarial deconfounding. The task loss pushes the passive embedding to be class-discriminative. A class-discriminative embedding is by definition linearly separable by class. GRL tries to fight this but cannot win against the task loss without destroying utility — the two objectives are in direct conflict. GRL cannot suppress a 90% AUC linear attacker while maintaining diagnostic accuracy. This is expected and not a failure of the implementation; it's a known limitation of adversarial representation learning approaches.

VQ cannot fix this either. Codebook indices that produce accurate diagnoses will map class-specific images to class-specific codes, so a linear probe on the indices will recover class information. **Label inference AUC is structurally unsuppressable in this setup without cryptographic methods.**

The right privacy claim for VQ is **reconstruction resistance**, not label inference suppression.

### The actual finding in the numbers

Comparing H_vq_medium (VQ alone, no GRL) vs. V8_main (VQ + GRL):

| | SSIM | LR-AUC |
|---|---|---|
| H_vq_medium (VQ, no GRL) | **0.4530** | 0.9377 |
| V8_main (VQ + GRL) | 0.5405–0.5663 | **0.8870–0.8895** |

GRL trades reconstruction privacy for marginal label inference improvement. Adding GRL makes SSIM go from 0.45 to 0.55 — the representation becomes more reconstructible because the GRL forces the encoder to retain more information (to still be useful for the active client while fighting the adversary).

**This is actually the most interesting result:** VQ alone gives better reconstruction privacy than VQ+GRL, while VQ+GRL gives modestly better label inference resistance. These two privacy objectives are in tension with each other, and the tradeoff is mediated by the GRL strength (λ). This is a non-obvious finding.

### What the reconstruction SSIM numbers mean

| SSIM range | Visual interpretation |
|---|---|
| 0.75 (plain VFL) | Reconstructed image structurally close to original. Lesion type, texture, color largely recoverable. High privacy risk. |
| 0.45 (VQ alone) | Substantial degradation. General color/region preserved but fine texture lost. Lesion type may still be identifiable. |
| 0.55 (VQ + GRL) | Better than plain but worse than VQ alone. |

The jump from 0.75 → 0.42 is real and meaningful as a privacy improvement. The open question is whether 0.42 SSIM actually protects against clinically relevant reconstruction — a human expert might still identify the lesion type from a 0.42 SSIM image of a dermatology lesion.

### What is missing before this is a publishable paper

1. **V8 WACC numbers**: The final table is incomplete — V8_main's utility (WACC) is not shown. If VQ+GRL utility drops significantly below H_vq_medium, the case for GRL collapses entirely.

2. **S_sign_quant privacy numbers**: Sign quantization gets 128 bits (smaller than VQ's 64 bits) at 0.7579 WACC. If S_sign gives similar SSIM reduction to VQ, then the contribution of the learned codebook over naive sign quantization is unclear.

3. **Stronger attacker evaluation**: InverNet at 20 epochs, 64×64. A diffusion-model-based inversion would likely raise SSIM. The privacy claim is only as strong as the attack is strong.

4. **Clinical SSIM threshold**: For dermatology, what SSIM is needed before the reconstruction is medically meaningless? This requires a perceptual study or reference to clinical image quality literature.

5. **Tradeoff curve**: The full privacy-utility Pareto front (varying λ_grl, codebook size K, number of subspaces M) needs to be plotted.

### What level of venue is realistic with the current results

| Venue | Realistic? | What's needed |
|---|---|---|
| NeurIPS / ICML / ICLR main | No | Needs theoretical contribution (information-theoretic bounds, formal privacy guarantee) |
| MICCAI main | Possible | Needs complete results + clinical framing + stronger attack |
| MIDL (Medical Imaging with DL) | Yes, realistic | Empirical + clean framing is enough at MIDL |
| NeurIPS / ICML workshop | Yes | Workshop-appropriate scope |
| MICCAI workshop | Yes | Current results nearly sufficient |

---

## Dynamic Temperature Scaling — What the Seniors Are Working On

**Closest published paper:** "Rethinking the Temperature for Federated Heterogeneous Distillation" (**ReT-FHD, ICML 2025**). Key ideas:
- Multi-level Elastic Temperature: dynamically adjusts distillation intensity across model layers
- Category-Aware Global Temperature Scaling: per-class temperature based on confidence distributions
- Applied in heterogeneous FL where clients have different model architectures and data distributions

**Other relevant work:**
- "Improving Local Training in FL via Temperature Scaling" (arXiv 2401.09986): "Logit Chilling" — low temperature (0–1) during local training speeds convergence and improves accuracy in non-IID settings, up to 6× faster convergence
- Sentinel (IEEE 2025): Bidirectional knowledge distillation with adaptive temperature for heterogeneous IoT FL

The field is active, ICML 2025 is the top result, and the seniors' work is in the right area.

---

## Connection Between Current Work and Temperature Scaling (The Bridge)

In VFL with Product Quantization, there is an implicit "temperature" in the codebook assignment:

- **Hard assignment** (commitment weight → ∞): Each embedding maps to exactly one code. Maximum compression, maximum information loss, best reconstruction privacy.
- **Soft assignment** (commitment weight → 0): Each embedding is a weighted sum of all codes. Less compression, less information loss, worse reconstruction privacy but better utility.

The **VQ commitment weight** is mathematically analogous to the inverse of distillation temperature:
- Low temperature in distillation = sharper, more confident knowledge transfer
- High commitment weight in VQ = sharper, more discrete code assignment

In a **heterogeneous VFL setting** where the imaging party and the tabular/metadata party have very different data distributions (class imbalance differs between modalities, different feature statistics), the optimal commitment weight for each party may differ. Dynamically adapting the VQ commitment weight based on:
- The confidence distribution of each party's local encoder output
- The cross-party gradient signal (how much the active client benefits from each code)
- The training phase (annealing from soft to hard assignment)

...is a direct analog of per-client, per-class dynamic temperature scaling in heterogeneous FL knowledge distillation.

This is the bridge between the current VQ-based VFL privacy work and the seniors' temperature scaling project.

---

## Year-Long Project Roadmap

| Phase | Work | Estimated Duration |
|---|---|---|
| **Phase 1 (now)** | Fix V8 results: get V8 WACC, get S_sign privacy numbers, complete table, reframe claim as reconstruction privacy via VQ | 1–2 months |
| **Phase 2** | Adopt cross-modal attention (HybridVFL-style) + VQ as privacy layer. Replicate HybridVFL utility on HAM10000/ISIC, add InverNet evaluation. This is "private HybridVFL." | 2–3 months |
| **Phase 3** | Dynamic VQ temperature: adapt commitment weight per modality/party based on heterogeneity. Connect formally to ReT-FHD (ICML 2025). Run ablations. | 3–4 months |
| **Phase 4** | Scale to a second medical dataset with genuine vertical partition: TCGA (imaging + genomics split across two parties) or MIMIC (vitals + notes + labs) | 2–3 months |
| **Phase 5 (parallel)** | Information-theoretic analysis: bound reconstruction fidelity as function of codebook size K and commitment temperature | throughout |

**Supervisor pitch:** "We established that product quantization in VFL provides a favorable reconstruction-privacy/utility tradeoff on medical dermoscopy data, and we identified an unexplored connection between VQ commitment temperature and dynamic temperature scaling in heterogeneous FL. We propose extending this to a dynamic, per-modality temperature mechanism for privacy-aware multimodal VFL, with formal bounds and validation on multi-modal clinical datasets."

---

## Critical Missing Experiments (Do These Next)

1. Run S_sign_quant through the full privacy evaluation (InverNet SSIM + LR/MLP AUC) — currently missing from the table
2. Get V8_main WACC (utility) — the table shows it was trained but WACC isn't in the final output shown
3. Run InverNet on H_vq_small (VQ alone, no GRL) — shows whether GRL helps or hurts reconstruction privacy specifically
4. Plot the full Pareto curve: WACC vs. SSIM across all methods — this is the central figure of the paper
5. Try one stronger attacker (increase InverNet epochs to 50 or add a stronger decoder) — validates that SSIM 0.42 holds against a more determined adversary
