# PAD-UFES-20 External Validation Plan for v9

This document defines how to use PAD-UFES-20 as an external validation dataset for the v9 VQ-only VFL privacy study.

The goal is not to invent a new method for PAD. The goal is to test whether the same v9 mechanism, a product-quantized passive image representation in VFL, gives the same kind of utility/privacy behavior on an independent skin-lesion dataset with different image acquisition and richer clinical metadata.

Sources:
- PAD-UFES-20 dataset paper: https://pmc.ncbi.nlm.nih.gov/articles/PMC7479321/
- PAD-UFES-20 Mendeley dataset page: https://data.mendeley.com/datasets/zr7vgbcyr2/1
- Related clinical-metadata paper: https://doi.org/10.1016/j.compbiomed.2019.103545
- Kaggle mirror: https://www.kaggle.com/datasets/mahdavi1202/skin-cancer

---

## 1. Why PAD-UFES-20 Is Worth Running

PAD-UFES-20 is the best next dataset for this project because it matches the current v9 architecture with minimal changes:

| v9 ISIC-2019 setup | PAD-UFES-20 equivalent |
|---|---|
| Passive party owns lesion image | Passive party owns clinical smartphone lesion image |
| Active party owns patient metadata | Active party owns structured clinical metadata |
| Active party owns label | Active party owns diagnostic label |
| Task is skin-lesion diagnosis | Task is skin-lesion diagnosis |
| Privacy attack reconstructs passive image from transmitted representation | Same reconstruction attack can be used |

This makes PAD much stronger than jumping directly to TCGA, MIMIC, or multimodal pathology. Those are valuable later, but they would force new encoders, new label spaces, new preprocessing, and possibly a new VFL architecture. PAD tests the current v9 idea directly.

PAD also strengthens the paper because it is not merely another ISIC split. ISIC-2019 uses dermoscopy-style images and limited metadata. PAD uses smartphone clinical images with real acquisition variability: different devices, lighting, resolutions, and clinical backgrounds. If VQ still improves the privacy/utility tradeoff here, the paper can claim external validation across a different skin-imaging acquisition regime.

The claim PAD can support:

> The learned VQ communication bottleneck generalizes from dermoscopic skin-lesion VFL to smartphone clinical skin-lesion VFL, maintaining diagnostic utility while reducing reconstructability of the passive image representation.

The claim PAD cannot support by itself:

> The method works for all medical imaging.

PAD is still dermatology. It proves external acquisition-domain robustness, not universal medical-domain generality.

---

## 2. Dataset Description

PAD-UFES-20 contains:

- 2,298 clinical skin-lesion images.
- 1,373 patients.
- 1,641 skin lesions.
- Six diagnosis labels:
  - ACK: actinic keratosis.
  - BCC: basal cell carcinoma.
  - MEL: malignant melanoma.
  - NEV: melanocytic nevus.
  - SCC: squamous cell carcinoma, including Bowen's disease / SCC in situ.
  - SEK: seborrheic keratosis.
- Up to 21 patient or lesion clinical features.
- 26 metadata CSV attributes total, including IDs, diagnostic label, and biopsy indicator.
- PNG images collected by smartphone devices.
- CC BY 4.0 license.

Published class counts:

| Class | Samples | Biopsy Proven |
|---|---:|---:|
| ACK | 730 | 24.4% |
| BCC | 845 | 100% |
| MEL | 52 | 100% |
| NEV | 244 | 24.6% |
| SCC | 192 | 100% |
| SEK | 235 | 6.4% |
| Total | 2,298 | 58.4% |

The class imbalance is severe, especially for MEL. Balanced accuracy must be the main utility metric. Plain accuracy should not be used as a headline metric.

---

## 3. Research Goal on PAD

The PAD run should answer four concrete questions:

1. Does VQ still preserve diagnostic utility on an independent skin-lesion dataset?
2. Does VQ reduce image reconstruction quality relative to continuous VFL?
3. Does VQ provide privacy beyond simple dimension reduction?
4. Does learned VQ beat naive sign quantization at the same bit budget?

The minimum successful paper story is:

```text
A_plain_vfl -> high utility, high reconstructability, 40960 bits
A_proj_vfl  -> controls for 128-d projection alone, 4096 bits
S_sign_quant -> naive 128-bit discrete baseline
H_vq_*      -> learned discrete bottleneck, 32-128 bits
```

The central PAD figure should match the ISIC figure:

```text
x-axis: reconstruction SSIM, lower is more private
y-axis: balanced accuracy, higher is better
point size or label: transmitted bits
color: method family
```

If VQ points move up-left or stay high-left relative to `A_plain_vfl` and `A_proj_vfl`, PAD supports the novelty.

---

## 4. VFL Partition to Use

Use the same conceptual partition as v9:

### Passive image party

Owns:
- Raw lesion image.

Computes:
- EfficientNet-B0 image embedding.
- Optional 128-d projection.
- Optional sign quantization.
- Optional product quantization.

Transmits:
- Continuous embedding for `A_plain_vfl` and `A_proj_vfl`.
- Sign vector for `S_sign_quant`.
- Codebook-derived 128-d quantized representation for VQ methods during training.
- Report communication bits as in v9:
  - `A_plain_vfl`: 1280 x 32 = 40960 bits.
  - `A_proj_vfl`: 128 x 32 = 4096 bits.
  - `S_sign_quant`: 128 bits.
  - VQ: `M * log2(K)` bits.

### Active metadata and label party

Owns:
- Non-leaky clinical metadata.
- Diagnostic label.

Computes:
- Metadata MLP.
- Final classifier.
- Task loss.

This is a realistic VFL simulation: a dermatology imaging source holds clinical photographs, while the clinical system holds structured anamnesis/exam metadata and diagnostic outcomes.

---

## 5. Fields to Exclude

These fields must not be used as model inputs.

| Field | Use? | Reason |
|---|---|---|
| `patient_id` | No | Identifier. Use only for grouped splitting. |
| `lesion_id` | No | Identifier. Potential leakage across repeated images. Use only for audits. |
| `img_id` | No | Identifier and image lookup key only. |
| `diagnostic` | No | This is the target label. |
| `biopsed` / `biopsy` | No | Severe label proxy: BCC/SCC/MEL are 100% biopsied, while benign classes are much less biopsied. |
| `background_father` | No for primary | Ancestry/protected proxy, high missing/unknown risk, and likely site-specific shortcut. |
| `background_mother` | No for primary | Same as above. |

The most important exclusion is `biopsed`. Including it would make the experiment look stronger but would be methodologically indefensible. It lets the active party infer cancer-like labels from the diagnostic pathway rather than from image/clinical phenotype.

IDs must still be preserved in the dataframe for splitting and image lookup, but never passed into the model.

---

## 6. Metadata Feature Sets

Run PAD in two metadata settings.

### 6.1 Primary: PAD-core+skin

This is the main paper setting because it is closest to ISIC-2019 metadata.

Use:
- `age`
- `gender`
- `region`
- `fitspatrick`

Rationale:
- `age`, `gender`, and lesion location are direct analogues of ISIC age/sex/site.
- `fitspatrick` is clinically meaningful, non-leaky dermatology context. It is not present in ISIC, so describe the setting as "ISIC-like metadata plus Fitzpatrick skin type".

Use this as the primary PAD setting:

```text
PAD-core+skin = age + gender + region + fitspatrick
```

Optional strict comparability setting:

```text
PAD-core = age + gender + region
```

Run `PAD-core` without Fitzpatrick only if compute allows. It is useful for a strict ISIC-like comparison, but the primary PAD experiment should include Fitzpatrick because it is a legitimate clinical feature in PAD and does not leak the label.

### 6.2 Secondary: PAD-clinical

This tests whether the method still behaves properly when the active party has richer clinical information.

Use:
- `age`
- `gender`
- `region`
- `fitspatrick`
- `diameter_1`
- `diameter_2`
- `smoke`
- `drink`
- `pesticide`
- `skin_cancer_history`
- `cancer_history`
- `itch`
- `grew`
- `hurt`
- `changed`
- `bleed`
- `elevation`

Do not include in PAD-clinical:
- `biopsed`
- IDs
- `diagnostic`
- `background_father`
- `background_mother`
- `has_piped_water`
- `has_sewage_system`

Rationale:
- Symptoms and lesion measurements are clinically plausible active-party features.
- Risk factors such as pesticide exposure and cancer history are plausible clinical metadata.
- Water/sewage and parental background are more socioeconomic/ancestry-like. They can create reviewer concerns about spurious shortcuts and fairness. Keep them out unless you explicitly run a separate sensitivity analysis.

### 6.3 Optional: PAD-full-sensitivity

Only use this as an appendix sensitivity check, not the main result.

Add:
- `background_father`
- `background_mother`
- `has_piped_water`
- `has_sewage_system`

If this improves performance, do not use it as the headline. Report it as a sensitivity result and discuss confounding.

---

## 7. Metadata Preprocessing

Fit all preprocessing on the training split only. Apply learned encoders/imputers to validation/test.

### Numeric fields

Numeric:
- `age`
- `diameter_1`
- `diameter_2`
- Possibly `fitspatrick` if treated ordinally.

Recommended:
- Convert to float.
- Impute missing numeric values with training median.
- Add missingness indicators for `diameter_1` and `diameter_2`.
- Standardize numeric fields using training mean/std.

For `fitspatrick`, safer option:
- Treat as categorical one-hot, not continuous, because Fitzpatrick type is ordinal but not linearly spaced.

### Categorical fields

Categorical:
- `gender`
- `region`
- `fitspatrick` if categorical.
- `background_father` and `background_mother` only in optional sensitivity.

Recommended:
- Strip whitespace.
- Lowercase categories.
- Convert blank, NA, and null to `missing`.
- Keep `UNK` as `unknown`, not as missing.
- One-hot encode with train-fitted categories.
- Use `handle_unknown='ignore'` behavior for validation/test categories unseen during training.

### Boolean fields

Boolean:
- `smoke`
- `drink`
- `pesticide`
- `skin_cancer_history`
- `cancer_history`
- `itch`
- `grew`
- `hurt`
- `changed`
- `bleed`
- `elevation`
- `has_piped_water` only in optional full sensitivity.
- `has_sewage_system` only in optional full sensitivity.

Recommended:
- Do not convert blank to false.
- Encode as three-state categorical: `true`, `false`, `missing_or_unknown`.
- One-hot encode each boolean field into 2 or 3 columns depending on observed values.

This matters because "not answered" is not the same as "no".

---

## 8. Image Preprocessing

The architecture should remain v9:

- EfficientNet-B0 passive backbone.
- ImageNet normalization.
- 224 x 224 encoder input.
- Same passive method families.

PAD images are raw smartphone PNGs with variable size, aspect ratio, lighting, and lesion scale. Do not naively center-crop the raw image before resizing; that can remove lesion context or the lesion itself.

Recommended cache creation:

1. Load PNG.
2. Apply EXIF transpose if applicable.
3. Convert to RGB.
4. Resize with aspect ratio preserved.
5. Pad to square.
6. Save/cache as 256 x 256 RGB array.

Then reuse the v9 transform structure on the square cache, but make the PAD crop lesion-safe.

Train:
- Random resized crop to 224 with a conservative scale, recommended `scale=(0.70, 1.0)`.
- Horizontal flip.
- Vertical flip.
- Mild color jitter.
- Normalize with ImageNet mean/std.

Validation:
- Center crop to 224.
- Normalize with ImageNet mean/std.

Why not blindly copy the ISIC `scale=(0.08, 1.0)` crop?

PAD images are raw clinical photographs. Even after square padding, aggressive random crops can remove the lesion or train on mostly background skin. That would add avoidable noise and make the external validation less meaningful. This is a dataset-preprocessing adaptation, not a method change: every method receives the same image pipeline, and the passive architecture is unchanged.

Before training final runs, do a crop audit:
- Render 100 random train crops.
- Confirm the lesion remains visible in most crops.
- If too many crops lose the lesion, increase the lower scale bound to `0.80`.
- Save the audit grid for reproducibility.

Important: the reconstruction target must mirror the encoder's validation spatial extent. If the encoder sees a center-cropped 224 region, InverNet must reconstruct that same 224 region resized to 64 x 64. Do not reconstruct the full raw image if the encoder never saw the full raw image.

For PAD, absolute SSIM may be lower than ISIC because smartphone backgrounds and lighting are less standardized. The paper should emphasize relative SSIM/LPIPS against `A_plain_vfl` and `A_proj_vfl`, not an absolute SSIM threshold.

---

## 9. Splitting Protocol

Use patient-safe splitting. This is non-negotiable.

Reason:
- PAD has 1,373 patients, 1,641 lesions, and 2,298 images.
- A patient may have multiple lesions.
- A lesion may have multiple images.
- If the same patient appears in train and validation, the model can exploit patient-level shortcuts.

Primary split rule:

```text
Group by patient_id.
Stratify approximately by diagnostic.
No patient_id may appear in more than one split.
```

Recommended implementation:
- Use `StratifiedGroupKFold` if available.
- If not available, implement a greedy group-stratified split.
- Validate class counts after split.

Minimum acceptable paper split:
- One fixed patient-group stratified train/validation split.
- 80/20 or 75/25.
- Two seeds per method.

Better final-paper split:
- Three patient-group stratified folds.
- Same method grid on each fold.
- Report mean +/- std across folds.

Rare-class validation check:
- Validation must contain MEL.
- Target at least 8-10 MEL samples in validation for a single 80/20 split.
- If a split gives too few MEL samples, discard and regenerate the split with a fixed documented seed.

Do not use image-level random splits.
Do not use lesion-level splits if patient leakage remains possible.

---

## 10. Model and Training Changes

Keep the method exactly v9 except for dataset-specific input and label dimensions.

Change:
- `NUM_CLASSES = 6`
- `CLASS_NAMES = ['ACK', 'BCC', 'MEL', 'NEV', 'SCC', 'SEK']` or whatever sorted/encoded order is chosen.
- `META_DIM` comes from PAD-core+skin or PAD-clinical preprocessing.
- Dataset loader reads PNG images and metadata CSV instead of ISIC CSV/cache.
- Fold file uses patient-group split indices.

Keep:
- EfficientNet-B0 backbone.
- Projection dimension 128.
- Product quantizer implementation.
- Sign baseline.
- Active metadata MLP shape unless metadata dimension becomes very large.
- Class-weighted cross entropy.
- Balanced accuracy selection.
- Reconstruction attacker protocol.
- Communication-bit accounting.
- Stage A/B/C separation.

Class weights:
- Start with the current v9 inverse-frequency class weights.
- Because MEL has only 52 samples, log the weights explicitly.
- If training becomes unstable, run a sensitivity with capped class weights, but do not silently change this.

Tail classes:
- For PAD, define tail classes as the four smallest classes by training-set count.
- Expected tail classes from full data: MEL, SCC, SEK, NEV.
- Report tail balanced recall as in v9.

---

## 11. Method Grid

Do not start with all 11 methods. Run the paper-critical grid first.

### Minimum PAD grid

| Method | Bits | Why it is needed |
|---|---:|---|
| `A_plain_vfl` | 40960 | Continuous leakage baseline. |
| `A_proj_vfl` | 4096 | Tests whether 128-d projection alone explains privacy. |
| `S_sign_quant` | 128 | Naive same-bit discrete baseline. |
| `H_vq_K64` | 48 | Current strongest ISIC utility point. |
| `H_vq_K256` | 64 | Main reference VQ setting. |
| `H_vq_M4` | 32 | Most compressed VQ point. |
| `H_vq_M16` | 128 | Equal-bit comparison with sign quantization. |
| `H_vq_commit_high` | 64 | Current high-performing beta setting. |

### Optional later methods

| Method | Why optional |
|---|---|
| `S_rand_sign` | Useful but not essential if compute is tight. |
| `H_vq_no_kmeans` | Good for mechanism ablation, not necessary for external validation. |
| `H_vq_commit_low` | Useful for beta curve, lower priority than K/M/equal-bit comparisons. |

### Extra controls for paper robustness

These are not part of the exact v9 method, but they are important controls:

| Control | Purpose |
|---|---|
| `M_meta_only` | Tests whether PAD metadata alone solves the task. |
| `I_image_only` | Tests how much utility comes from image alone. |

If `M_meta_only` is close to VFL performance, the paper can still claim image-reconstruction privacy, but the diagnostic-utility story must be written more carefully: the active metadata party already has strong signal, so image transmission contributes less.

---

## 12. Reconstruction Privacy Evaluation

Use the same Stage B logic:

- Freeze trained passive client.
- Collect transmitted representation under each method.
- Train InverNetV9 from representation to 64 x 64 image target.
- Use MSE + 0.1 * LPIPS.
- Use 50 epochs for final paper runs.
- Report SSIM and LPIPS.
- Save aligned visual grids.

For PAD, visual grids are especially important because clinical photographs can contain background, rulers, lighting artifacts, or surrounding skin context. A lower SSIM may reflect background mismatch rather than lesion privacy. The visual grid should answer:

- Is lesion shape recovered?
- Is lesion color recovered?
- Is border irregularity recovered?
- Is clinically meaningful texture recovered?
- Does VQ remove lesion-specific information beyond projection?

Use the same grid samples for every method within a split.

Sample selection for grids:
- Include at least one MEL if available.
- Include SCC or SEK.
- Include one BCC.
- Include one ACK.
- Avoid cherry-picking only the best-looking privacy cases.
- Define sample IDs before looking at reconstructions.

---

## 13. Expected Outcomes

### Utility

Most likely:
- VQ should remain competitive with continuous VFL.
- VQ may improve WACC through regularization because PAD is small and noisy.
- Results will have higher variance than ISIC due to only 2,298 images and 52 MEL samples.

Potential issue:
- `A_proj_vfl` may be strong because projection alone regularizes the small dataset.

If `A_proj_vfl` is close to VQ:
- The paper should say VQ improves communication and reconstruction privacy beyond projection only if Stage B confirms lower SSIM/LPIPS.
- Do not claim utility superiority if utility is tied.

### Reconstruction privacy

Most likely:
- `A_plain_vfl` is easiest to reconstruct.
- `A_proj_vfl` is harder than plain.
- VQ and sign are harder than projection.
- Absolute SSIM may be lower than ISIC across all methods because PAD images are less standardized.

The key privacy comparison is:

```text
A_plain_vfl SSIM - H_vq_K64/H_vq_M4 SSIM
A_proj_vfl SSIM  - H_vq_K64/H_vq_M4 SSIM
S_sign_quant SSIM vs H_vq_M16 SSIM at 128 bits
```

### Paper-positive result

Strongest outcome:

```text
H_vq_K64 or H_vq_M4:
  WACC >= A_plain_vfl or close to it
  SSIM < A_proj_vfl
  LPIPS > A_proj_vfl
  bits <= 64
```

Equal-bit positive outcome:

```text
H_vq_M16:
  WACC >= S_sign_quant
  SSIM <= S_sign_quant
  LPIPS >= S_sign_quant
  both at 128 bits
```

This directly supports learned VQ over naive sign coding.

### Paper-negative but still useful result

If VQ reduces reconstruction but costs utility:
- The paper becomes a privacy/utility tradeoff study.

If VQ utility improves but reconstruction does not improve:
- The paper becomes a compression/regularization study, not a privacy paper.

If sign matches VQ at equal bits:
- The learned-codebook novelty weakens.
- You can still claim discrete bottlenecks help, but not that learned VQ is clearly superior.

---

## 14. Paper Language If PAD Works

Use this kind of wording:

> To test whether the observed VQ bottleneck behavior was specific to ISIC-2019 dermoscopy, we repeated the full VFL protocol on PAD-UFES-20, an independent smartphone clinical skin-lesion dataset with patient-level metadata. We preserved the passive image encoder, active metadata classifier, communication-bit accounting, and reconstruction-attack protocol, changing only the dataset loader and label/metadata preprocessing. Patient-level grouped splits were used to avoid identity leakage, and biopsy status was excluded because it is a label-proxy field.

Avoid:

> This proves the method works on medical datasets generally.

Better:

> This supports external validity across two skin-lesion acquisition regimes: dermoscopy-style ISIC images and smartphone clinical PAD images.

---

## 15. Required Tables and Figures

### Dataset table

Include:
- Number of patients, lesions, images.
- Number of classes.
- Class counts.
- Metadata feature set.
- Split protocol.
- Excluded leakage fields.

### Utility table

For PAD-core+skin and optionally PAD-clinical:

| Method | Bits | WACC | Tail WACC | Macro AUC | Seeds/Folds |
|---|---:|---:|---:|---:|---:|

### Privacy table

| Method | Bits | SSIM | LPIPS | InverNet epochs |
|---|---:|---:|---:|---:|

### Pareto figure

One figure per dataset:
- ISIC-2019.
- PAD-core+skin.

Optional combined figure:
- All datasets in panels.

### Visual reconstruction grid

Rows:
- `A_plain_vfl`
- `A_proj_vfl`
- `S_sign_quant`
- `H_vq_K64`
- `H_vq_M4`
- `H_vq_M16`

Columns:
- Original.
- Reconstruction.
- Difference.

---

## 16. Implementation Checklist

1. Download PAD from Mendeley or Kaggle.
2. Verify image count and metadata row count.
3. Confirm every `img_id` maps to exactly one PNG.
4. Standardize diagnosis labels to six classes.
5. Remove or ignore any BOD label if present by mapping it into SCC, matching the paper.
6. Build patient-group stratified splits.
7. Verify no patient overlap across splits.
8. Verify no lesion overlap across splits.
9. Verify validation has MEL and SCC.
10. Build PAD-core+skin metadata matrix.
11. Build PAD-clinical metadata matrix.
12. Save preprocessing audit JSON:
    - feature names.
    - category levels.
    - missing counts.
    - class counts per split.
    - excluded fields.
13. Build 256 x 256 square image cache.
14. Train minimum method grid.
15. Run Stage B reconstruction for all minimum methods.
16. Produce tables and Pareto figures.
17. Run metadata-only and image-only controls.
18. If PAD-core+skin supports the claim, optionally run PAD-clinical.

---

## 17. Second-Pass Audit

This section checks the plan for hidden mistakes or reviewer traps.

### 17.1 Does PAD actually match the current v9 architecture?

Yes. PAD has image plus tabular metadata plus single diagnostic label. The v9 architecture only needs:

- image tensor,
- metadata vector,
- class label.

No new encoder type is required.

### 17.2 Is PAD independent enough from ISIC?

Partly yes. It is still dermatology, but it is clinically different:

- smartphone clinical images instead of dermoscopy-style images,
- Brazilian clinical program instead of ISIC challenge/archive,
- richer clinical metadata,
- different class distribution.

So it supports external skin-lesion validation, not cross-specialty validation.

### 17.3 Is there label leakage?

There would be if `biopsed` were included. The plan excludes it.

There would also be possible leakage or memorization if IDs were included. The plan excludes IDs from model inputs and uses patient grouping for splits.

### 17.4 Is patient leakage handled?

Yes, if and only if splitting is grouped by `patient_id`. Image-level random split is invalid for this dataset.

### 17.5 Is lesion leakage handled?

Yes, because patient grouping is stricter than lesion grouping. If all samples from a patient stay in one split, all lesions and repeated images from that patient also stay in one split.

### 17.6 Is biopsy status okay to mention?

Yes, as dataset description and limitation. No, as model input.

The paper should explicitly state:

> Biopsy status was excluded from model inputs because it is entangled with diagnostic class.

### 17.7 Could metadata dominate the model?

Yes, especially in PAD-clinical. That is why PAD-core+skin is primary and metadata-only control is required.

If metadata-only is strong, the image-side privacy claim still matters, but the paper must not imply the image party is the only useful party.

### 17.8 Could clinical images make reconstruction artificially hard?

Yes. PAD images are less standardized than ISIC, so all methods may reconstruct worse. That is why relative comparison against `A_plain_vfl` and `A_proj_vfl` is more important than absolute SSIM.

### 17.9 Could VQ look good only because the dataset is small?

Possibly. Small data can make any bottleneck act as a regularizer. This does not invalidate the result, but the paper should frame utility gains as empirical regularization, not guaranteed optimization superiority.

### 17.10 Is class imbalance severe enough to break conclusions?

It can. MEL has only 52 samples. Use balanced accuracy, tail recall, class-weighted loss, and patient-stratified splits with validation MEL count checks.

### 17.11 Should PAD replace ISIC?

No. PAD should be an external validation dataset, not the primary dataset. ISIC remains the larger primary benchmark. PAD shows robustness.

### 17.12 Should TCGA/MIMIC be run before PAD?

No. TCGA/MIMIC would be valuable but would require new data engineering and likely new encoders. PAD answers the instructor's request faster and more cleanly: "Does the same idea work on another dataset?"

---

## 18. Final Recommendation

Run PAD-UFES-20.

Use PAD-core+skin first, with patient-group splits and the minimum method grid. Exclude biopsy status, identifiers, parental background, and socioeconomic fields from the main model. Run Stage B reconstruction for every method in the minimum grid. Add metadata-only and image-only controls so reviewers cannot argue that the PAD result is a metadata shortcut.

If PAD shows the same broad Pareto pattern as ISIC, it becomes strong external validation for the paper:

```text
ISIC-2019: dermoscopy-like skin lesion VFL
PAD-UFES-20: smartphone clinical skin lesion VFL
Same VQ bottleneck, same privacy attack, same communication accounting
```

That is a clean and defensible paper story.
