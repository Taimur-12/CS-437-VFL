# Latest Dataset Recommendations for v9 External Validation

Date checked: 2026-05-08.

This note answers a specific question: if the instructor wants the latest dataset/work in the area, what should be used with the current v9 VQ-only VFL architecture?

The answer is:

1. Use **ISIC 2024 / SLICE-3D** as the main latest dataset.
2. Keep **PAD-UFES-20** as a clean external validation dataset, but do not present it as latest.
3. Use **SkinDisNet 2025** only if a very recent non-cancer dermatology dataset is needed.
4. Use **BRSET 2024** only if the paper needs a cross-specialty ophthalmology validation.

---

## 1. Best Latest Fit: ISIC 2024 / SLICE-3D

### Why this is the best answer to the instructor

ISIC 2024 / SLICE-3D is the strongest "latest dataset with existing work" for the current project.

It is recent, public, large, clinically relevant, and already actively studied:

- The SLICE-3D dataset descriptor was published in Scientific Data on 2024-08-14.
- It contains over 400,000 skin lesion crops extracted from 3D total-body photography.
- It was the official training dataset for the ISIC 2024 Skin Cancer Detection with 3D-TBP challenge.
- A 2025 npj Digital Medicine paper analyzed the ISIC 2024 competition results and ablations.
- A 2025 Scientific Reports paper used ISIC 2024 for explainable multimodal risk prediction with 3D TBP images plus structured clinical/lesion features.

Sources:
- SLICE-3D Scientific Data paper: https://www.nature.com/articles/s41597-024-03743-w
- ISIC 2024 challenge dataset page: https://challenge2024.isic-archive.com/
- ISIC 2024 background: https://challenge2024.isic-archive.com/background/
- npj Digital Medicine 2025 ISIC 2024 competition analysis: https://www.nature.com/articles/s41746-025-02070-7
- Scientific Reports 2025 ISIC 2024 multimodal paper: https://www.nature.com/articles/s41598-025-33536-z

### Why it works with v9

The dataset maps very cleanly to the v9 VFL split:

| v9 component | ISIC 2024 / SLICE-3D mapping |
|---|---|
| Passive party | 3D-TBP lesion crop image |
| Active party | structured metadata |
| Active label | `target`, malignant vs benign |
| Passive model | EfficientNet-B0 image encoder |
| Transmitted representation | continuous / projection / sign / VQ representation |
| Privacy attack | reconstruct lesion crop from transmitted representation |

Only minor changes are needed:

- `NUM_CLASSES = 2`.
- Use binary classification metrics in addition to balanced accuracy.
- Use patient-group splits by `patient_id`.
- Handle extreme class imbalance carefully.

### Why this is better than PAD for "latest"

PAD-UFES-20 is from 2020. It is excellent for a clean second dataset, but it is not latest.

ISIC 2024 / SLICE-3D is newer and has a much stronger current-literature trail:

- 2024 dataset descriptor.
- 2024 Kaggle/ISIC challenge.
- 2025 competition analysis.
- 2025 multimodal explainability paper.
- Related 2026 image-plus-metadata fusion work showing the field is actively moving toward image+metadata skin-lesion diagnosis.

### What novelty it lets us claim

Existing ISIC 2024 work focuses on centralized image+metadata fusion, triage, and explainability.

It does not evaluate:

- vertical federated splitting,
- passive image-party privacy,
- reconstruction attacks from intermediate image embeddings,
- learned product-quantized communication bottlenecks.

That is exactly the gap v9 fills.

Good paper wording:

> To test the proposed VQ bottleneck on a recent large-scale benchmark, we evaluate on ISIC 2024 / SLICE-3D, a 3D total-body-photography skin cancer dataset used in the 2024 ISIC Grand Challenge and subsequent 2025 multimodal AI studies. Unlike prior centralized image-plus-metadata models, we simulate a vertical split in which the image holder transmits only a compressed representation to a metadata-and-label holder, and we evaluate reconstruction privacy from the transmitted representation.

---

## 2. Critical ISIC 2024 Leakage Rules

ISIC 2024 has many metadata fields. Some are safe clinical metadata; others are image-derived or diagnosis-derived and should not be used in the primary VFL setting.

### Use in primary setting: ISIC2024-core

Use only:

- `age_approx`
- `sex`
- `anatom_site_general`

Optional but keep separate:

- `clin_size_long_diam_mm`

Reason: this is closest to the ISIC-2019 metadata setup and avoids giving the active party proprietary image-derived lesion measurements.

### Do not use as model input

Never use:

- `target` except as label.
- `patient_id` except for grouped splitting.
- `isic_id` except for image lookup.
- `lesion_id` except for leakage audits.
- `iddx_full`, `iddx_1`, `iddx_2`, `iddx_3`, `iddx_4`, `iddx_5`.
- `mel_mitotic_index`.
- `mel_thick_mm`.
- `diagnosis_*` fields if present.
- `concomitant_biopsy` if present.
- `diagnosis_confirm_type` if present.
- `tbp_lv_dnn_lesion_confidence`.

These are label-proxy or diagnosis-process fields and would make the evaluation indefensible.

### Do not use in primary active metadata

Exclude all `tbp_lv_*` fields in the primary experiment:

- `tbp_lv_A`, `tbp_lv_B`, `tbp_lv_C`, etc.
- color, border, asymmetry, x/y/z location, perimeter, area, DNN lesion confidence.

Reason: these are WB360-derived appearance measurements extracted from the imaging system. In a clean VFL threat model, the active party should not receive detailed image-derived measurements if the passive image party is the private data holder.

Use them only in an explicitly labeled sensitivity setting:

```text
ISIC2024-rich-WB360 = age/sex/site + tbp_lv_* appearance metadata
```

Do not make this the main result.

---

## 3. ISIC 2024 Experiment Protocol

### Main task

Binary skin cancer detection:

```text
target = 1 malignant / skin cancer
target = 0 benign
```

### Metrics

Use:

- AUROC.
- AUPRC.
- balanced accuracy.
- sensitivity at fixed specificity.
- pAUC above 80% TPR if feasible, matching the challenge spirit.
- reconstruction SSIM and LPIPS.

Do not rely on plain accuracy because the dataset is extremely imbalanced.

### Splits

Use grouped splits:

```text
group = patient_id
```

No patient may appear in both train and validation.

Recommended:

- Create one fixed patient-group validation split.
- Ensure validation contains enough positives.
- If compute allows, use three patient-group folds.

### Class imbalance

Do not train naively on all negatives with standard cross-entropy.

Recommended:

- Keep the validation set at natural prevalence.
- For training, use balanced or semi-balanced sampling.
- For each epoch, sample all positives plus a controlled negative ratio, such as 1:10 or 1:20.
- Report that training used negative subsampling but validation used natural prevalence.

### Method grid

Minimum grid:

| Method | Why |
|---|---|
| `A_plain_vfl` | continuous leakage baseline |
| `A_proj_vfl` | projection-only privacy control |
| `S_sign_quant` | naive 128-bit discrete baseline |
| `H_vq_K64` | current best ISIC-2019 utility point |
| `H_vq_K256` | reference VQ setting |
| `H_vq_M4` | 32-bit extreme compression |
| `H_vq_M16` | equal-bit comparison vs sign |

Optional:

- `H_vq_commit_high`
- `H_vq_no_kmeans`

### Expected result

Do not expect WACC behavior to match ISIC-2019 exactly because the task is binary and extremely imbalanced.

The result we want:

```text
VQ preserves AUROC / balanced accuracy relative to A_proj_vfl
VQ reduces SSIM relative to A_plain_vfl and A_proj_vfl
VQ transmits 32-64 bits instead of 4096-40960 bits
```

If this happens, it is a strong latest-dataset proof.

---

## 4. Second Best Latest Dermatology Dataset: SkinDisNet 2025

SkinDisNet is very recent and public:

- Data in Brief article accepted 2025-10-28 and published online 2025-11-03.
- Mendeley Data version 2 published 2025-06-26.
- 1,710 smartphone clinical images.
- 416 patients.
- Six skin disease classes:
  - Atopic Dermatitis.
  - Contact Dermatitis.
  - Eczema.
  - Scabies.
  - Seborrheic Dermatitis.
  - Tinea Corporis.
- Seven metadata attributes.

Sources:
- PubMed/Data in Brief record: https://pubmed.ncbi.nlm.nih.gov/41323762/
- ScienceDirect page: https://www.sciencedirect.com/science/article/pii/S2352340925009606
- Mendeley dataset: https://data.mendeley.com/datasets/yj3md44hxg/

### Why it works with v9

It has:

- image,
- tabular metadata,
- diagnosis label,
- patient references.

So the v9 architecture fits directly.

### Why it is not my first recommendation

It is small. It is not cancer detection. It is a region-specific skin disease classification dataset. It is useful as a "very recent non-cancer dermatology" validation, but it is weaker as the main proof of the paper than ISIC 2024.

Critical rule:

- Use only the original/preprocessed images.
- Do not mix augmented images across splits.
- Split by patient, not image.

Good use:

```text
Appendix / secondary validation:
Does the VQ bottleneck also behave correctly on a 2025 smartphone skin-disease dataset outside cancer?
```

---

## 5. Best Cross-Specialty Option: BRSET 2024

BRSET is a strong recent ophthalmology option:

- Published in PLOS Digital Health on 2024-07-11.
- 16,266 color fundus photos.
- 8,524 Brazilian patients.
- Metadata includes age, sex, nationality, clinical history, insulin use, diabetes duration, eye side, camera, and more.
- Labels include diabetic retinopathy and other retinal findings.
- The dataset paper includes validation experiments using modern vision models.

Sources:
- PLOS Digital Health paper: https://journals.plos.org/digitalhealth/article?id=10.1371/journal.pdig.0000454
- PMC mirror: https://pmc.ncbi.nlm.nih.gov/articles/PMC11239107/
- PhysioNet dataset link from paper: https://physionet.org/content/brazilian-ophthalmological/1.0.0/

### Why it works

It maps to:

- passive image party = fundus photo,
- active party = demographics/clinical metadata,
- label = diabetic retinopathy or retinal disease target.

### Why it is not first

It is multi-label / ophthalmology. To use it cleanly, v9 needs a task adaptation:

- binary diabetic retinopathy target, or
- 3-class DR target, or
- multi-label BCE classifier.

This is still compatible, but it is less "exact current architecture" than ISIC 2024 or SkinDisNet.

Good use:

```text
Cross-specialty validation if time permits.
```

---

## 6. Very Recent but Lower Priority: MCR-SL 2025

MCR-SL is highly relevant and very recent:

- Zenodo dataset published 2025-10-10.
- Article in Data, 2025.
- 779 clinical images.
- 1,352 dermoscopic images.
- 240 unique lesions.
- 60 subjects.
- Rich clinical context.
- Nine lesion types.

Sources:
- Zenodo dataset: https://zenodo.org/records/17306338
- Article page: https://www.mdpi.com/2306-5729/10/10/166

Why it is lower priority:

- It is very small.
- Only 240 lesions and 60 subjects.
- It is better for a qualitative/small-cohort appendix than for the main external proof.

---

## 7. Updated Recommendation

If the instructor specifically asked for "latest data where work has been done already", PAD-UFES-20 is not enough.

Use this order:

1. **ISIC 2024 / SLICE-3D**: main latest validation. This is the best answer.
2. **PAD-UFES-20**: clean external validation, but older.
3. **SkinDisNet 2025**: very recent non-cancer dermatology appendix.
4. **BRSET 2024**: cross-specialty ophthalmology validation if time allows.

Minimal paper strategy:

```text
Primary dataset:
  ISIC-2019, current v9 results.

Latest dataset:
  ISIC 2024 / SLICE-3D, binary skin-cancer VFL with image + core metadata.

Clean external dataset:
  PAD-UFES-20, smartphone clinical skin-lesion VFL.
```

This gives the paper a strong structure:

```text
ISIC-2019: original dermoscopy + metadata benchmark
ISIC-2024/SLICE-3D: latest large-scale 3D-TBP skin-cancer benchmark with active 2025 work
PAD-UFES-20: independent smartphone clinical skin-lesion validation
```

The novelty statement becomes much stronger:

> Recent ISIC 2024 studies show that image-plus-metadata fusion improves 3D-TBP skin-cancer triage, but they assume centralized access to image-derived representations and metadata. We study the vertical version of this setting, where the image holder sends only a compressed representation to the metadata-and-label holder, and we evaluate reconstruction privacy of that representation.

