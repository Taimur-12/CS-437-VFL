# VFL in Medicine: True Novelty Map (2019–2026)

> **What this document is:** A novelty map — not a venue gap map. For each direction, it identifies what has been done *anywhere* (arXiv, workshops, medical journals, non-DL venues) and what specific DL contribution survives as genuinely unaddressed after that full accounting. Venue absence is not proof of novelty. A direction with zero NeurIPS papers can still be exhausted if arXiv or medical journals have already executed the core idea.

---

## Novelty Tiers

| Tier | Meaning |
|---|---|
| 🔵 **Uncharted** | No meaningful prior work found at any venue. First-mover. A paper here would be defining the problem, not competing with anyone. |
| 🟢 **Genuine gap** | Prior work covers the general setting but the specific DL mechanism — the architecture, training objective, or attack model — required for the medical VFL instantiation is not done. Clear differentiation is possible. |
| 🟡 **Thin gap** | The core idea exists in prior work. A paper here needs to introduce a meaningfully different mechanism, not just apply the known approach to medical data. Risk of being seen as incremental. |
| 🔴 **False gap** | Appears open at DL venues but is already covered in arXiv or non-DL venues. A DL conference reviewer with broad knowledge would reject for insufficient novelty. |

---

## The Core Principle Applied in This Document

For each direction, the question is not:
> *"Is there a NeurIPS paper on this?"*

The question is:
> *"Has the specific DL contribution I am planning — the training objective, the model architecture, the attack mechanism — been executed anywhere, by anyone, in any form?"*

If the answer is yes (even arXiv preprint, even medical journal, even workshop), you do not have novelty in that specific contribution. You need to find the part that remains undone.

---

## Direction-by-Direction Analysis

---

### 1. VFL Embedding Inversion Attacks on Medical Data

**What exists everywhere (the full prior work record):**
- **CAFE (NeurIPS 2021):** The foundational VFL-specific embedding inversion paper. Shows that intermediate representations in VFL can be inverted to reconstruct raw training data. Tested on MNIST, CIFAR, NUS-WIDE — not medical data.
- **GradInvDiff (MICCAI 2025):** Diffusion model + gradient matching for reconstructing medical images in federated learning. **Critical:** This is horizontal FL gradient inversion, not VFL embedding inversion. These are different attack surfaces.
- **"Defending Against Gradient Inversion for Biomedical Images" (arXiv 2503.16542):** Defense for horizontal FL, not VFL split architecture.
- **"Stealing Medical Privacy via Diffusion-based Gradient Inversion" (MICCAI 2025):** Again horizontal FL.
- Multiple other gradient inversion papers for horizontal FL medical imaging.

**The architectural distinction that matters:**
In horizontal FL, the attacker reconstructs training data from model weight gradients shared during aggregation. In VFL, the attacker at Party B sees the intermediate embedding that Party A's encoder transmits across the split layer — a different signal, different dimensionality, different information density. CAFE's inversion attack is specific to this embedding transmission. The medical imaging inversion papers address horizontal FL gradients, which is technically different.

**What remains after all prior work:**
Inversion attacks targeting the split-layer embedding transmission in VFL applied to medical data specifically. Key questions no paper has answered: Does CAFE's inversion fidelity degrade or improve on high-dimensional medical imaging embeddings (ViT patch tokens, CNN feature maps from pathology slides) vs. natural image embeddings? Can clinical tabular embeddings (lab values, vital signs, encoded by an EHR transformer) be inverted — and does the structured, non-IID nature of clinical data make them more or less recoverable? Does the domain knowledge an adversarial hospital party has about disease ontologies improve reconstruction quality?

**Why this is technically distinct from existing work:** The attack mechanism operates on the embedding space, not the gradient space. The design of an inversion network for medical embeddings (which have different statistical structure from natural image embeddings) requires new architecture choices.

**Killer prior work risk:** If CAFE's authors or follow-ups released a medical-specific version on arXiv, this gap closes. Check arXiv for "CAFE medical" or "VFL embedding inversion healthcare" before claiming this.

| Metric | Assessment |
|---|---|
| **Novelty tier** | 🟢 **Genuine gap** |
| **Nearest prior work** | CAFE (NeurIPS 2021) for VFL mechanism; GradInvDiff (MICCAI 2025) for medical but horizontal FL |
| **Specific unclaimed contribution** | Inversion network for medical split-layer embeddings (imaging, EHR, clinical text encodings) in VFL architecture |
| **Strength of differentiation** | Medium-high — technical difference is real, but requires explicit justification against both CAFE and GradInvDiff |

---

### 2. Label Inference Under Medical Label Distributions

**What exists everywhere:**
- **USENIX Security 2022 (Fu et al.):** Foundational label inference attack in VFL. Mentions medical diagnosis as a threat scenario but does all experiments on non-medical benchmarks (Purchase, Adult, CIFAR).
- **LEA (arXiv 2603.03777, 2026):** Label Enumeration Attack — clustering-based label recovery across VFL scenarios, no auxiliary data needed. General, no medical focus.
- **"Revisiting Label Inference Attacks in VFL" (arXiv 2603.18680, 2026):** Analyzes why VFL is vulnerable; proposes defenses. General, no medical focus.
- **VTarbel (arXiv 2507.14625):** Targeted label attack requiring minimal knowledge. General.
- **Multiple defense papers (MDPI Electronics 2024, ResearchGate 2024):** Multi-level defense strategies. General.

**What remains after all prior work:**
The attack and defense mechanisms are well-established for balanced binary or multi-class classification. Medical labels have distinct statistical properties that are not studied: highly imbalanced class distributions (rare diseases may have 1% prevalence), multi-label targets (a patient has multiple ICD codes simultaneously), survival outcomes (continuous-time labels with censoring), hierarchical label structures (ICD code hierarchy). The specific question — how does attack success rate scale as a function of label imbalance and label cardinality, and does a defense that works for balanced targets fail for rare conditions — is open.

**The DL novelty required:** This is not just an evaluation paper. The novelty would be in designing a defense mechanism whose theoretical privacy guarantee is parameterized by the class imbalance ratio — a guarantee that standard DP-based defenses do not provide. Proving that existing defenses fail catastrophically for rare disease labels (because rarity leaks information even with DP) and designing a defense that compensates is a real DL contribution.

**Killer prior work risk:** High. If someone has studied this for medical data specifically on arXiv, the gap is closed. The general attack/defense is dense enough that a purely empirical evaluation on MIMIC would be rejected at most DL venues.

| Metric | Assessment |
|---|---|
| **Novelty tier** | 🟡 **Thin gap** |
| **Nearest prior work** | USENIX 2022 (mechanism), arXiv 2603.03777 (2026 follow-up) |
| **Specific unclaimed contribution** | Defense with theoretical guarantees under skewed medical label distributions |
| **Strength of differentiation** | Medium — needs new mechanism, not just medical evaluation |

---

### 3. Semi-Supervised / Self-Supervised VFL for Medicine

**What exists everywhere (update: this space is more crowded than previously assessed):**
- **One-shot VFL / ICCV 2023:** Semi-supervised VFL for limited overlap, tested on CIFAR. Achieves 46.5% accuracy gain.
- **LASER-VFL (ICLR 2025):** *Vertical Federated Learning with Missing Features During Training and Inference.* This is a major ICLR 2025 paper that directly tackles the case where feature partitions are missing at training and inference. Flexible to varying sets of feature blocks. **This ate significant space in this direction.**
- **APC-VFL (ScienceDirect 2026):** Single-round VFL using unsupervised representation learning + knowledge distillation. Tested explicitly on MIMIC-III, achieving 634× communication reduction. **Medical data, VFL-specific, uses representation learning.**
- **VFL Knowledge Transfer via Representation Distillation for Healthcare (WWW 2023 / arXiv 2302.05675):** VFL representation distillation specifically for healthcare collaboration networks. Experiments on real medical datasets. **This is a directly relevant prior work that covers the core of "self-supervised VFL for medicine."**
- **FedCVT (ACM TIST 2022), FedHSSL (arXiv 2023):** Semi-supervised VFL general methods.

**What remains after all prior work:**
The general SSL/semi-supervised VFL mechanism and the medical deployment (via APC-VFL and VFL Knowledge Transfer) are covered. The unclaimed territory is a contrastive pre-training objective that exploits the structure of medical modalities specifically — not just general representation alignment but objectives that use clinical knowledge: e.g., a contrastive loss that uses disease labels as hard negative constraints (patients with the same disease but different stage should be close, patients with different diseases should be far), or augmentations that are clinically meaningful (for imaging: realistic pathological transformations, not random crops; for EHR: masking clinically correlated features). The novelty claim must be in the objective, not just the framework.

**What is NOT novel:** "We apply SSL to VFL on medical data." APC-VFL and VFL Knowledge Transfer already do this. The paper needs a genuinely new training objective.

| Metric | Assessment |
|---|---|
| **Novelty tier** | 🟡 **Thin gap** |
| **Nearest prior work** | LASER-VFL (ICLR 2025), APC-VFL (ScienceDirect 2026), WWW 2023 healthcare VFL representation learning |
| **Specific unclaimed contribution** | Contrastive objective using clinical domain structure (disease taxonomy, survival ordering) as supervision signal in VFL pre-training |
| **Strength of differentiation** | Low-medium — must be a new objective with theoretical motivation, not just application |

---

### 4. Backdoor Attacks in Medical VFL — Clinically Targeted Poisoning

**What exists everywhere:**
- **Villain (USENIX Security 2023):** Backdoor in split learning / VFL, no label access needed.
- **BadVFL (ECML PKDD 2023):** 93%+ attack success at 1% poisoning rate in VFL.
- **Constructing Adversarial Examples for VFL (ICLR 2024):** Formal attack framework using multi-armed bandit optimization.
- **Cooperative Decentralized Backdoor Attacks on VFL (arXiv 2501.09320, 2025):** New 2025 paper. Multi-adversary, no server gradient access, models potential collusion among multiple parties. This is the most advanced VFL backdoor paper as of 2025.
- **Universal Backdoor Defense via Label Consistency (IJCAI 2025):** Defense.

**What remains after all prior work:**
All existing backdoor papers use trigger-based attacks where the trigger is a synthetic artifact (pixel pattern, feature vector perturbation) chosen to maximize attack success rate. The medical threat model is distinct: a clinically realistic trigger that is specific to a patient subgroup — a backdoor that causes the model to misdiagnose a specific demographic (patients over 65, patients from a specific geographic region, patients with a comorbidity) without relying on obvious synthetic patterns. This requires designing a trigger that (a) is naturally occurring in the medical feature space, (b) is specific to a clinically relevant subgroup, and (c) cannot be detected by standard trigger detection methods that look for out-of-distribution features. The cooperative paper (arXiv 2501.09320) is the nearest prior work and it models collusion, but it does not study clinically realistic triggers or subgroup targeting.

**Why this requires new DL mechanisms:** Designing a clinically-structured trigger requires solving the problem of finding a feature perturbation that (a) stays within the clinical data manifold and (b) activates the backdoor. This is a constrained optimization problem that existing backdoor papers do not solve.

| Metric | Assessment |
|---|---|
| **Novelty tier** | 🟢 **Genuine gap** |
| **Nearest prior work** | arXiv 2501.09320 (2025 cooperative backdoor), ICLR 2024 adversarial VFL |
| **Specific unclaimed contribution** | On-manifold clinically-structured trigger design for subgroup-targeted misdiagnosis in medical VFL |
| **Strength of differentiation** | High — the clinical structure constraint is a genuinely new technical requirement |

---

### 5. Cross-Modal VFL Architecture — The Fusion Problem

**What exists everywhere:**
- **FedMM (arXiv 2402.15858, 2024):** Multi-modal VFL for computational pathology (H&E histology + genomics). Uses late fusion (concatenation of embeddings from each party). The central architectural contribution is allowing parties with only one modality to participate. Published on arXiv, not at a DL venue.
- **QAVFL (MDPI Brain Sciences, 2025):** VFL with clinical text + fundus images + biosignals for glaucoma. Medical journal.
- **Cross-modal VFL for MRI (PubMed 2024):** Medical journal.
- **Many multi-modal FL papers with missing modality (arXiv 2024–2025):** FedRecon, FedMAC, CAR-MFL — but ALL of these are horizontal FL (different hospitals miss different modalities). Architecturally different from VFL.

**The architectural distinction that is genuinely unaddressed:**
FedMM does late fusion — Party A trains an encoder, Party B trains an encoder, their outputs are concatenated and fed to a classifier. This is the simplest possible fusion. It does not capture cross-modal interactions: the fact that a genomic feature might modify how an imaging feature should be interpreted. Cross-attention between Party A's embedding and Party B's embedding, computed under the constraint that raw features cannot cross party boundaries, is NOT solved in FedMM. The specific challenge: standard cross-attention requires the query, key, and value to come from both modalities. In VFL, the split layer constraint means Party B computes its keys/values locally from features Party A cannot see. Designing cross-modal attention that respects this split while still computing meaningful cross-modal interactions is a genuine architecture problem.

**What is NOT novel:** "We use VFL to train a model on imaging + genomics data." FedMM already does this for pathology. The novelty must be in the fusion mechanism.

| Metric | Assessment |
|---|---|
| **Novelty tier** | 🟢 **Genuine gap** |
| **Nearest prior work** | FedMM (arXiv 2402.15858) for VFL multi-modal; horizontal FL multi-modal papers for missing modality |
| **Specific unclaimed contribution** | Cross-party cross-modal attention respecting VFL split architecture, with missing-modality robustness at inference |
| **Strength of differentiation** | High — technical gap is real, requires novel attention mechanism design |

---

### 6. Graph Neural Networks for Medical Patient Graphs in VFL

**What exists everywhere:**
- **VFGNN (IJCAI 2022):** The only VFL-specific GNN paper. Vertically federated GNN for node classification. General, not medical. Party A holds some node features, Party B holds others.
- **Vertical Federated GNN for Recommendation (ICML 2023):** General, not medical.
- **GNN for Heart Failure Prediction on Patient Similarity Graph (arXiv 2411.19742, 2024):** Patient similarity GNN. BUT this is horizontal FL — the GNN is trained across hospitals that each have full graphs with their own patients.
- **Federated GNN for Disease Classification with Human-in-the-Loop (Scientific Reports 2024):** Horizontal FL.

**What remains after all prior work:**
VFL-specific patient graph learning has not been done in medicine. The setup: patients are nodes, edges are clinical similarity measures (computed from shared electronic records or treatment patterns), and node features are vertically partitioned — Party A (imaging center) holds image-derived features for each patient-node, Party B (genomics lab) holds molecular features for the same patient-nodes. VFGNN is the algorithmic template, but applying it to a patient graph in medicine introduces new technical problems: how to compute edges when node features are split, how to design message passing that respects the VFL constraint, and how to handle the missing-patient-overlap issue (some patient-nodes exist in only one party's dataset).

**Killer prior work risk:** Low. No paper found at any venue addresses VFL GNN for medical patient graphs. Confirm before submitting.

| Metric | Assessment |
|---|---|
| **Novelty tier** | 🟢 **Genuine gap** |
| **Nearest prior work** | VFGNN (IJCAI 2022) for VFL GNN mechanism; arXiv 2411.19742 for medical patient GNN but horizontal FL |
| **Specific unclaimed contribution** | VFL GNN on patient similarity graphs with vertically partitioned node features (imaging, omics, EHR split across parties) |
| **Strength of differentiation** | High — combines two relatively sparse areas, no direct prior work |

---

### 7. VFL-Specific Fairness in Medical Diagnostics

**What exists everywhere:**
- **FairVFL (NeurIPS 2022):** The only VFL-specific fairness paper at any top venue. Contrastive adversarial training to enforce equitable representations. General (not medical), uses Criteo and other non-clinical benchmarks.
- **FlexFair (Nature Communications, 2025):** Fairness in federated medical imaging. Multiple fairness criteria (demographic parity, equal opportunity). **BUT: this is horizontal FL, not VFL.** In FlexFair, different hospitals train on the same type of features for different patients.
- **FedIDA (arXiv 2505.09295, 2025):** FL for demographic disparities + data imbalance. Cardiac arrest data. **Horizontal FL.**
- **"Improving Fairness in AI on EHR via FL" (arXiv 2305.11386):** Horizontal FL.

**The VFL-specific fairness problem that no paper solves:**
In horizontal FL, the party that holds demographic attributes (race, sex, insurance type) is the same party that holds the medical labels. Applying fairness constraints is straightforward: you train with an adversarial debiasing objective that sees both the demographic attribute and the label.

In VFL, demographic attributes might live at a different party than the medical label. Hospital A holds imaging features. Hospital B holds EHR features that include demographics. Hospital C holds the diagnosis labels. Hospital A and C can never see Hospital B's demographics directly. Standard fairness constraints that require the model to see both sensitive attributes and labels simultaneously cannot be applied in this architecture. This is a genuinely new constraint that no fairness paper has addressed. The novelty is in designing a fairness objective that can be enforced in this split setting — either via adversarial training with privacy-preserving demographic surrogates or via a constrained optimization on the embedding space that provably bounds demographic influence without revealing the demographic features.

| Metric | Assessment |
|---|---|
| **Novelty tier** | 🟢 **Genuine gap** |
| **Nearest prior work** | FairVFL (NeurIPS 2022) for VFL fairness mechanism; FlexFair (Nature Comm. 2025) for medical FL fairness but horizontal |
| **Specific unclaimed contribution** | Fairness objective for VFL where demographic attributes and medical labels are held by different parties |
| **Strength of differentiation** | High — the split-demographic-attribute problem is architecturally unique to VFL |

---

### 8. VFL + Foundation Models / LLMs for Clinical Text

**What exists everywhere:**
- All LLM + federated learning medical work is horizontal FL: LoRA-based fine-tuning, layer-skipping FL (arXiv 2504.10536), FedMRG for report generation (arXiv 2506.17562).
- "Federated Large Language Models: Current Progress and Future Directions" (arXiv 2409.15723): Survey, entirely horizontal FL.
- "Input Reconstruction Attack Against Vertical FL LLMs" (arXiv 2023): Studies the attack on VFL with LLM backbones, but is an attack paper not an architecture/training paper.
- No paper designs or validates a VFL training procedure for LLMs in clinical settings.

**What remains:**
First-mover territory. The specific technical problem: which transformer layers should be the split layer? The early layers of clinical LLMs extract surface-level text patterns; later layers encode semantic content. In VFL where Party A has radiology reports and Party B has nursing notes, the split should preserve semantic alignment without exposing raw text. Second, how should gradients flow across the split in a transformer's self-attention mechanism without leaking the full attention pattern (which can reconstruct input tokens)? Third, what cross-party objective allows the jointly-fine-tuned model to outperform two separately fine-tuned models on clinical NLP downstream tasks (ICD coding, de-identification, clinical NER)?

**DL novelty type:** Transformer split architecture design, gradient alignment for clinical LLMs in VFL, cross-party attention without attention pattern leakage.

| Metric | Assessment |
|---|---|
| **Novelty tier** | 🔵 **Uncharted** |
| **Nearest prior work** | arXiv 2023 VFL LLM attack paper (attack only); horizontal FL LLM papers (different architecture) |
| **Specific unclaimed contribution** | Split-layer design for transformer/LLM fine-tuning in VFL; clinical NLP fine-tuning objective under VFL constraints |
| **Strength of differentiation** | Very high — first-mover |

---

### 9. Privacy-Preserving Cross-Party Attribution in Medical VFL

**What exists everywhere:**
- "Interplay between FL and XAI: A Scoping Review" (arXiv 2411.05874, 2024): Broad review of FL + explainability. Notes that "few studies quantify the impact of FL on explanations." Mostly horizontal FL.
- "Interpretable VFL for Cancer Prognosis" (ScienceDirect 2025): Adds an interpretability module to VFL cancer prognosis. Medical journal. Does not address the privacy-attribution tension.
- FairVFL (NeurIPS 2022): Uses some interpretability components but not for cross-party attribution.
- Federated SHAP papers: compute SHAP values in horizontal FL settings. Not VFL-specific.

**The specific problem no paper solves:**
In VFL, a physician wants to know: "For this patient's cancer risk prediction, how much did Hospital B's genomic data contribute versus Hospital A's imaging data?" Computing this attribution requires the model to reveal gradient or feature importance signals that could partially reconstruct Hospital B's features. Standard Shapley values require evaluating the model on all subsets of features — in VFL, subsets of features can only be evaluated with the cooperation of specific parties, and the evaluation itself leaks marginal information about each party's features.

No paper has designed a Shapley-based or gradient-based attribution method for VFL that (a) produces clinically interpretable attributions and (b) provides a formal privacy guarantee on the attribution signal itself. This is a unique intersection of explainability and VFL-specific privacy.

| Metric | Assessment |
|---|---|
| **Novelty tier** | 🔵 **Uncharted** |
| **Nearest prior work** | arXiv 2411.05874 (review), ScienceDirect 2025 (interpretation in VFL but no privacy guarantee on attribution) |
| **Specific unclaimed contribution** | Privacy-preserving Shapley attribution in VFL with formal bound on information leakage via attribution |
| **Strength of differentiation** | Very high — problem not formalized anywhere |

---

### 10. Medical VFL Benchmark

**What exists everywhere:**
- VFLAIR (ICLR 2024): VFL research library with attack/defense support. General, not medical.
- VertiBench (ICLR 2024): Evaluates feature distribution diversity in VFL. General, not medical.
- "Vertical FL in Practice: The Good, the Bad, and the Ugly" (arXiv 2502.08160, 2025): Identifies that common practical VFL scenarios lack viable solutions. Proposes a new taxonomy but is a position/analysis paper, not a benchmark.
- No paper creates a medical-specific VFL benchmark with genuinely vertically partitioned data.

**What remains:**
A benchmark paper that curates or constructs paired datasets where features are naturally (not artificially) vertically partitioned — e.g., TCGA imaging + genomics, where the pathology lab and genomics core genuinely hold different feature types on the same tumor samples; or MIMIC-III where ICU monitoring (time series), lab results, and clinical notes are held by operationally different hospital departments. The contribution is: (a) defining the realistic vertical partition, (b) characterizing the patient overlap distribution, (c) providing standardized splits and evaluation protocols, and (d) benchmarking existing VFL methods (LASER-VFL, APC-VFL, FedCVT, VFGNN) on the medical data.

This has no meaningful competition at any venue.

| Metric | Assessment |
|---|---|
| **Novelty tier** | 🔵 **Uncharted** |
| **Nearest prior work** | VFLAIR/VertiBench (ICLR 2024) — general VFL only |
| **Specific unclaimed contribution** | Medical VFL benchmark with naturally partitioned datasets, realistic overlap, and standardized evaluation |
| **Strength of differentiation** | Very high — no competition |

---

### 11. Multi-Omics VFL with Deep Encoders

**What exists everywhere:**
- **AFEI (2023):** Adaptive VFL for heterogeneous multi-omics integration. Uses ensemble methods, not deep learning encoders specifically.
- **Domain-aware priors for VFL in genomics (arXiv 2601.00050, 2025):** Incorporates domain knowledge (gene pathways, co-expression networks) as priors in VFL genomics. Uses graph-structured priors. Partially addresses the DL aspect.
- **Federated Multi-Omics for Parkinson's (Cell Patterns, 2024):** Uses standard ML baselines.
- **PLOS Comp Bio (2024):** Multi-omics federated analysis. Statistical ML methods.

**The gap assessment:**
The existing multi-omics VFL papers use relatively shallow models or standard ML methods. No paper designs deep learning encoders specifically suited to each omics type — RNA-seq encoder (sparse, high-dimensional, count data), protein expression encoder (continuous, moderately dimensional), methylation encoder (binary/continuous, genomic-position-indexed) — and then fuses these via cross-party attention in a VFL architecture. The technical challenge is designing encoders that handle the extreme feature dimensionality (tens of thousands of genes) and the biological structure (gene co-expression, pathway membership) while communicating only low-dimensional embeddings across party boundaries.

**Killer prior work risk:** Medium. Fetch arXiv 2601.00050 and check how deep the architecture is before claiming this direction.

| Metric | Assessment |
|---|---|
| **Novelty tier** | 🟢 **Genuine gap** |
| **Nearest prior work** | arXiv 2601.00050 (domain-aware priors), AFEI (ensemble VFL) |
| **Specific unclaimed contribution** | Omics-type-specific deep encoders (RNA-seq, proteomics, methylation) with cross-party fusion in VFL for cancer subtyping or survival prediction |
| **Strength of differentiation** | Medium-high — depends on how deep arXiv 2601.00050 actually goes |

---

## False Gaps: Directions That Look Open but Are Already Covered

These are directions where the DL landscape doc identified "near-zero DL venue papers" as an opportunity, but full prior work review shows the core idea is addressed in arXiv or non-DL venues. Submitting a paper here to a DL venue without a genuinely new mechanism would likely be rejected for insufficient novelty.

| Direction | Why It Is a False Gap | What Kills the Novelty Claim |
|---|---|---|
| **Semi-supervised VFL generally** | LASER-VFL (ICLR 2025) directly solves missing features during training and inference; APC-VFL (2026) does single-round SSL on MIMIC-III | Two direct papers, one at ICLR |
| **Communication-efficient VFL for hospitals** | ICML 2023 has two dedicated VFL communication papers; NeurIPS 2023 does joint privacy + communication | Dense DL venue coverage already |
| **VFL representation learning for healthcare generally** | VFL Knowledge Transfer via Representation Distillation (WWW 2023 / arXiv 2302.05675) + APC-VFL | Both specifically address healthcare VFL representation learning |
| **Asynchronous VFL** | FedVS (ICML 2023) + Secure Bilevel Async VFL (AAAI 2021) cover the core; medical angle alone is not a DL contribution | No new algorithmic mechanism to add |
| **Gradient inversion defense for medical FL** | arXiv 2503.16542 (defenses for biomedical gradient inversion) + MICCAI 2025 GradInvDiff cover the defense side for horizontal FL | Only a VFL-specific defense (different architecture) would be novel |
| **VFL for drug discovery / QSAR** | MELLODDY is 10 pharma companies with real data; Nature Machine Intelligence 2025; field is industrially established | Pharmaceutical VFL is mature; DL novelty requires generative drug design, not QSAR |

---

## Summary Table: True Novelty Map

| Direction | Novelty Tier | Nearest Prior Work | Unclaimed Contribution | Killer Risk |
|---|---|---|---|---|
| VFL Embedding Inversion on Medical Data | 🟢 Genuine gap | CAFE (NeurIPS 2021), GradInvDiff (MICCAI 2025, but horizontal FL) | Inversion of medical split-layer embeddings (VFL-specific attack surface) | Low — VFL vs. horizontal FL distinction is real |
| Label Inference Under Medical Label Distributions | 🟡 Thin gap | USENIX 2022, arXiv 2603.03777 (2026) | Defense with guarantee under skewed / multi-label clinical targets | High — needs new mechanism or rejected as evaluation |
| Semi-supervised / SSL VFL for Medicine | 🟡 Thin gap | LASER-VFL (ICLR 2025), APC-VFL (2026), WWW 2023 VFL-healthcare | Clinical-domain contrastive objective (not just applying SSL) | High — space more crowded than it appeared |
| Backdoor with Clinically-Structured Triggers | 🟢 Genuine gap | arXiv 2501.09320 (2025 cooperative backdoor) | On-manifold clinical triggers for subgroup-targeted misdiagnosis | Medium — arXiv 2501.09320 is recent and thorough |
| Cross-Modal VFL Fusion Architecture | 🟢 Genuine gap | FedMM (arXiv 2402.15858, late fusion) | Cross-party cross-modal attention respecting VFL split constraint | Medium — FedMM must be clearly distinguished |
| GNN for Medical Patient Graphs in VFL | 🟢 Genuine gap | VFGNN (IJCAI 2022, general), arXiv 2411.19742 (horiz. FL medical) | VFL GNN with vertically partitioned patient node features | Low — direct prior work gap is real |
| VFL Fairness — Split Demographic Attributes | 🟢 Genuine gap | FairVFL (NeurIPS 2022, general), FlexFair (Nature Comm. 2025, horiz.) | Fairness objective when demographics and labels are at different parties | Low — VFL-specific formulation is genuinely new |
| VFL + LLMs for Clinical Text | 🔵 Uncharted | VFL LLM attack paper (arXiv 2023, attack only) | Split-layer design for transformer fine-tuning; clinical NLP VFL objective | Very low — first-mover |
| Privacy-Preserving Cross-Party Attribution | 🔵 Uncharted | arXiv 2411.05874 (review), ScienceDirect 2025 (journal, no privacy bound) | Shapley-in-VFL with formal bound on attribution leakage | Very low — problem not formalized anywhere |
| Medical VFL Benchmark | 🔵 Uncharted | VFLAIR/VertiBench (ICLR 2024, general) | Benchmark with naturally partitioned medical datasets + standardized eval | Very low — confirmed nothing exists |
| Multi-Omics VFL with Deep Encoders | 🟢 Genuine gap | arXiv 2601.00050, AFEI (shallow/ensemble methods) | Omics-specific deep encoders + cross-party fusion for cancer prediction | Medium — fetch arXiv 2601.00050 to verify depth |

---

## Strategic Synthesis

**Highest-confidence novelty claims (low risk of prior work invalidating):**

1. **VFL + LLMs for clinical text** — First mover. No prior VFL LLM architecture paper for medicine. The problem is real, the motivation is clear, the technical challenges are well-defined. Highest-leverage single direction.

2. **Medical VFL benchmark** — Confirmed absence at all venues. A benchmark paper here would be the citation anchor for the field for years.

3. **Privacy-preserving cross-party attribution** — Problem not formalized anywhere. Intersection of two sparse literatures (VFL + XAI) with a specific privacy constraint unique to VFL. High theoretical depth possible.

**Genuine gaps with clear technical differentiation:**

4. **VFL fairness with split demographic attributes** — The formulation of "fairness when demographics and labels are at different parties" is new. FairVFL is the only prior paper and it's general. The medical application adds clinical stakes.

5. **VFL GNN for patient graphs** — Two-part gap: VFL GNN exists (IJCAI 2022) but isn't medical; medical patient GNNs exist but are horizontal FL. Combining both is open. Requires verifying no arXiv paper crosses both.

6. **Cross-modal cross-party attention** — FedMM uses late fusion (concatenation). Designing attention that operates across party boundaries in VFL is architecturally novel. The medical multi-modal case (imaging + genomics + clinical) makes this concrete.

**Directions requiring a new mechanism, not just application:**

7. **Backdoor with clinical triggers** — Mechanism novelty is in designing on-manifold clinically realistic triggers. Must be clearly different from synthetic triggers in existing work. arXiv 2501.09320 (2025) is the closest prior work — study it before building this direction.

8. **VFL embedding inversion on medical data** — Technical distinction from horizontal FL gradient inversion must be made explicit. CAFE is the right comparison point, not GradInvDiff.

9. **Multi-omics with deep encoders** — Check arXiv 2601.00050 depth before claiming. If it uses shallow encoders or simple architectures, the gap for designing omics-specific transformers in VFL is real.

**Proceed with caution — thin gaps:**

10. **Label inference under medical label distributions** — Needs a new defense mechanism, not just evaluation. Without a new algorithm, this is a venue-hopeful evaluation paper.

11. **Semi-supervised / SSL VFL for medicine** — LASER-VFL (ICLR 2025) and APC-VFL (2026) have compressed this space. A new contrastive objective designed around clinical domain knowledge could still work, but differentiation requires care.

---

## What Changed From the Venue Gap Map

| Direction | Old Assessment (Venue Gap) | New Assessment (Novelty Map) |
|---|---|---|
| Semi-supervised VFL | 🔵 Frontier (ICCV 2023 only) | 🟡 Thin gap — LASER-VFL (ICLR 2025), APC-VFL on MIMIC (2026), WWW 2023 healthcare paper all exist |
| Communication-efficient VFL | 🟡 Active | 🔴 False gap for new work — covered by ICML 2023 ×2, NeurIPS 2023; medical angle alone is not enough |
| Self-supervised VFL for medicine | 🔵 Frontier | 🟡 Thin gap — VFL representation distillation for healthcare (WWW 2023) and APC-VFL already do this |
| Backdoor in medical VFL | 🟢 Fertile | 🟢 Genuine gap — but specifically requires clinically-structured triggers, not generic backdoor evaluation on medical data |
| Cross-modal VFL | 🔵 Frontier | 🟢 Genuine gap — FedMM exists on arXiv (late fusion only); the architectural novelty of cross-party attention is real |
| VFL + LLMs | 🔵 Frontier | 🔵 Uncharted — confirmed, no architecture paper anywhere |
| Privacy-preserving attribution | (not in prior doc) | 🔵 Uncharted — novel problem formulation |
| VFL fairness medical | 🟢 Fertile | 🟢 Genuine gap — the specific VFL formulation (split demographics + labels) is new; general FL fairness is covered |

---

## Sources

- [LASER-VFL: Vertical Federated Learning with Missing Features (ICLR 2025)](https://openreview.net/forum?id=OXi1FmHGzz)
- [Vertical FL in Practice: The Good, the Bad, and the Ugly (arXiv 2502.08160)](https://arxiv.org/html/2502.08160v1)
- [Vertical Federated Knowledge Transfer for Healthcare (WWW 2023 / arXiv 2302.05675)](https://arxiv.org/abs/2302.05675)
- [APC-VFL: Active Participant Centric VFL on MIMIC-III (ScienceDirect 2026)](https://www.sciencedirect.com/science/article/abs/pii/S0950705126000778)
- [Cooperative Decentralized Backdoor Attacks on VFL (arXiv 2501.09320)](https://arxiv.org/abs/2501.09320)
- [Revisiting Label Inference Attacks in VFL (arXiv 2603.18680)](https://arxiv.org/abs/2603.18680)
- [LEA: Label Enumeration Attack in VFL (arXiv 2603.03777)](https://arxiv.org/html/2603.03777)
- [GradInvDiff: Diffusion-based Gradient Inversion on Medical FL (MICCAI 2025)](https://papers.miccai.org/miccai-2025/paper/1362_paper.pdf)
- [Defending Against Gradient Inversion for Biomedical Images (arXiv 2503.16542)](https://arxiv.org/html/2503.16542)
- [FedMM: Multi-Modal VFL for Computational Pathology (arXiv 2402.15858)](https://arxiv.org/html/2402.15858v1)
- [Interplay between FL and XAI: Scoping Review (arXiv 2411.05874)](https://arxiv.org/html/2411.05874v1)
- [FairVFL: Fair VFL with Contrastive Adversarial Learning (NeurIPS 2022)](https://arxiv.org/html/2405.17495v1)
- [FlexFair: Flexible Fairness in Federated Medical Imaging (Nature Communications 2025)](https://www.nature.com/articles/s41467-025-58549-0)
- [FedIDA: Federated Learning for Demographic Disparities (arXiv 2505.09295)](https://arxiv.org/abs/2505.09295)
- [CAFE: Catastrophic Data Leakage in VFL (NeurIPS 2021)](https://arxiv.org/abs/2106.05508)
- [VFGNN: Vertically Federated GNN (IJCAI 2022)](https://www.ijcai.org/proceedings/2022/272)
- [Domain-Aware Priors for VFL in Genomics (arXiv 2601.00050)](https://arxiv.org/pdf/2601.00050)
- [VFLAIR: Research Library and Benchmark for VFL (ICLR 2024)](https://openreview.net/forum?id=sqRgz88TM3)
- [VertiBench: Feature Distribution Diversity in VFL (ICLR 2024)](https://iclr.cc/virtual/2024/poster/18132)
- [VFL Survey Repository (ACM Computing Surveys 2025)](https://github.com/shentt67/VFL_Survey)
