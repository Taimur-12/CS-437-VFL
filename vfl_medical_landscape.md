# Vertical Federated Learning in Medicine: A Research Landscape Analysis (2019–2025)

## Preface: What VFL Means in This Context

**Vertical Federated Learning (VFL)** is the specific FL paradigm where different institutions hold *different features* about the *same patients* — not just data from different hospitals with the same variables. Classic example: Hospital A holds imaging data, Hospital B holds labs and EHR, Hospital C holds genomics, all on overlapping patient populations. They train a joint model without transferring raw data. This is fundamentally different from horizontal FL (same features, different patients) and needs to be kept distinct when reading the literature — many papers claiming VFL are actually doing horizontal FL with a vertical spin, so actual dedicated VFL papers are a smaller, more specific set.

**A critical stat from a 2024 systematic review (Cell Reports Medicine, n=577 papers):** Only **23 studies** out of 577 examined used vertical data partitioning. The other 554 used horizontal partitioning. This means VFL in medicine is genuinely underrepresented, even as overall federated learning for healthcare explodes.

---

## Classification System

Each direction is rated across four dimensions:

| Metric | What It Measures |
|---|---|
| **Paper Volume** | Sparse / Moderate / Dense / Saturated |
| **Trajectory** | Emerging / Rising / Plateauing / Declining |
| **Technical Maturity** | Nascent / Developing / Established / Overmature |
| **Opportunity Rating** | 🔴 Crowded · 🟡 Active · 🟢 Fertile · 🔵 Frontier |

**Opportunity Ratings Defined:**
- 🔴 **Crowded** — Diminishing marginal novelty. Incremental papers likely, but hard to distinguish
- 🟡 **Active** — Solid base of work, but clear open problems with room for real contributions
- 🟢 **Fertile** — Surprisingly little work given the importance. High leverage for new papers
- 🔵 **Frontier** — Almost no work. Highest risk (no community yet), highest potential payoff

---

## Direction-by-Direction Analysis

---

### 1. VFL for EHR / Structured Clinical Data

**What's being done:** Multiple institutions hold *complementary* structured features about shared patients — one might have lab values, another medications, another demographic + billing codes. VFL allows joint model training for mortality, readmission, sepsis, length-of-stay prediction. Key papers: FedRL (2025, PubMed 39954511) proposes representation learning with data subset approach; FedWeight (2025, npj Digital Medicine) handles covariate shift in cross-hospital EHR. Work on MIMIC-III/IV and eICU for sepsis and mortality is active. Vaid et al. were among the first with real hospital deployments (5 hospitals, Mount Sinai Health System).

**What it's NOT doing well:** Most "EHR VFL" papers still simulate vertical partitioning artificially (splitting one dataset into halves). Very few papers use genuinely different institutions with naturally different feature sets. The missing data problem under vertical partitioning (a party has features for only a subset of the shared patients) is rarely tackled head-on.

| Metric | Status |
|---|---|
| Volume | Moderate (genuine VFL EHR papers: ~15–20; EHR FL broadly: Dense) |
| Trajectory | Rising |
| Technical Maturity | Developing |
| Opportunity | 🟢 **Fertile** |

**Open gaps:** The gap between "simulated VFL on split MIMIC" and "real cross-institutional VFL with naturally complementary data" is enormous. Work that bridges this gap with real deployment scenarios would be high impact.

---

### 2. VFL for Medical Imaging (Cross-Modal / Cross-Institution)

**What's being done:** The dominant sub-problem is where different institutions hold *different imaging modalities* for the same patients — e.g., one center has CT, another has PET, another has MRI. The key paper is "Cross-Modal Vertical Federated Learning for MRI Reconstruction" (PubMed 38294925, 2024). The Adaptive Multimodal Fusion in VFL for Glaucoma Screening (MDPI Brain Sciences, 2025) is also notable — using fundus images, clinical text, and biomedical signals split across parties with a Quality Aware VFL (QAVFL) framework. This is different from the bulk of FL imaging work, which is horizontal (same modality, different institutions).

**What's still horizontal FL in disguise:** The majority of medical imaging FL papers (MRI tumor segmentation, COVID chest X-ray, etc.) are horizontal. They're technically polished and clinically validated, but not VFL.

| Metric | Status |
|---|---|
| Volume | Sparse (genuinely cross-modal VFL imaging: ~5–10 papers) |
| Trajectory | Emerging |
| Technical Maturity | Nascent |
| Opportunity | 🔵 **Frontier** |

**Open gaps:** Different hospitals often specialize in modalities. The methods barely exist, and the clinical need is clear. This is one of the highest-potential directions.

---

### 3. VFL for Multi-Omics Integration

**What's being done:** This is where VFL has its clearest natural fit: genomics lab holds genomic data, clinical center holds phenotypic/EHR data, proteomics lab holds protein expression — all on the same cohort. Key papers:
- **AFEI** (2023): Adaptive optimized VFL for heterogeneous multi-omics integration
- **Domain-aware priors for VFL in genomics** (arXiv 2601.00050, 2025)
- **Federated Multi-Omics for Parkinson's Disease** (Cell Patterns, 2024): FL model performance tracks centralized ML models for multi-omics PD prediction
- **PLOS Computational Biology (2024)**: Federated privacy-protected meta- and mega-omics data analysis in multi-center studies

Cancer prognosis via multi-omics VFL is a specific sub-thread (ScienceDirect 2025: Interpretable VFL with privacy-preserving multi-source data integration for prognostic prediction).

**What's missing:** GWAS-scale VFL, VFL for longitudinal omics over time, VFL for single-cell RNA-seq data across labs.

| Metric | Status |
|---|---|
| Volume | Sparse-to-Moderate (~10–20 papers) |
| Trajectory | Rising rapidly |
| Technical Maturity | Developing |
| Opportunity | 🟢 **Fertile** |

**Open gaps:** Genomics/omics is a massive field where VFL is the *natural* paradigm (data is genuinely vertically partitioned by design), yet only a handful of papers exist. Major opportunity, especially for cancer genomics.

---

### 4. VFL for Cancer Prognosis & Survival Prediction

**What's being done:** Multi-source data integration for cancer outcome prediction. Papers include:
- **Robustly Federated Model for Gastric Cancer Recurrence** (Nature Communications, 2024): 4 data centers, AUC 0.710–0.869
- **FL Survival Model for NSCLC** (ScienceDirect, 2024): Radiotherapy decision support using real-world data
- **Decentralized FL-based Cancer Survival Prediction** (Heliyon/PMC11153246, 2024)
- **Interpretable VFL for Prognostic Prediction** (ScienceDirect, 2025): Multi-source omics integration with interpretability focus

The dominant approach: use one party with imaging, another with clinical/EHR, another with omics, then VFL for fusion. Cancer is a natural test-bed because oncology routinely generates multi-modal data across different institutional labs.

| Metric | Status |
|---|---|
| Volume | Moderate (~15–25 papers) |
| Trajectory | Rising |
| Technical Maturity | Developing |
| Opportunity | 🟡 **Active** |

**Open gaps:** The space of cancer types, modality combinations, and clinical outcomes is vast. Specific cancers with multi-omics datasets (TCGA-compatible scenarios) are underexplored.

---

### 5. VFL for Drug Discovery / Pharmaceutical QSAR

**What's being done:** This is the **most mature and industrially validated** direction in all of medical VFL. Pharma companies hold complementary activity data on the same chemical compounds — a textbook VFL scenario.
- **MELLODDY** (J. Chem. Inf. Model., 2023): The gold standard. 10 pharmaceutical companies, 2.6 billion experimental data points, 21M+ molecules, 40K+ assays. Demonstrated consistent improvement in QSAR classification and regression over single-partner baselines. Published by consortium including Bayer, Janssen, Eli Lilly, Novartis, Pfizer, et al.
- **FLuID** (Nature Machine Intelligence, 2025): Federated learning via knowledge distillation, validated with 8 pharma companies
- **Federated Graph Learning for Drug Discovery** (Nature Machine Intelligence, 2026): FedLG using Lanczos algorithm
- **Federated Toxicology — Effiris Hackathon** (Chem. Res. Toxicol., 2023): Industrial perspective on federated computational toxicology
- **Federated GNNs for Molecular Property Prediction** (Cell Patterns, 2022): Heterogeneous federated molecular property prediction

| Metric | Status |
|---|---|
| Volume | Dense (~30+ papers, multiple real deployments) |
| Trajectory | Plateauing (core QSAR problem partially solved) |
| Technical Maturity | Established |
| Opportunity | 🟡 **Active** |

**Open gaps:** (a) VFL for generative drug design (not just property prediction), (b) VFL for clinical trial data across pharma partners, (c) VFL for drug-drug interactions with cross-institutional patient data, (d) VFL for rare disease compound screening.

---

### 6. Privacy & Security in Medical VFL (Attacks & Defenses)

**What's being done:** This is the most technically active area in VFL research broadly. Key threats:
- **Label Inference Attacks**: Passive clients inferring labels from gradients
- **Feature Inference / Gradient Inversion**: Reconstructing raw features from intermediate embeddings
- **Backdoor Attacks**: Injecting poisoned samples (Universal Adversarial Backdoor — UAB, Computers & Security 2023; BadVFL with 93%+ attack success rate at 1% poisoning)
- **Model Extraction Attacks**: Stealing the joint model
- **Membership Inference**: Determining if a patient record was in training

**Defense work:** SVFL (feature disentanglement, ScienceDirect 2023), Label Consistency Purification (IJCAI 2025), IHVFL (intention-hiding VFL for medical data, Cybersecurity 2023), DP + HE combinations (DPSHE, ScienceDirect 2025), FeSEC.

**Medical-specific angle:** Healthcare labels (e.g., diagnosis codes) being inferred by non-label-holding parties is a HIPAA-critical concern. Papers explicitly scoped to medical are fewer — most attack/defense papers are general VFL, then claim medical applicability.

| Metric | Status |
|---|---|
| Volume | Dense in general, Moderate in explicitly medical scope |
| Trajectory | Rising fast |
| Technical Maturity | Developing to Established |
| Opportunity | 🟡 **Active** |

**Open gaps:** (a) Privacy guarantees under clinical-grade threat models (HIPAA/GDPR), (b) attacks exploiting medical domain knowledge (e.g., disease ontologies), (c) security analysis of real VFL deployments in hospital networks rather than simulations.

---

### 7. Cryptographic Privacy Mechanisms for Medical VFL (DP, HE, MPC)

**What's being done:**
- **Differential Privacy (DP)**: Adding calibrated noise to embeddings/gradients. Used in bipolar disorder prediction (5 Korean hospitals), COVID detection (FeSEC), breast cancer FL
- **Homomorphic Encryption (HE)**: Computation on encrypted data. DPSHE (ScienceDirect 2025) combines DP + threshold HE for medical images. PPFLHE for chronic kidney disease
- **Secure Multi-Party Computation (MPC)**: Less common in medical due to computational cost
- **Private Set Intersection (PSI)**: For patient matching / sample alignment (IHVFL 2023)

The computational overhead of HE at clinical scale is prohibitive. Most papers demonstrate correctness on small medical datasets. The performance vs. privacy tradeoff in real hospital infrastructure is rarely benchmarked.

| Metric | Status |
|---|---|
| Volume | Moderate (explicitly medical: ~15–25 papers) |
| Trajectory | Rising |
| Technical Maturity | Developing |
| Opportunity | 🟢 **Fertile** |

**Open gaps:** Efficient HE for medical VFL, DP under small-cohort medical settings (low privacy budget is dangerous for rare conditions), and PSI designed for clinical identifiers.

---

### 8. Sample Alignment / Private Set Intersection in Medical VFL

**What's being done:** Before training, VFL parties must find which patients they share in common without revealing their full patient lists — the sample alignment problem, solved via PSI protocols. Papers: IHVFL (Cybersecurity 2023) addresses intention-hiding VFL for medical data with PSI. TreeCSS (DASFAA 2024) uses tree-based PSI. arXiv 2106.05508 discusses VFL without revealing intersection membership.

**Medical-specific concern:** Patient identifiers (name, DOB, MRN) are highly sensitive. Fuzzy matching (for misspellings, anonymized IDs) makes PSI harder. Most PSI literature assumes exact matching.

| Metric | Status |
|---|---|
| Volume | Sparse (medical-specific PSI papers: < 10) |
| Trajectory | Emerging |
| Technical Maturity | Nascent |
| Opportunity | 🔵 **Frontier** |

**Open gaps:** Fuzzy entity matching for patient record linkage in VFL is a largely unsolved problem. Clinical record systems have inconsistent identifiers, and privacy-preserving fuzzy matching at VFL scale is essentially open research.

---

### 9. Communication Efficiency in Medical VFL

**What's being done:** VFL requires multiple rounds of embedding exchange during both forward and backward passes. Papers:
- "Communication-efficient Vertical Federated Learning via Compressed..." (arXiv 2406.14420, 2024)
- "Accelerated Methods with Compression for Horizontal and Vertical FL" (J. Optim. Theory Appl., 2025)
- FedMRG for communication-efficient medical report generation

**The medical angle:** Hospital network bandwidth varies enormously (rural hospitals vs. academic medical centers). Asynchronous VFL (handling stragglers in federated training across slow hospital connections) is underexplored.

| Metric | Status |
|---|---|
| Volume | Sparse in medical specifically (~5–10 papers) |
| Trajectory | Emerging |
| Technical Maturity | Nascent |
| Opportunity | 🟢 **Fertile** |

**Open gaps:** In medical contexts, the constraints are different from general VFL (HIPAA-compliant channels, lower bandwidth, higher latency, burst traffic). Medical-specific communication-efficient VFL is barely started.

---

### 10. Fairness & Bias in Medical VFL

**What's being done:**
- **FairVFL** (NeurIPS 2022): Fair VFL framework with contrastive adversarial learning — the canonical paper
- **FlexFair** (Nature Communications, 2025): Flexible fairness metrics for federated medical imaging, evaluated on polyp segmentation, fundus segmentation, cervical cancer, skin disease
- **Unified Fair Federated Learning for Digital Healthcare** (Patterns/PMC10801255, 2024)
- **Analyzing Personalization's Impact on Fairness in Healthcare FL** (PMC11052754, 2024)

A 2024 systematic review found **only 3 studies** in all of healthcare FL addressed fairness. FairVFL is the only explicitly vertical FL fairness paper found.

| Metric | Status |
|---|---|
| Volume | Sparse (< 5 dedicated VFL fairness papers in medicine) |
| Trajectory | Emerging |
| Technical Maturity | Nascent |
| Opportunity | 🔵 **Frontier** |

**Open gaps:** Questions like "does VFL amplify demographic disparities in cancer screening?" are not answered. Fairness in VFL reward distribution among participating hospitals (incentive mechanisms) is also unaddressed.

---

### 11. Semi-Supervised & Self-Supervised Learning in Medical VFL

**What's being done:** Labels are expensive in medical settings. Semi-supervised federated learning (FSSL) is growing in horizontal FL — prototype pseudo-labeling, contrastive learning, consistency regularization. Key horizontal papers: FedProto (2023), FedVCK (arXiv 2412.18557, 2024), label-efficient self-supervised FL (PMC10880587, 2024). In vertical FL specifically, FedRL (PubMed 39954511, 2025) addresses limited labeled samples via representation learning.

**The VFL-specific twist:** In VFL, labels are usually held by only one party (the "active client"). Semi-supervised VFL means some of those labeled patients may not have corresponding records at all parties — a harder problem than horizontal semi-supervised FL, and almost no papers address it specifically.

| Metric | Status |
|---|---|
| Volume | Moderate in horizontal medical FL; Sparse in vertical medical FL |
| Trajectory | Rising (horizontal) / Emerging (vertical) |
| Technical Maturity | Nascent for VFL-specific work |
| Opportunity | 🟢 **Fertile** |

**Open gaps:** Extending semi-supervised and self-supervised methods to the VFL medical setting — where the label-holder is a distinct party — is a largely open problem.

---

### 12. VFL + Foundation Models / LLMs for Medicine

**What's being done:**
- **Federated Fine-tuning of LLMs in Healthcare** (arXiv 2510.00543, 2025): LoRA-based federated fine-tuning under extreme non-IID conditions
- **FedMentalCare** (arXiv 2503.05786, 2025): Privacy-preserving LLM fine-tuning for mental health status analysis
- **Federated LLMs for Adverse Drug Reactions** (PMC12516295, 2025): Scoping review of federated LLM applications in ADR detection
- **FedMRG** (arXiv 2506.17562): Communication-efficient heterogeneous FL for LLM-driven medical report generation
- **Federated Foundation Models for Biomedical Healthcare** (BioData Mining, 2025): Open challenges paper

**Specifically VFL + LLMs:** Almost none. The LLM + FL literature is overwhelmingly horizontal. Clinical notes and medical reports are a classic VFL case (one institution has radiology notes, another has nursing notes, another has operative notes), but no paper has formally tackled VFL specifically for LLM fine-tuning.

| Metric | Status |
|---|---|
| Volume | Sparse in VFL specifically (1–2 adjacent papers); Dense in horizontal federated LLMs |
| Trajectory | Exploding (horizontal) / Emerging (vertical) |
| Technical Maturity | Nascent |
| Opportunity | 🔵 **Frontier** |

**Open gaps:** This is the single highest-potential research direction in the entire landscape. VFL + federated LLM fine-tuning for multi-institutional clinical NLP is essentially open. First-mover advantage is real here.

---

### 13. VFL for Neurology (Alzheimer's, Parkinson's, EEG)

**What's being done:**
- **Vertical Federated Alzheimer's Detection on Multimodal Data** (arXiv 2312.10237, 2023): MRI (ResNet-18) at one party + demographic features at another, OASIS-2 dataset, 82.9% accuracy — two federates with genuinely different modalities
- **Integrative Federated Framework for Parkinson's Biomarker Fusion** (MDPI Computers, 2025): FL for freezing of gait detection, EEG-fMRI diagnosis, speech classification
- **Federated XAI Models for Parkinson's** (Cognitive Computation, Springer 2024): Federated SHAP + fuzzy rule-based systems
- **Federated Fine-tuning of SAM-Med3D for Dementia Classification** (Springer, 2025): Brain MRI

Most neurology FL work is horizontal. The Alzheimer's multimodal paper is the clearest VFL-specific contribution.

| Metric | Status |
|---|---|
| Volume | Sparse (VFL-specific: ~3–5 papers) |
| Trajectory | Emerging |
| Technical Maturity | Nascent |
| Opportunity | 🟢 **Fertile** |

**Open gaps:** Neurological diseases naturally involve multi-institutional, multi-modal data (clinical evaluation, neuroimaging, biomarkers, speech/gait) held by different specialists. VFL is the right architecture but barely applied here.

---

### 14. VFL for Ophthalmology

**What's being done:**
- **QAVFL for Glaucoma Screening** (MDPI Brain Sciences, 2025): Quality-aware VFL with clinical text + retinal fundus + biomedical signals across parties — the most developed VFL ophthalmology paper
- Federated learning for diabetic retinopathy (horizontal, OPHDIAT dataset, ~700K images, AUC 0.9317)
- Distributed training of foundation models for ophthalmic diagnosis (Nature Communications Engineering, 2025)
- Multi-disease diagnostics from OCTA (ScienceDirect, 2025)

Most ophthalmology FL is horizontal. The QAVFL paper is the standout genuinely vertical work.

| Metric | Status |
|---|---|
| Volume | Sparse in VFL specifically (1–2 papers) |
| Trajectory | Emerging |
| Technical Maturity | Nascent |
| Opportunity | 🟢 **Fertile** |

**Open gaps:** Ophthalmology clinics, optometrists, primary care, and specialist centers hold fundamentally different data on shared patients. VFL architecture fits naturally but almost no work exists.

---

### 15. VFL for Computational Pathology

**What's being done:** Computational pathology FL is growing rapidly, peaking in 2022–2023 with 6 papers per year, continuing into 2024–2025. But **almost all work is horizontal FL** — same modality (WSI), different institutions. Key papers: FL for gigapixel whole slide images (Medical Image Analysis, 2022), FedWSIDD (Springer, 2025), FedMM (arXiv 2402.15858, 2024) for multi-modal computational pathology.

**The VFL gap:** In pathology, genomics + morphology integration across different labs is a classic vertical partition — hospital pathology lab has WSI; genomics core has sequencing data on the same tumor blocks. This is an obvious VFL use case with near-zero papers.

| Metric | Status |
|---|---|
| Volume | Dense in horizontal FL pathology; Sparse (near-zero) in VFL pathology |
| Trajectory | Plateauing (horizontal) / Not yet started (vertical) |
| Technical Maturity | Nascent for VFL-specific work |
| Opportunity | 🔵 **Frontier** |

**Open gaps:** Morphology-genomics integration across institutions via VFL in computational pathology is a completely open research direction with clear clinical motivation.

---

### 16. Split Learning for Medicine

**What's being done:** Split learning (SL) — splitting the model itself at a "cut layer" and having different parties compute different parts — is technically distinct from VFL but closely related. Key papers:
- **Split Learning for Health** (MIT Media Lab, 2019): The original paper, still highly cited
- **Multi-site Split Learning** (Scientific Reports, 2022): COVID-19, X-ray, cholesterol datasets; multi-hospital feasibility study
- **SplitFed** (various): Combines split learning + federated aggregation
- **Performance and Information Leakage in Splitfed and Multi-Head Split Learning** (PubMed 35893586, 2022)

The field has found that split learning leaks more information than previously thought (intermediate activations can be inverted). The community is now combining SL with DP and encryption.

| Metric | Status |
|---|---|
| Volume | Moderate (~20–30 papers) |
| Trajectory | Steady, with security concerns slowing adoption |
| Technical Maturity | Developing |
| Opportunity | 🟡 **Active** |

**Open gaps:** The combination of split learning + VFL for resource-constrained clinical devices (wearables, edge hospital equipment) is underexplored. Also: secure split learning for on-device medical AI.

---

### 17. Explainability / Interpretability in Medical VFL

**What's being done:**
- **Interpretable VFL for Cancer Prognosis** (ScienceDirect 2025): Multi-source data integration with interpretability module
- **Federated XAI for Parkinson's** (Cognitive Computation 2024): Federated SHAP + fuzzy rule-based systems
- **Explainable, Domain-Adaptive, Federated AI in Medicine** (IEEE JAS, 2023): Trio survey covering all three simultaneously
- **FairVFL** (NeurIPS 2022): Fairness + interpretability in VFL

**The core tension:** In VFL, the model is split across parties. Explaining what each party's features contributed to the final prediction requires cross-party attribution — and that attribution signal itself can leak information about the features the other party holds. This creates a privacy-explainability tension unique to VFL.

| Metric | Status |
|---|---|
| Volume | Sparse (VFL-specific XAI in medicine: ~3–5 papers) |
| Trajectory | Emerging |
| Technical Maturity | Nascent |
| Opportunity | 🟢 **Fertile** |

**Open gaps:** Privacy-preserving cross-party attribution (how much did Hospital B's genomics data contribute to this patient's cancer risk score, without revealing B's features to A?) is essentially unsolved.

---

### 18. VFL for Mental Health / Psychiatry

**What's being done:** Mostly horizontal FL. VFL-specific work is minimal. The pattern — one party has social media text/survey data, another has clinical notes, another has EHR — is a natural vertical partition for mental health, but no paper formally treats it as VFL.
- Federated FL for bipolar disorder (5 Korean hospitals, 2023)
- FedMentalCare (arXiv 2503.05786, 2025): LLM-based FL for mental health
- FL for violence prediction in psychiatric settings (Expert Sys. App., 2022)
- Depression detection via multilingual FL (PMC11284503, 2024)

| Metric | Status |
|---|---|
| Volume | Sparse in VFL specifically |
| Trajectory | Emerging (general mental health FL is growing) |
| Technical Maturity | Nascent |
| Opportunity | 🟢 **Fertile** |

**Open gaps:** High clinical impact, strong privacy motivation (mental health data is among the most sensitive), clear VFL structure (multiple data owners per patient: social media platform + clinic + pharmacy + wearable), almost no VFL-specific work.

---

### 19. VFL for IoT / Wearables + Clinical Data Fusion

**What's being done:** Predominantly horizontal FL. Federated learning on wearables (seizure detection, fall detection, activity recognition) uses distributed sensor data. The VFL angle would be: wearable manufacturer holds sensor data, clinic holds EHR, insurance holds claims — all on the same patient.
- FL + Blockchain for wearable IoT (arXiv 2301.04511, 2023)
- Privacy-preserving edge FL for mobile health (ScienceDirect 2024)
- FEEL framework for disease detection from real-time environment data

The IoT + FL space is large. The VFL angle is implied but not formally developed for this application.

| Metric | Status |
|---|---|
| Volume | Dense in horizontal IoT FL; Sparse in genuine VFL for IoT + clinical fusion |
| Trajectory | Steady |
| Technical Maturity | Developing (general) / Nascent (VFL-specific) |
| Opportunity | 🟢 **Fertile** (VFL-specific angle) |

---

### 20. VFL for Real-World Clinical Deployment

**What's being done:** The gap is stark. A 2024 systematic review found **only 5.2%** of FL healthcare papers involved actual real-world deployment, and of those, only 10 studies involved truly distributed clinical settings. The VFL fraction of real deployments is even smaller — possibly 1–2 studies globally. Exceptions exist in pharma (MELLODDY) but not hospital medicine.

**Barriers:** Requires formal data sharing agreements even though raw data isn't transferred. HIPAA compliance of model outputs. Audit trails. IT infrastructure disparity. Regulatory approval of federated models.

| Metric | Status |
|---|---|
| Volume | Near-zero (< 5 genuine VFL clinical deployments documented) |
| Trajectory | Slowly Emerging |
| Technical Maturity | Nascent |
| Opportunity | 🔵 **Frontier** |

**Open gaps:** "Real-world VFL deployment" papers documenting what actually happens when you try to deploy VFL in hospitals would be extremely high impact. The "translation gap" paper space — failure modes, implementation challenges, solutions in actual deployment — is wide open.

---

### 21. Benchmarking, Datasets & Standardization for Medical VFL

**What's being done:** General FL frameworks — FATE (FederatedAI), Flower, PySyft, NVFlare, FedML — all support VFL to varying degrees. FATE has the most mature VFL module. Flower is highest rated overall (84.75% score in comparative studies). Medical VFL benchmarks are essentially nonexistent. MIMIC-III/IV is used but always artificially partitioned. UK Biobank is used for genomics FL.

**The critical gap:** There is no standard benchmark for VFL in medicine. No established dataset pair where Institution A holds one set of features and Institution B holds another, with ground truth labels at one party, with realistic missingness and heterogeneity.

| Metric | Status |
|---|---|
| Volume | Near-zero (< 5 papers focus on VFL benchmarking in medicine) |
| Trajectory | Emerging |
| Technical Maturity | Nascent |
| Opportunity | 🔵 **Frontier** |

**Open gaps:** Publishing a well-designed benchmark for medical VFL (with realistic feature partitions, missing patient overlap, and evaluation protocol) would be a foundational contribution the entire community would build on.

---

## Summary Table

| Direction | Volume | Trajectory | Maturity | Opportunity |
|---|---|---|---|---|
| EHR / Structured Clinical Data VFL | Moderate | Rising | Developing | 🟢 Fertile |
| Cross-Modal Medical Imaging VFL | Sparse | Emerging | Nascent | 🔵 Frontier |
| Multi-Omics VFL | Sparse-Moderate | Rising fast | Developing | 🟢 Fertile |
| Cancer Prognosis VFL | Moderate | Rising | Developing | 🟡 Active |
| Drug Discovery / Pharma QSAR VFL | Dense | Plateauing | Established | 🟡 Active |
| Privacy Attacks & Defenses in Medical VFL | Dense (general) / Moderate (medical) | Rising | Developing | 🟡 Active |
| Cryptographic Techniques (DP, HE, MPC) | Moderate | Rising | Developing | 🟢 Fertile |
| Sample Alignment / Patient Matching PSI | Sparse | Emerging | Nascent | 🔵 Frontier |
| Communication Efficiency in Medical VFL | Sparse | Emerging | Nascent | 🟢 Fertile |
| Fairness & Bias in Medical VFL | Sparse | Emerging | Nascent | 🔵 Frontier |
| Semi-Supervised / Self-Supervised VFL | Moderate (horiz.) / Sparse (VFL) | Emerging | Nascent | 🟢 Fertile |
| VFL + Foundation Models / LLMs | Sparse | Exploding | Nascent | 🔵 Frontier |
| Neurology VFL (Alzheimer's, Parkinson's) | Sparse | Emerging | Nascent | 🟢 Fertile |
| Ophthalmology VFL | Sparse | Emerging | Nascent | 🟢 Fertile |
| Computational Pathology VFL | Near-zero | Not started | Nascent | 🔵 Frontier |
| Split Learning for Medicine | Moderate | Steady | Developing | 🟡 Active |
| Explainability / XAI in Medical VFL | Sparse | Emerging | Nascent | 🟢 Fertile |
| Mental Health / Psychiatry VFL | Sparse | Emerging | Nascent | 🟢 Fertile |
| IoT / Wearables + Clinical VFL | Dense (horiz.) / Sparse (VFL) | Steady | Developing | 🟢 Fertile |
| Real-World Clinical Deployment | Near-zero | Slowly emerging | Nascent | 🔵 Frontier |
| Benchmarking & Standardization | Near-zero | Emerging | Nascent | 🔵 Frontier |

---

## Cross-Cutting Observations

**1. The Horizontal FL Confusion Tax.**
Across all surveys, ~95% of FL-in-medicine work is horizontal. Many papers nominally claiming "VFL" or "multi-institutional FL" are still simulating vertical partitioning by artificially splitting a single horizontally collected dataset. This makes the genuine VFL medical literature much smaller and harder to find than it appears.

**2. The Deployment Gap is Real and Wide.**
Only 5.2% of all medical FL papers are real deployments. For VFL, it's effectively zero outside pharma (MELLODDY). Every "simulation to clinical pipeline" paper has near-certain publication value.

**3. Pharma is Way Ahead of Clinical Medicine.**
MELLODDY (10 pharma companies, 2.6B data points) is more sophisticated than anything in hospital VFL. The reason: pharma has a shared molecule vocabulary, commercial incentives, and established legal frameworks for IP sharing. Hospitals lack all three.

**4. Privacy and Utility Are Still in Tension.**
The combination of (a) adding enough DP noise to be safe, (b) with enough patients in the overlapping set to train a useful model, (c) in rare disease settings where both are small — is not solved.

**5. The Freshest Frontier: VFL × Foundation Models.**
Federated LLM fine-tuning is the hottest area in all of ML. VFL-specific federated LLM fine-tuning for multi-institutional clinical NLP — where different institutions hold different note types on overlapping patients — is essentially zero papers. This is almost certainly where the next wave lands.

---

## Sources

- [Federated machine learning in healthcare: Systematic review (Cell Reports Medicine, 2024)](https://pmc.ncbi.nlm.nih.gov/articles/PMC10897620/)
- [VFL for Effectiveness, Security, Applicability: A Survey (arXiv 2405.17495)](https://arxiv.org/html/2405.17495v1)
- [Vertical Federated Alzheimer's Detection on Multimodal Data (arXiv 2312.10237)](https://arxiv.org/html/2312.10237v2)
- [MELLODDY Cross-pharma FL (J. Chem. Inf. Model., 2023)](https://pmc.ncbi.nlm.nih.gov/articles/PMC11005050/)
- [AFEI for Multi-Omics VFL (ResearchGate, 2023)](https://www.researchgate.net/publication/372678994_AFEI_adaptive_optimized_vertical_federated_learning_for_heterogeneous_multi-omics_data_integration)
- [VFL for Healthcare — FedRL (PubMed 39954511, 2025)](https://pubmed.ncbi.nlm.nih.gov/39954511/)
- [Cross-Modal VFL for MRI Reconstruction (PubMed 38294925, 2024)](https://pubmed.ncbi.nlm.nih.gov/38294925/)
- [QAVFL for Glaucoma Screening (MDPI Brain Sciences, 2025)](https://www.mdpi.com/2076-3425/15/9/990)
- [Privacy Survey in VFL (arXiv 2402.03688)](https://arxiv.org/html/2402.03688)
- [IHVFL for Medical Data (Cybersecurity, Springer 2023)](https://link.springer.com/article/10.1186/s42400-023-00166-9)
- [FairVFL — Fair VFL with Contrastive Adversarial Learning (NeurIPS 2022)](https://arxiv.org/html/2405.13832v1)
- [Federated Multi-Omics for Parkinson's Disease (Cell Patterns, 2024)](https://www.sciencedirect.com/science/article/pii/S2666389924000448)
- [Review on FL for Computational Pathology (PMC11584763)](https://pmc.ncbi.nlm.nih.gov/articles/PMC11584763/)
- [Universal Backdoor Attacks in VFL (Computers & Security, 2023)](https://www.sciencedirect.com/science/article/abs/pii/S0167404823005114)
- [Robustly FL Model for Gastric Cancer Recurrence (Nature Communications, 2024)](https://www.nature.com/articles/s41467-024-44946-4)
- [Open Challenges: Federated Foundation Models for Biomedical Healthcare (BioData Mining, 2025)](https://pmc.ncbi.nlm.nih.gov/articles/PMC11700470/)
- [Split Learning for Health — MIT Media Lab (2019)](https://www.academia.edu/38209207/Split_learning_for_health_Distributed_deep_learning_without_sharing_raw_patient_data)
- [FL Survival Model for NSCLC (ScienceDirect, 2024)](https://www.sciencedirect.com/science/article/pii/S0936655524001055)
- [FedWeight for EHR Cross-Site (npj Digital Medicine, 2025)](https://www.nature.com/articles/s41746-025-01661-8)
- [Domain-Aware Priors for VFL in Genomics (arXiv 2601.00050, 2025)](https://arxiv.org/pdf/2601.00050)
- [Federated Semi-Supervised Medical Image Segmentation (PubMed 37703140, 2023)](https://pubmed.ncbi.nlm.nih.gov/37703140/)
- [Federated LLMs for Adverse Drug Reactions — Scoping Review (PMC12516295, 2025)](https://pmc.ncbi.nlm.nih.gov/articles/PMC12516295/)
- [Decentralized FL-based Cancer Survival Prediction (PMC11153246, 2024)](https://pmc.ncbi.nlm.nih.gov/articles/PMC11153246/)
- [FlexFair — Flexible Fairness in Federated Medical Imaging (Nature Communications, 2025)](https://www.nature.com/articles/s41467-025-58549-0)
- [Federated Learning for Diabetic Retinopathy — Multi-center (PubMed 38082571, 2023)](https://pubmed.ncbi.nlm.nih.gov/38082571/)
