# Paper Writing Patterns — Empirical Guide

Research-based reference derived from analyzing 7 closely related papers. Use this when writing or critiquing any section of our VFL privacy paper. Citations to source papers appear in brackets.

**Papers analyzed:**
- [UIFV] arXiv 2406.12588 — data reconstruction attack in VFL
- [HybridVFL] arXiv 2512.10701 — multimodal VFL for skin lesion (HAM10000)
- [FedVQCS] arXiv 2204.07692 — vector quantization in FL (communication)
- [PRIVEE] arXiv 2512.12840 — privacy-preserving VFL against feature inference
- [OPUS-VFL] arXiv 2504.15995 — optimal privacy-utility tradeoffs in VFL
- [FedEM] arXiv 2503.06021 — privacy-preserving FL (SSIM/LPIPS metrics)
- [FedEvPrompt] arXiv 2411.10071 — evidential FL for skin lesion (ISIC-2019)

---

## 1. Abstract

### Structure formula (across all 7 papers)
All papers use a single paragraph of **4–8 sentences** following this invariant order:
1. Problem statement — one sentence. Never starts with "In this paper."
2. Proposed solution + what makes it different — one sentence naming the method.
3. Technical approach or key mechanism — one sentence.
4. Key quantified result — always includes at least one specific number.
5. Implication or broader claim — one sentence.

### Length
- UIFV: 3 sentences (very short). HybridVFL: 4 sentences. FedEvPrompt: 7 sentences. PRIVEE: 5 sentences. OPUS-VFL: 8 sentences.
- Target for our paper: **5–6 sentences, ≤150 words.**

### Lead sentence patterns observed
- UIFV: Leads with vulnerability ("privacy risks, where adversaries might reconstruct...")
- HybridVFL: Leads with FL limitations in edge AI.
- OPUS-VFL: Leads with "critical limitations."
- All attack/privacy papers lead with the **problem**, not the result.
- Result papers (FedVQCS) lead with the contribution framed as improvement.

### Numbers in abstract
- OPUS-VFL includes 3 specific numbers: "up to 20%," "over 30%," "up to 25%."
- PRIVEE includes "threefold improvement."
- HybridVFL and FedEvPrompt defer numbers to the main text.
- **Rule**: For a privacy paper at a competitive venue, include at least 1–2 specific numbers in the abstract. Our headline: "+3.4pp WACC, SSIM 0.503 vs 0.622, 853× fewer bits."

### What NOT to write in the abstract
- Do not use passive constructions to state your own contributions ("a method is proposed...").
- Do not explain your method in detail — just name it.
- Do not list all 11 methods — state the headline result only.
- Do not claim formal DP guarantees unless proven.

### Template for our abstract (fitted to observed patterns)
> S1: Problem (VFL embeddings leak private images through passive inversion).
> S2: We propose [method name], a learned product quantization bottleneck applied at the passive party's communication boundary.
> S3: VQ discretization simultaneously regularizes the classifier and reduces representation invertibility.
> S4: On ISIC-2019, our best operating point (H_vq_K64) improves balanced accuracy by 3.4 percentage points over continuous VFL (0.786 vs. 0.752) while reducing reconstruction SSIM from 0.622 to 0.503 at 853× lower communication cost.
> S5: These results show that learned discrete transmission is Pareto-dominant over both continuous and sign-quantized baselines on utility, privacy, and efficiency simultaneously.

---

## 2. Introduction

### Paragraph structure (observed across papers)
All 7 papers follow the same macro-structure. Paragraph count ranged from **3–9 paragraphs**.

Canonical order:
1. **Clinical / application motivation** (1–2 paragraphs): frames why the problem matters in the real world. Medical papers spend ~60% of para 1 on clinical context before pivoting to technical content [FedEvPrompt, HybridVFL].
2. **Technical landscape** (1–2 paragraphs): introduces FL/VFL, cites 2–4 foundational works, identifies the specific gap.
3. **Gap statement** (1 paragraph): explicitly states what prior work does not do. Usually a direct sentence: "most FL evaluations remain confined to the HFL framework" [HybridVFL] or "existing defenses fail because..." [PRIVEE].
4. **Solution and contributions** (1–2 paragraphs): names the method, states 3–4 numbered or bulleted contributions. All 7 papers end the Introduction with an explicit contributions list.
5. **Paper organization** (optional): "The remainder of the paper is organized as follows..." — present in UIFV, absent in others. Skip for our paper (saves space).

### Citation density
- Problem motivation paragraphs: 1–3 citations.
- Gap/related work paragraphs: 3–5 citations.
- Contribution paragraph: 0–1 citations (your own work, briefly cite related).
- Total Introduction citations across all papers: **8–20 citations.**

### Clinical motivation framing (critical for MIDL/MICCAI)
- FedEvPrompt: "wealthier regions having a data advantage" — opens with health equity, not ML.
- HybridVFL: "stringent data privacy regulations" (GDPR) in para 1.
- **Rule**: For our paper, open with why dermoscopy data cannot be centralized (regulatory, patient consent, multi-institutional), then pivot to VFL. Do not open with "Federated learning has emerged as..."

### Gap statement — how to write it
Pattern observed: one crisp declarative sentence stating what doesn't exist.
- "most FL evaluations remain confined to the HFL framework" [HybridVFL]
- "[prior VQ works] do not study the vertical setting and include no reconstruction privacy evaluation" [FedVQCS framing, our paper's usage]
- **For our paper**: "Prior VQ-in-FL work [FedVQCS, FedMPQ] treats quantization as a communication compression strategy; none evaluates the effect of VQ on representation reconstructability."

### Contributions list format
All papers use 3–4 bullet points or numbered items. Length: 1–2 sentences each. Never more than 4 items.
- Verb-first: "We show that...", "We propose...", "We demonstrate..."
- Include at least one empirical claim with a number.
- Final bullet: broader implication or released code.

---

## 3. Related Work

### Structure: separate section vs. integrated in Introduction
- **Separate section** (before Methodology): UIFV, HybridVFL, OPUS-VFL, PRIVEE.
- **Integrated into Introduction** (Sec I-A subsection): FedVQCS.
- FedEvPrompt and FedEM: no standalone Related Work — brief mentions in Introduction.
- **Rule for our paper**: Given the ICML template and space, use a standalone Related Work subsection at the end of Introduction, or a short section before Methodology. 3–4 subsections.

### Subsection organization pattern
HybridVFL uses: (1) VFL basics, (2) Multimodal integration, (3) Disentangled fusion. 
UIFV uses: (1) VFL attacks, (2) VFL defenses, (3) data reconstruction methods.
OPUS-VFL uses: (1) VFL incentive mechanisms, (2) cryptographic privacy, (3) DP-based VFL.

**For our paper, use:**
1. VFL communication compression (Compressed-VFL, SparseVFL, FedVQCS — cite as compression-only, no privacy eval)
2. Privacy attacks in VFL and split learning (UIFV — threat model; URVFL — active, out of scope)
3. VQ in FL (FedVQCS, FedMPQ — compression only, no reconstruction privacy eval)
4. Multimodal VFL fusion (HybridVFL — orthogonal: fusion vs. transmission)

### How to position prior work
Every subsection in these papers ends by explicitly stating the gap that their paper fills. Pattern: "[Prior work] does X. Unlike our work, they do not Y."
- OPUS-VFL: Each subsection ends with "Closing Gaps in Prior Work: Contributions of OPUS-VFL."
- **For our paper**: End each subsection with 1 sentence explaining why that line of work does not address our contribution.

### Citation density in Related Work
- 4–8 citations per subsection (HybridVFL: 8, 5, 4 across three subsections).
- Cite by contribution cluster, not by listing individual papers one by one.
- Do not describe every prior paper in detail — group them.

---

## 4. Methodology

### Structure (based on the already-written main.tex)
Our methodology section is already written and well-structured. Key patterns confirmed by related papers:

**Equation count**: FedVQCS: 10+ equations. HybridVFL: 8. PRIVEE: 12+. OPUS-VFL: 15.
Our paper has 7 equations — appropriate for the contribution scope.

**Notation formalism level**: All papers use formal notation (calligraphic for functions, subscript indexing for parties). Our notation is consistent with this.

**Attacker description placement**: In UIFV (the paper our threat model follows), the attacker is described in a dedicated subsection (3.2) within Methodology. We correctly place InverNetV9 as a Methodology subsection.

**Reviewer trap mitigation**: Our main.tex already includes the explicit sentence: "The product-quantized bottleneck is applied to this 128-dimensional projected representation z_i, not to the 1280-dimensional EfficientNet embedding." This is the right call — observed pattern in other papers: explicitly state what your method does NOT operate on when there is a likely confusion point.

**Algorithm / pseudocode**: FedVQCS and FedEM include algorithm boxes. UIFV does not. For a 6-section ICML-format paper, no pseudocode box needed — equations + prose is sufficient (as we have).

**Baseline description**: HybridVFL uses a 4-row comparison: centralized unimodal → centralized multimodal → federated baseline → proposed. Our Method Grid Table (Tab 1) follows the same logic: A_plain → A_proj/S_sign → H_vq. This is the correct pattern.

### What papers do in Methodology that we should replicate
- State the training loss explicitly with an equation (we do: Eq. 6).
- State the metric used as primary evaluation and why (weighted accuracy for imbalance).
- Describe the attacker separately from the defender (we do: §2.5 InverNetV9 subsection).
- Explicitly state what is NOT changed when proposing a method: "Our method changes only the representation transmitted by the passive image party" (already in our main.tex — good).

---

## 5. Results

### Table vs. figure preference
- FedEM: heavy table reliance, figures for reconstruction examples and accuracy curves.
- OPUS-VFL: primarily tables, no Pareto curve.
- FedEvPrompt: tables for per-client accuracy, figures for data distribution.
- **FedEM is the only paper with a Pareto curve** (Figure 6: privacy-utility tradeoff as perturbation increases).
- **Rule for our paper**: Pareto curve is non-standard but acceptable for showing Pareto dominance of H_vq_K64. It should accompany the main results table, not replace it. Label it as "Figure X: Privacy-utility tradeoff" and reference it from the Results text.

### How papers narrate results tables
Pattern across all papers: **do not walk through every row**. Identify 2–3 key findings from the table and state them explicitly.

OPUS-VFL example: "OPUS-VFL consistently achieves lower adversarial success rates in label inference attacks than other baseline methods." Then one sentence with specific numbers for the headline comparison.

FedEM example: "FedEM outperforms established privacy-preserving techniques in safeguarding privacy." Followed by specific metric comparison for 1–2 key methods.

**Rule for our paper**: 
- 1 sentence stating the overall finding from Table 2 (utility).
- 1–2 sentences citing H_vq_K64's headline numbers (+3.4pp WACC, 853× fewer bits).
- 1 sentence on the equal-bit comparison at 128b (H_vq_M16 vs S_sign_quant).
- 1 sentence on Table 3 (privacy: SSIM/LPIPS).
- Do NOT walk through all 11 rows.

### Numerical density in results paragraphs
All papers: **3–5 specific numbers per results paragraph**. Never more than 5 (becomes hard to read). Never fewer than 2 (too vague).

### How to compare two methods
Template observed: "[Method A] achieves [X] compared to [Method B]'s [Y], a [Z-point / Z×] improvement."
- "14.69 percentage points improvement over concatenation" [HybridVFL] — states absolute pp improvement.
- "0.1 bit per local model entry" [FedVQCS] — states efficiency gain.
- "reduces... by up to 20%" [OPUS-VFL] — states relative change.
- **For our paper**: state absolute pp for WACC ("3.4 percentage points"), use × for bit reduction ("853×"), state absolute for SSIM ("0.622 vs. 0.503").

### Equal-bit comparison framing (critical)
The gap between H_vq_M16 and S_sign_quant on SSIM is 0.002 — too small to claim VQ is more private.
Pattern from PRIVEE: "both PRIVEE-DP and PRIVEE-DP++ maintain 0% accuracy loss while offering encryption-level privacy." They only make strong claims when the evidence supports it.
**Rule**: Frame the 128-bit comparison as: "H_vq_M16 achieves 0.5 percentage points higher WACC than S_sign_quant at equivalent privacy (SSIM 0.540 vs. 0.538), showing that VQ-based transmission does not sacrifice privacy to achieve utility gains."

### Reconstruction examples (figure)
FedEM and UIFV both include side-by-side reconstruction grids. Pattern: show the best and worst reconstruction cases side by side. Caption explicitly states what SSIM value corresponds to each.
**For our paper**: once invernet_grid_regen.ipynb Kaggle run is complete, include 2×4 grid (4 methods × 2 examples per method). Caption must state SSIM values for each image.

### Reporting uncertainty
FedEvPrompt: "77.26±4.65" — reports mean ± std for each method.
HybridVFL: "14.69 percentage points" — no uncertainty on this headline number.
**Rule for our paper**: Report WACC as "0.786 ± 0.007" (2 seeds). SSIM and LPIPS can be reported without ± if they are computed on the full validation set (not averaged over seeds).

---

## 6. Discussion / Ablation

### Where ablation studies appear
- HybridVFL: no ablation section at all (deferred as future work).
- OPUS-VFL: ablation integrated into Results (Sections 5.x).
- UIFV: ablation on attack scenarios in Results.
- FedEM: "Discussions" section after Results, includes limitations.
- **Pattern**: Most papers integrate ablations into Results or Discussion. A standalone "Ablation" section is rare in 6-section ICML-format papers. **For our paper**: present K, M, β, and init ablations as a Discussion subsection titled "Ablation" or within a Discussion section that covers both the regularization mechanism and hyperparameter sensitivity.

### How to frame "why your method works"
Pattern: state a hypothesis, then cite empirical evidence for it.
- HybridVFL: "The minimal cost of privacy for HybridVFL is low, evidenced by only a 0.9-point gap." Evidence cited immediately.
- PRIVEE: "While our proposed piecewise uniform noise mechanism effectively preserves the ranking..." Mechanism explained, then number cited.
- **For our paper**: "VQ acts as a regularizer by stripping high-frequency embedding variation the classifier does not need. This hypothesis is supported by three observations: (1) H_vq_K64 raises the within-class silhouette score from 0.015 (A_plain_vfl) to 0.074; (2) A_proj_vfl, which uses the same 128-d projection without discretization, achieves a negative silhouette score (−0.015), isolating the effect of VQ discretization; (3) codebook utilization of 92–99% rules out codebook collapse as an alternative explanation."

### How papers handle the A_proj_vfl control
HybridVFL frames concatenation baseline explicitly: "validates the criticality of advanced fusion mechanisms." Our A_proj_vfl serves the same role — it isolates the effect of projection from discretization. State this role explicitly in Discussion: "A_proj_vfl demonstrates that 128-d projection alone is insufficient — SSIM 0.585 vs. 0.503 for H_vq_K64 — confirming that privacy improvement requires VQ discretization, not just dimensionality reduction."

### Limitations format
- PRIVEE: 3 explicit points in a "Discussions: Limitations" subsection. Uses "while...it does not..." framing.
- OPUS-VFL: embedded in "Future Work" — constructive, not self-critical.
- FedEM: ~50 words in Discussion, no dedicated subsection.
- HybridVFL: explicit limitations list ("current evaluation is limited to a single dataset," "computational latency deferred," "no formal DP guarantees").
- **Rule for our paper**: 3–4 bullet points or numbered items in a "Limitations" subsection within Discussion. State:
  1. Threat model scope: URVFL-class (active, gradient-based) attacks are out of scope.
  2. Single dataset: ISIC-2019 only; generalization to other modalities is unverified.
  3. No formal privacy bounds: InverNetV9 is an empirical attacker, not adversarially optimal.
  4. Synthetic federation: passive/active party split is constructed, not from separate institutions.
- Each limitation can be 1–2 sentences. Use "While X, Y remains as future work" framing.

---

## 7. Conclusion

### Length and structure
- HybridVFL: ~200 words, 5 future directions in bullets.
- FedEvPrompt: ~180 words, 5 sentences.
- All conclusions: restate what was shown → key finding → limitations that are also future work.
- **Rule**: 150–200 words. No new results. 3–4 future work directions.

### What belongs in Conclusion (pattern)
1. One sentence restating the core Pareto claim (utility + privacy + efficiency simultaneously).
2. One sentence naming the key non-obvious finding (smaller K is a stronger regularizer, not weaker).
3. 3–4 bullet future work items:
   - URVFL-class active attacks (gradient-based)
   - Diffusion-based inversion as a stronger future attacker
   - Cross-domain validation (non-dermoscopy modalities)
   - Formal privacy bounds (DP composability with VQ)

### What NOT to include
- Do not repeat table numbers from Results.
- Do not add new qualitative assessments not in Discussion.
- Do not end with "In conclusion, we propose..." (circular).

---

## 8. Citation Practices

### Citation density by section (observed)
| Section | Citations per paragraph |
|---------|------------------------|
| Abstract | 0 |
| Introduction (motivation paras) | 1–3 |
| Introduction (gap paras) | 3–5 |
| Related Work | 4–8 per subsection |
| Methodology | 1–2 per subsection (for specific techniques only) |
| Results | 0–1 (only to cite metrics like SSIM [wang2004]) |
| Discussion | 1–2 (to cite related work being contrasted) |
| Conclusion | 0–1 |

### What to cite in Methodology (observed pattern)
- The specific technique being used: VQ-VAE for STE [oord2017], product quantization [jegou2011], K-means++ [arthur2007].
- Dataset paper: [combalia2022isic, isic2019challenge].
- Evaluation metrics: SSIM [wang2004ssim], LPIPS [zhang2018lpips].
- Baseline methods: if a baseline comes from a specific prior paper, cite it.
- **Do NOT** cite surveys or general FL papers in Methodology — cite the specific work you use.

### What NOT to cite
- Do not cite your own paper (obviously).
- Do not cite a paper just because it exists in the space — only cite if you directly use or contrast it.
- Do not cite URVFL in Methodology — it's a different threat model; mention in Limitations only.

---

## 9. Writing Style Markers

### Active vs. passive voice
- Results and Discussion: all papers use **active voice** predominantly.
  - "H_vq_K64 achieves..." not "The best result was achieved by..."
  - "We observe that VQ representations show..." not "It is observed that..."
- Methodology: mixed — passive acceptable for describing mathematical operations.
  - "The projected vector is split into M equal subspaces" (passive OK here).

### Sentence length
- Short declarative sentences for results: 10–15 words.
- Longer complex sentences for mechanism descriptions: 20–30 words.
- Never exceed 35 words in a single sentence in Results or Discussion.

### Comparative framing vocabulary (use these, not banned words)
- "achieves X compared to Y's Z"
- "outperforms / surpasses"
- "at equivalent / identical communication cost"
- "with no changes to..."
- "confirming that..."
- "isolating the effect of..."
- "rules out / precludes..."
**Banned words**: delve, crucial, leverage, robust, furthermore, underscore, tapestry.

### How to introduce a baseline
Pattern: state what it does, not what it is named.
- "A_proj_vfl transmits a 128-d learned continuous projection (4,096 bits), serving as a projection-only control." 
- Never: "We compare against A_proj_vfl, which is a baseline method."

### Framing empirical privacy vs. formal guarantees
PRIVEE: "While our proposed mechanism effectively preserves the ranking of class predictions... it does not satisfy formal differential privacy (DP) guarantees."
OPUS-VFL: "Honest-but-curious assumption explicitly stated."
**Rule for our paper**: Acknowledge in one clear sentence in Methodology or Limitations that InverNetV9 provides empirical privacy measurement, not formal (ε,δ)-DP bounds. Do not hedge this fact — state it directly.

### How to describe a negative result
FedVQCS: "significant quantization error is inevitable when the communication overhead must be less than one bit per entry." States the limitation as an observed fact, not an apology.
**For our paper**: The equal-bit SSIM gap (0.002) is not significant. State it directly: "The SSIM difference between H_vq_M16 and S_sign_quant at 128 bits is 0.002, within measurement noise, indicating equivalent empirical privacy at this bit budget."

---

## 10. Medical Imaging–Specific Patterns

### How to introduce ISIC-2019
FedEvPrompt (uses ISIC-2019): dataset described in Methodology / Experimental Setup. Includes total image count, number of classes, data split, and heterogeneity structure. Does NOT describe every class individually.
Template for our paper:
> "We evaluate on ISIC-2019, an 8-class dermoscopic skin-lesion classification benchmark comprising 25,331 images with optional patient metadata. Severe class imbalance (NV dominates; DF, VASC, SCC are minorities) motivates balanced accuracy (WACC) as the primary utility metric, aligned with the ISIC-2019 challenge protocol."

### How to frame synthetic federation
FedEvPrompt: frames it positively as "realistic distribution" without explicitly flagging "simulated." Uses "organized the dataset into [N] nodes, with each node representing data from a specific source."
**For our paper**: Frame the partition explicitly: "The VFL partition assigns dermoscopic images to the passive party and age, sex, anatomical-site metadata with diagnostic labels to the active party. Raw images are never transmitted." This is already in our main.tex — keep it.

### Balanced accuracy / class imbalance reporting
FedEvPrompt: reports WACC with ± std. Does not report per-class accuracy in the main table.
HybridVFL: reports balanced accuracy and notes "0.9-point gap" vs. oracle.
**Rule for our paper**: Report WACC ± std as the primary metric. Tail WACC (4 minority classes) as a secondary metric. No need to report per-class breakdown in the main table — include in Discussion if needed to explain β findings.

### SSIM in a clinical context
FedEM uses SSIM/MSE but not in a medical context — cites [wang2004] for SSIM definition only.
UIFV uses SSIM and explicitly states what the range means: "ranging from 0 to 1 with 1 indicating perfect similarity."
**For our paper**: Define SSIM direction on first use: "We report SSIM [wang2004ssim] and LPIPS [zhang2018lpips]; higher SSIM and lower LPIPS indicate easier reconstruction and therefore weaker empirical privacy." (This is already in our main.tex — good.)

---

## 11. Checklist: Per-Section Standards

Use this checklist when critiquing or reviewing each section of our paper.

### Abstract
- [ ] Single paragraph, 5–6 sentences, ≤150 words
- [ ] Leads with the problem (VFL embedding leakage)
- [ ] Names the method (product quantization bottleneck)
- [ ] Includes at least 2 specific numbers (WACC improvement + SSIM improvement)
- [ ] States the implication in the final sentence
- [ ] No passive "is proposed" constructions
- [ ] None of the 7 banned words

### Introduction
- [ ] Opens with clinical/institutional motivation (≥2 sentences before technical FL content)
- [ ] Identifies the specific gap in 1 direct sentence
- [ ] Ends with explicit contributions list (3–4 bullets, verb-first)
- [ ] Contributions list includes at least 1 number
- [ ] Citation count: 8–15 total across Introduction
- [ ] Does NOT include paper organization sentence ("remainder is organized as follows")

### Related Work
- [ ] 4 subsections: communication compression, privacy attacks, VQ in FL, multimodal fusion
- [ ] Each subsection ends with 1 sentence stating the gap our paper fills
- [ ] HybridVFL cited as orthogonal (fusion vs. transmission)
- [ ] FedVQCS cited as "no reconstruction privacy evaluation"
- [ ] URVFL cited as "different threat model — active, out of scope"
- [ ] No subsection longer than 150 words

### Methodology (already written in main.tex — verify these)
- [ ] States explicitly: "VQ operates on the 128-d projection, NOT the 1280-d embedding"
- [ ] States explicitly: "Our method changes only the representation transmitted by the passive image party"
- [ ] InverNetV9 described in its own subsection
- [ ] SSIM/LPIPS direction defined on first use
- [ ] Threat model: passive/honest-but-curious, no gradient access, no label access
- [ ] Method Grid Table present with all 11 methods + bit counts

### Results
- [ ] Two tables: (1) utility (WACC ± std, bits), (2) privacy (SSIM, LPIPS)
- [ ] Pareto curve figure referenced and present
- [ ] Reconstruction grid figure referenced (once Kaggle run complete)
- [ ] Text narrates ≤3 key findings from tables (does NOT walk through all 11 rows)
- [ ] H_vq_K64 headline: +3.4pp WACC, SSIM 0.503 vs. 0.622, 853× fewer bits
- [ ] Equal-bit comparison framed as "VQ wins utility at equivalent privacy" (NOT "VQ is more private")
- [ ] Numbers density: 3–5 per paragraph

### Discussion
- [ ] Subsection 1: Regularization mechanism (silhouette evidence + A_proj_vfl = −0.015)
- [ ] Subsection 2: Ablation findings (K, M, β, init)
- [ ] Subsection 3: Limitations (4 points: URVFL scope, single dataset, no DP bounds, synthetic federation)
- [ ] Active voice throughout
- [ ] Silhouette table included (7 methods, scores, codebook util, bits)
- [ ] t-SNE figure referenced

### Conclusion
- [ ] 150–200 words
- [ ] Restates Pareto claim without new numbers
- [ ] States key non-obvious finding (smaller K = stronger regularizer)
- [ ] 3–4 future work directions
- [ ] No new results or qualitative assessments not in Discussion

---

*Last updated: 2026-05-08. Source papers fetched and analyzed from arXiv. Add to this file if additional papers are analyzed.*
