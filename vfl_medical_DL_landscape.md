# Vertical Federated Learning in Medicine: A Deep Learning Research Landscape (2019–2025)

> **Scope of this document:** This analysis is filtered specifically for *Deep Learning novelty* — work publishable at DL/ML venues (NeurIPS, ICML, ICLR, ICCV, CVPR, AAAI, IJCAI, MICCAI, KDD, UAI). It excludes directions whose novelty lives in cryptography, clinical medicine, statistics, or systems engineering, and which are therefore routed to security venues (USENIX, CCS), medical journals (Nature Medicine, JMIR), or bioinformatics venues (Bioinformatics, PLOS Comp. Bio.) regardless of how important they are.

---

## The Brutal Reality Check First

From the ACM Computing Survey 2025 on VFL (Wuhan University MARS Group, ~175+ papers):

**Papers at top DL/ML venues (NeurIPS, ICML, ICLR, ICCV, CVPR, AAAI, IJCAI) that are explicitly medical/healthcare: approximately 2–3.**

The intersection of `{DL conference VFL papers}` ∩ `{medical applications}` is almost empty right now. This is not a sign that the space is exhausted — it is a sign that the field hasn't arrived yet. Medical VFL lives mostly in medical journals, applied ML journals, and arXiv. The DL community has built the methodological toolkit for VFL; almost nobody has carried it into medicine with DL-grade novelty.

This is the core opportunity signal.

---

## Classification System

| Metric | What It Measures |
|---|---|
| **DL Venue Papers** | How many papers at NeurIPS/ICML/ICLR/ICCV/CVPR/AAAI/IJCAI/MICCAI exist |
| **Medical DL Papers** | Papers with both DL novelty AND explicit medical application |
| **Trajectory** | Emerging / Rising / Plateauing / Declining |
| **DL Novelty Type** | What kind of DL contribution the direction requires |
| **Opportunity** | 🔴 Crowded · 🟡 Active · 🟢 Fertile · 🔵 Frontier |

**What counts as DL novelty:**
- New model architecture or training objective
- New learning paradigm (self-supervised, semi-supervised, contrastive, generative)
- New attack or defense mechanism using neural networks
- New gradient/embedding-level analysis
- New federated optimization scheme for neural networks

**What does NOT count as DL novelty (and belongs elsewhere):**
- Applying existing VFL to a new medical dataset with no new method
- Privacy via homomorphic encryption / MPC — this is cryptography
- Patient record linkage / PSI — this is systems/cryptography
- Tree-based federated methods (SecureBoost, XGBoost VFL) — not DL
- Clinical deployment studies — medicine/informatics

---

## Direction-by-Direction Analysis

---

### 1. Gradient & Feature Reconstruction Attacks on Medical VFL

**What's been done at DL venues:**
The core attack literature is published at top venues:
- **CAFE** (NeurIPS 2021): *Catastrophic Data Leakage in Vertical Federated Learning* — shows that intermediate embeddings in VFL can be inverted to reconstruct raw training data. This is the most cited attack in VFL and directly threatens medical images and clinical features.
- **Feature Inference Attack on Model Predictions in VFL** (ICDE 2021)
- **Label Inference Attacks Against Vertical FL** (USENIX Security 2022): labels (diagnoses) can be inferred by non-label-holding parties
- **FedPass: Privacy-Preserving VFL with Adaptive Obfuscation** (IJCAI 2023): defense via adaptive perturbation of intermediate representations

**Medical angle that's missing:** CAFE and label inference attacks are demonstrated on generic tabular/image datasets (MNIST, CIFAR, Criteo). No paper has specifically instantiated these attacks against medical embeddings — e.g., can a malicious pharmacy party reconstruct an MRI embedding passed from a hospital? Does reconstruction fidelity differ for medical vs. natural images? Can EHR features (lab values, diagnoses) be recovered? The medical threat model is fundamentally different from the generic one: adversaries have domain knowledge (disease ontologies, ICD codes), data is highly structured, and the stakes (HIPAA violations) are much higher.

| Metric | Status |
|---|---|
| DL Venue Papers | Dense (NeurIPS, USENIX Security, ICDE, IJCAI) |
| Medical DL Papers | Near-zero |
| Trajectory | Rising fast |
| DL Novelty Type | New attack architectures, inversion networks, defense objectives |
| Opportunity | 🟢 **Fertile** |

**What a DL paper here looks like:** Design a gradient inversion network specifically for medical data (EHR tabular, medical image embeddings, clinical text encodings) and show that standard defenses (DP noise) fail to protect medical features at acceptable utility. Or design a medically-aware defense that exploits domain structure.

---

### 2. Label Inference Attacks — The Diagnosis Leakage Problem

**What's been done at DL venues:**
- **Label Leakage and Protection in Two-Party Split Learning** (arXiv 2021, widely cited)
- **Label Inference Attacks Against VFL** (USENIX Security 2022): passive clients can infer the active party's labels from gradients alone
- **Label Leakage in VFL: A Survey** (IJCAI 2024): formalizes all known label leakage vectors
- **ExPloit loss function** (arXiv 2022): neural network to extract labels
- **Defending Batch-Level Label Inference** (IEEE Trans. Big Data 2022)

**Medical angle:** In medical VFL, the label is often the diagnosis (cancer / no cancer, diabetic / non-diabetic). If the pharmacy party or insurance party can infer a patient's diagnosis from gradients — without ever being told it — this is a direct HIPAA violation. The DL novelty here is in the attack models (what architecture recovers labels best?) and defenses (what training objective hides labels without destroying utility?). No paper has specifically studied this under a medical label distribution — rare diseases, multi-class ICD codes, survival labels — which may be qualitatively different from standard classification targets.

| Metric | Status |
|---|---|
| DL Venue Papers | Moderate (USENIX, IJCAI, IEEE Trans.) |
| Medical DL Papers | Sparse (1–2 mentions, no dedicated work) |
| Trajectory | Rising |
| DL Novelty Type | Adversarial inference models, DP-based label protection with utility preservation |
| Opportunity | 🟢 **Fertile** |

**What a DL paper here looks like:** Study how label inference attack success rate scales with medical label characteristics — imbalanced classes, rare conditions, multi-label disease codes. Then design a defense that maintains diagnostic model AUC while provably bounding label leakage, validated on real clinical datasets (MIMIC, TCGA).

---

### 3. Semi-Supervised VFL with Limited Patient Overlap

**What's been done at DL venues:**
- **One-shot VFL** (ICCV 2023): *Communication-Efficient Vertical FL with Limited Overlapping Samples* — tackles the case where only a small fraction of patients appear in all parties. Uses semi-supervised learning on unaligned samples to still train useful representations. Achieves 46.5% accuracy improvement and 330× communication reduction on CIFAR-10.
- **FedCVT** (ACM TIST 2022): Semi-supervised VFL via cross-view training and attention-weighted feature fusion
- **Self-supervised VFL** (NeurIPS Workshop 2022): uses SSL to exploit unaligned samples

**The medical reality:** In real hospital VFL, the patient overlap between two institutions might be 10–30% of each party's total records. In rare disease settings it could be even lower. Current VFL assumes a large, fully aligned dataset — an assumption that almost never holds in medicine. ICCV 2023 is the only DL venue paper attacking this directly, and it's on CIFAR-10 not medical data.

| Metric | Status |
|---|---|
| DL Venue Papers | Sparse (ICCV 2023, NeurIPS Workshop 2022) |
| Medical DL Papers | Near-zero |
| Trajectory | Emerging |
| DL Novelty Type | SSL/semi-supervised training objectives, cross-view alignment, pseudo-labeling in VFL setting |
| Opportunity | 🔵 **Frontier** |

**What a DL paper here looks like:** Extend one-shot VFL or FedCVT to medical settings where partial overlap is the norm (e.g., 20% of EHR patients also have genomic records). Design a training objective that learns from unaligned medical records using domain-appropriate augmentations (clinical text masking, imaging crop consistency). Validate on survival prediction or cancer subtyping.

---

### 4. Backdoor & Adversarial Attacks in VFL — Medical Safety Threat

**What's been done at DL venues:**
- **Constructing Adversarial Examples for VFL** (ICLR 2024): optimal client corruption via multi-armed bandit — formal attack framework
- **Villain: Backdoor Attacks Against Vertical Split Learning** (USENIX Security 2023)
- **Practical and General Backdoor Attacks Against VFL** (ECML PKDD 2023): BadVFL achieves 93%+ attack success rate at 1% poisoning
- **Universal Backdoor Defense via Label Consistency in VFL** (IJCAI 2025)
- **Universal Adversarial Backdoor to Fool VFL** (Computers & Security 2024)

**Medical safety angle:** A backdoor in a medical VFL model could cause misclassification of specific patient subgroups — e.g., making a cancer detection model fail on patients from a certain demographic. The attacker only controls one party's features and cannot touch the label, making medical VFL backdoors mechanistically different from standard backdoor attacks. No paper studies this in a medical context where the "trigger" could be a specific demographic feature, a rare biomarker pattern, or an adversarially perturbed imaging feature.

| Metric | Status |
|---|---|
| DL Venue Papers | Moderate (ICLR, USENIX, ECML PKDD, IJCAI) |
| Medical DL Papers | Near-zero |
| Trajectory | Rising |
| DL Novelty Type | Adversarial trigger design, poisoning under no-label-access constraint, robust training |
| Opportunity | 🟢 **Fertile** |

**What a DL paper here looks like:** Design a backdoor attack specific to medical VFL — where the attacker controls radiology features but not labels — and show it can selectively cause misdiagnosis. Then propose a detection/defense mechanism. MICCAI or NeurIPS would both be natural venues.

---

### 5. Communication-Efficient VFL via DL Compression

**What's been done at DL venues:**
- **Compressed-VFL** (ICML 2022): communication-efficient VFL via compressed embeddings, first principled compression method for VFL
- **One-shot VFL** (ICCV 2023): 330× communication reduction
- **A Unified Solution for Privacy and Communication Efficiency in VFL** (NeurIPS 2023): joint optimization of privacy + communication
- **FedVS: Straggler-Resilient and Privacy-Preserving VFL for Split Models** (ICML 2023): handles slow/dropping clients
- **LESS-VFL** (ICML 2023): communication-efficient feature selection via gradient sparsification

**Medical angle:** Hospital networks have heterogeneous bandwidth — academic medical centers vs. rural hospitals vs. cloud-connected clinics. The straggler problem (ICML 2023 FedVS) is directly relevant when hospitals have different IT infrastructure. No paper studies compression-communication tradeoffs under realistic hospital network constraints or validates on medical datasets where embedding dimensionality is determined by model architecture (ViT vs. CNN vs. transformer-based EHR encoder).

| Metric | Status |
|---|---|
| DL Venue Papers | Dense (ICML x2, NeurIPS, ICCV) |
| Medical DL Papers | Sparse |
| Trajectory | Rising |
| DL Novelty Type | Compressed embeddings, gradient sparsification, straggler-robust optimization |
| Opportunity | 🟡 **Active** |

**What a DL paper here looks like:** The general methods exist. The opening for a medical paper is: study how the compression-utility tradeoff changes under medical data specifics — small cohorts, high-dimensional imaging embeddings, structured clinical text — and propose a compression scheme that provably maintains diagnostic accuracy guarantees. MICCAI or a medical AI track at NeurIPS.

---

### 6. Multi-Modal VFL Architecture for Medicine

**What's been done at DL venues:**
- **FedMM** (arXiv 2402.15858, 2024): Federated Multi-Modal Learning with Modality Heterogeneity in Computational Pathology — allows clients with only one modality to participate in training a multi-modal model. Medical framing, but on arXiv not a DL conference yet.
- **Cross-Modal Vertical FL for MRI Reconstruction** (PubMed 38294925, 2024): medical journal
- **QAVFL for Glaucoma** (MDPI Brain Sciences, 2025): VFL with clinical text + fundus images + biosignals — medical journal

These papers have the right framing but haven't been submitted to DL venues. The technical contribution — how to fuse embeddings from different modalities held by different parties, with each party training its own encoder — is a DL architecture problem.

| Metric | Status |
|---|---|
| DL Venue Papers | Near-zero (arXiv only) |
| Medical DL Papers | Sparse (medical journals) |
| Trajectory | Emerging |
| DL Novelty Type | Cross-modal attention, modality-missing robust fusion, heterogeneous encoder aggregation |
| Opportunity | 🔵 **Frontier** |

**What a DL paper here looks like:** Design a VFL framework where Party A has MRI encodings and Party B has genomic feature encodings for the same patients. Train cross-modal attention to fuse representations without transferring raw data. Handle missing modality at inference time. Submit to MICCAI or ICLR. No one has done this cleanly at a DL venue.

---

### 7. Graph Neural Networks for Medical VFL

**What's been done at DL venues:**
- **VFGNN: Vertically Federated GNN for Privacy-Preserving Node Classification** (IJCAI 2022): nodes are split — one party has some node features, another party has others, the graph structure may be shared or split. Fully DL, published at IJCAI.
- **Vertical Federated Graph Neural Network for Recommender System** (ICML 2023): GNN in VFL setting for recommendation
- **FedNI: Federated Graph Learning with Network Inpainting for Disease Prediction** (IEEE TMI 2023): not strictly VFL but adjacent

**Medical angle:** Patient similarity graphs are a natural structure in medicine — patients as nodes, edges as clinical similarity, features split across parties (imaging features at one party, EHR features at another). The VFGNN paper (IJCAI 2022) is the template for this but is not medical. No one has applied vertical GNN federated learning to a medical patient graph — EHR-based patient networks, protein-protein interaction graphs with multi-omics, or pathology slide graphs.

| Metric | Status |
|---|---|
| DL Venue Papers | Moderate (IJCAI 2022, ICML 2023) |
| Medical DL Papers | Near-zero |
| Trajectory | Rising |
| DL Novelty Type | Graph neural network architecture for vertically partitioned node/edge features |
| Opportunity | 🟢 **Fertile** |

**What a DL paper here looks like:** Extend VFGNN to patient similarity networks in medicine — one party holds imaging-based patient similarities, another holds EHR-based similarities, another holds genomics-based features. Design a VFL GNN that aggregates across these without revealing raw node features. Apply to disease subtyping or mortality prediction. Submittable to IJCAI, AAAI, or MICCAI.

---

### 8. Fairness in VFL — The Underdiagnosis Problem

**What's been done at DL venues:**
- **FairVFL** (NeurIPS 2022): *A Fair Vertical Federated Learning Framework with Contrastive Adversarial Learning* — the only dedicated fairness paper in VFL at a top DL venue. Uses contrastive adversarial training to enforce equitable representations across parties.
- **FlexFair** (Nature Communications, 2025): flexible fairness regularization for federated medical imaging — medical journal, not DL conference.

**Medical angle:** Demographic fairness in medical VFL is almost entirely unexplored at DL venues. The core concern: if one party holds features correlated with race/sex (e.g., zip code, insurance type) and another party holds imaging, can the jointly trained VFL model be audited or corrected for fairness in diagnosis rates? The contrastive adversarial approach from FairVFL is the starting point, but it hasn't been applied or adapted for the medical fairness context.

| Metric | Status |
|---|---|
| DL Venue Papers | Sparse (NeurIPS 2022 — only one) |
| Medical DL Papers | Near-zero at DL venues |
| Trajectory | Emerging |
| DL Novelty Type | Adversarial debiasing, contrastive fairness objectives, representation fairness in split models |
| Opportunity | 🟢 **Fertile** |

**What a DL paper here looks like:** Adapt FairVFL to the medical diagnostic setting — where one party's features may encode protected attributes (demographics in EHR) — and show that the joint model can be made fair with respect to diagnosis accuracy across groups. Validate on a real clinical dataset (MIMIC, CheXpert). Submittable to NeurIPS or FAccT.

---

### 9. VFL Benchmarking for Medicine

**What's been done at DL venues:**
- **VFLAIR** (ICLR 2024): research library for VFL supporting attack/defense evaluations — general, not medical
- **VertiBench** (ICLR 2024): addresses feature distribution diversity in VFL benchmarks — general, not medical

Both ICLR 2024 papers are significant but not medical. No medical VFL benchmark exists that: (a) uses genuinely vertically partitioned datasets (not artificially split), (b) provides realistic patient overlap fractions, (c) evaluates attack/defense under medical data, (d) standardizes evaluation across existing VFL methods.

| Metric | Status |
|---|---|
| DL Venue Papers | Moderate (ICLR 2024 x2) |
| Medical DL Papers | Zero |
| Trajectory | Emerging |
| DL Novelty Type | Benchmark design, evaluation protocol, standardized splits |
| Opportunity | 🔵 **Frontier** |

**What a DL paper here looks like:** A medical VFL benchmark paper — curating a paired dataset (e.g., TCGA imaging + genomics split across two "virtual parties," MIMIC EHR partitioned by department type) with standardized train/test splits, missing overlap simulation, and evaluation of 5+ existing VFL algorithms. Submittable to NeurIPS Datasets & Benchmarks, ICLR, or MICCAI.

---

### 10. Self-Supervised & Contrastive Representation Learning in Medical VFL

**What's been done at DL venues:**
- **Self-supervised VFL** (NeurIPS Workshop 2022): early exploration of SSL for VFL, not medical
- **FedCVT** (ACM TIST 2022): cross-view training with attention for semi-supervised VFL — closer but not at a top DL venue
- **FedHSSL** (arXiv 2023): hybrid self-supervised + split learning
- **Label-Efficient Self-Supervised FL** (IEEE Trans. Medical Imaging 2024): horizontal FL only, not VFL

**Medical angle:** Medical data is label-scarce by nature — annotating pathology slides or genomic variants requires rare expertise. Self-supervised VFL lets each party pre-train on their own unlabeled data and then align representations across parties via contrastive objectives — without labels and without sharing raw features. No paper has applied this pipeline specifically to medical VFL at a DL conference.

| Metric | Status |
|---|---|
| DL Venue Papers | Sparse (NeurIPS Workshop only; rest arXiv) |
| Medical DL Papers | Near-zero |
| Trajectory | Emerging |
| DL Novelty Type | Cross-view contrastive objectives across parties, masked autoencoders in split architecture |
| Opportunity | 🔵 **Frontier** |

**What a DL paper here looks like:** Design a VFL pre-training framework where Party A (imaging) and Party B (genomics) each do modality-specific SSL locally, then align their representations via a cross-party contrastive objective on the small overlapping patient set. Fine-tune with labels on the aligned subset only. Validate on cancer subtyping. Submittable to ICML, ICLR, or MICCAI.

---

### 11. VFL + Foundation Models / LLMs for Clinical Text

**What's been done at DL venues:**
- **Input Reconstruction Attack against Vertical Federated LLMs** (arXiv 2023): shows VFL with LLMs leaks text inputs — not medical, not at a DL conference yet
- Federated fine-tuning of LLMs in healthcare is all horizontal FL (LoRA-based, arXiv 2025)
- No DL venue paper has done VFL specifically for LLM fine-tuning in medicine

**The DL setup:** One party has radiology reports (text), another has nursing notes, another has discharge summaries — all for the same patients. VFL fine-tuning of a shared LLM backbone without sharing raw text across parties. The novelty would be in the split-layer design for transformer models (which layer to split?), the gradient alignment strategy, and demonstrating that the joint model outperforms local models on downstream clinical NLP tasks.

| Metric | Status |
|---|---|
| DL Venue Papers | Near-zero |
| Medical DL Papers | Near-zero |
| Trajectory | Exploding (LLM FL broadly) / Not started (VFL + LLM medical) |
| DL Novelty Type | Transformer split architecture, cross-party attention, LoRA in VFL setting |
| Opportunity | 🔵 **Frontier** |

**What a DL paper here looks like:** VFL fine-tuning of a clinical LLM where different parties hold different clinical note types on overlapping patients. Design the split architecture, train with cross-party gradient alignment, and demonstrate on a medical NLP benchmark (clinical note classification, ICD coding, named entity recognition). This would be extremely novel at NeurIPS, ICLR, or ACL.

---

### 12. Asynchronous & Straggler-Robust VFL for Hospital Networks

**What's been done at DL venues:**
- **FedVS** (ICML 2023): Straggler-Resilient and Privacy-Preserving VFL for Split Models — addresses the problem of slow or dropping clients in VFL using secret sharing
- **Secure Bilevel Asynchronous VFL** (AAAI 2021): asynchronous gradient updates in VFL

**Medical angle:** Hospitals have wildly different compute and network capacities — a rural clinic may take 10× longer to compute embeddings than an academic center. FedVS is the only DL venue paper addressing this directly, and it's not medical. The medical-specific variant would consider heterogeneous embedding computation times and study the effect of asynchrony on model calibration for clinical decision support (where a poorly calibrated model is dangerous).

| Metric | Status |
|---|---|
| DL Venue Papers | Sparse (ICML 2023, AAAI 2021) |
| Medical DL Papers | Zero |
| Trajectory | Emerging |
| DL Novelty Type | Asynchronous optimization in split neural networks, gradient staleness correction |
| Opportunity | 🟡 **Active** |

---

## Directions That Look Medical but Lack DL Novelty (Avoid for DL Venues)

These are real and important research directions, but their novelty belongs to non-DL venues. A DL conference reviewer would reject them for "insufficient technical novelty" even if the medical application is compelling.

| Direction | Why It Lacks DL Novelty | Where It Belongs |
|---|---|---|
| Homomorphic Encryption for medical VFL | HE is cryptography, not DL | USENIX, CCS, PETS |
| Private Set Intersection for patient matching | PSI is cryptography/systems | USENIX, CCS, SIGMOD |
| Tree-based VFL (SecureBoost) for clinical tabular data | Not neural networks | VLDB, KDD (ML track) |
| Clinical deployment / real-world VFL studies | No technical novelty | JAMIA, npj Digital Medicine |
| VFL applied to a new clinical dataset with FedAvg | No new method | Medical journals |
| Regulatory / policy analysis of medical VFL | Social science | Health Informatics journals |
| Drug discovery QSAR with VFL (standard ML) | Molecular ML, not VFL DL | J. Chem. Inf. Model., JCIM |

---

## The DL Research Map: What Actually Has Papers vs. What Doesn't

```
HIGH DL VENUE DENSITY (papers exist, methods established)
├── Reconstruction / Feature Inference Attacks (NeurIPS, USENIX, ICDE)
├── Label Inference Attacks (USENIX, IJCAI)
├── Backdoor / Adversarial Attacks (ICLR, USENIX, ECML)
├── Communication Efficiency (ICML x2, NeurIPS, ICCV)
└── General Benchmarks (ICLR x2)

MEDIUM DL VENUE DENSITY (some papers, gaps remain)
├── Semi-supervised VFL (ICCV 2023)
├── Graph VFL (IJCAI 2022, ICML 2023)
├── Fairness in VFL (NeurIPS 2022)
└── Asynchronous VFL (ICML 2023, AAAI 2021)

LOW / ZERO DL VENUE DENSITY IN MEDICAL CONTEXT  ← The Opportunity Zone
├── Reconstruction Attacks on Medical Data (DL method exists, medical context missing)
├── Label Inference in Medical VFL (same)
├── Backdoor Attacks on Medical VFL (same)
├── Multi-Modal VFL Architecture (arXiv only, medical journals)
├── Self-supervised / Contrastive VFL for Medicine (workshop only)
├── GNN-based Medical VFL (no paper)
├── VFL + LLMs for Clinical Text (no paper)
├── Medical VFL Benchmarks (no paper)
└── Fairness in Medical VFL (no DL venue paper)
```

---

## Summary Table

| Direction | DL Venue Papers | Medical DL Papers | DL Novelty Type | Opportunity |
|---|---|---|---|---|
| Gradient/Feature Reconstruction Attacks | Dense | Near-zero | Inversion networks, reconstruction objectives | 🟢 Fertile |
| Label Inference — Diagnosis Leakage | Moderate | Sparse | Adversarial inference, DP label protection | 🟢 Fertile |
| Semi-supervised VFL / Limited Patient Overlap | Sparse | Near-zero | SSL objectives, cross-view alignment | 🔵 Frontier |
| Backdoor / Adversarial Attacks in Medical VFL | Moderate | Near-zero | Trigger design, no-label-access poisoning | 🟢 Fertile |
| Communication-Efficient VFL | Dense | Sparse | Compressed embeddings, straggler robustness | 🟡 Active |
| Multi-Modal VFL Architecture for Medicine | Near-zero | Sparse (med journals) | Cross-modal attention, modality-missing fusion | 🔵 Frontier |
| Graph Neural Networks for Medical VFL | Moderate (general) | Near-zero | Vertical GNN on patient graphs | 🟢 Fertile |
| Fairness in Medical VFL | Sparse (NeurIPS 2022) | Near-zero | Adversarial debiasing in split models | 🟢 Fertile |
| Medical VFL Benchmarking | Zero | Zero | Benchmark design, evaluation protocol | 🔵 Frontier |
| Self-supervised / Contrastive VFL for Medicine | Sparse | Near-zero | Masked pretraining in split architecture | 🔵 Frontier |
| VFL + LLMs for Clinical Text | Near-zero | Near-zero | Transformer split, cross-party attention | 🔵 Frontier |
| Asynchronous VFL for Hospitals | Sparse | Zero | Async optimization in split networks | 🟡 Active |

---

## Strategic Takeaways for Research Direction Selection

**Highest-leverage plays (DL novelty + medical application + almost no competition):**

1. **VFL + LLMs for Clinical NLP** — Federated LLM fine-tuning is the hottest area in ML. VFL-specific formulation with medical text is essentially paper zero. High risk (new area), high reward (first mover at a top venue).

2. **Semi-supervised VFL with sparse patient overlap** — ICCV 2023 is the only DL venue paper. Extending it to medical datasets with realistic overlap rates (rare disease cohorts) is a clean contribution.

3. **Multi-modal VFL architecture** — The papers exist in medical journals but haven't been presented at DL venues. FedMM (arXiv) is the template. A clean MICCAI or ICLR paper is realistically achievable.

4. **Medical VFL benchmark** — VertiBench and VFLAIR are at ICLR 2024 but general. A medical-specific VFL benchmark paper (ICLR Datasets & Benchmarks or NeurIPS) would become a citation anchor for the entire field.

**Lower-risk plays (existing DL methods, clear medical instantiation):**

5. **Gradient reconstruction attacks on medical data** — CAFE's inversion network applied to medical image embeddings. Straightforward DL extension with clear HIPAA-threat framing. MICCAI or a NeurIPS security workshop.

6. **Fairness in medical VFL** — FairVFL (NeurIPS 2022) is the only precedent. Adapting contrastive adversarial fairness to medical diagnostics with protected attributes. FAccT, NeurIPS, or CHIL.

7. **GNN-based patient graph VFL** — VFGNN (IJCAI 2022) is the template. Patient similarity network with vertically partitioned node features. IJCAI, AAAI, or MICCAI.

---

## Sources

- [VFL Survey Repository (ACM Computing Surveys 2025, Wuhan University MARS Group)](https://github.com/shentt67/VFL_Survey)
- [VFLAIR: Research Library and Benchmark for VFL (ICLR 2024)](https://openreview.net/forum?id=sqRgz88TM3)
- [VertiBench: Feature Distribution Diversity in VFL Benchmarks (ICLR 2024)](https://iclr.cc/virtual/2024/poster/18132)
- [One-shot VFL with Limited Overlapping Samples (ICCV 2023)](https://openaccess.thecvf.com/content/ICCV2023/papers/Sun_Communication-Efficient_Vertical_Federated_Learning_with_Limited_Overlapping_Samples_ICCV_2023_paper.pdf)
- [CAFE: Catastrophic Data Leakage in VFL (NeurIPS 2021)](https://arxiv.org/abs/2106.05508)
- [Label Inference Attacks Against VFL (USENIX Security 2022)](https://www.usenix.org/system/files/usenixsecurity25-du.pdf)
- [FairVFL: Fair VFL with Contrastive Adversarial Learning (NeurIPS 2022)](https://arxiv.org/html/2405.17495v1)
- [VFGNN: Vertically Federated GNN for Node Classification (IJCAI 2022)](https://www.ijcai.org/proceedings/2022/272)
- [Vertical Federated GNN for Recommender System (ICML 2023)](https://arxiv.org/html/2303.05786)
- [Compressed-VFL (ICML 2022)](https://arxiv.org/abs/2206.09007)
- [FedVS: Straggler-Resilient VFL (ICML 2023)](https://arxiv.org/abs/2304.09498)
- [LESS-VFL: Communication-Efficient Feature Selection (ICML 2023)](https://arxiv.org/abs/2305.02219)
- [Constructing Adversarial Examples for VFL (ICLR 2024)](https://openreview.net/forum?id=TtdmqiKJCZ)
- [FedPass: Privacy-Preserving VFL with Adaptive Obfuscation (IJCAI 2023)](https://arxiv.org/abs/2301.12623)
- [FedMM: Multi-Modal VFL for Computational Pathology (arXiv 2024)](https://arxiv.org/html/2402.15858v1)
- [Cross-Modal VFL for MRI Reconstruction (PubMed 2024)](https://pubmed.ncbi.nlm.nih.gov/38294925/)
- [Label Leakage in VFL: A Survey (IJCAI 2024)](https://www.ijcai.org/proceedings/2024/0902.pdf)
- [VFL Survey (ACM Computing Surveys 2025)](https://arxiv.org/html/2405.17495v1)
- [FedCVT: Semi-supervised VFL with Cross-view Training (ACM TIST 2022)](https://dl.acm.org/doi/10.1145/3510031)
- [Secure Bilevel Asynchronous VFL (AAAI 2021)](https://arxiv.org/abs/2008.08970)
