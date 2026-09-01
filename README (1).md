# Irregular Heartbeat Detection from PPG Alone (No ECG)

Exploring multi-vital, non-ECG heart monitoring for detecting irregular heartbeats (arrhythmia), as part of ongoing research into heart monitoring methods that don't rely solely on ECG.

## Dataset

[MIMIC PERform AF Dataset](https://zenodo.org/records/15906524) (Charlton et al.) — 35 critically-ill adult patients, clinically diagnosed as either **Atrial Fibrillation (AF, 19 patients)** or **non-AF / normal sinus rhythm (16 patients)**. Recordings include synchronized ECG, PPG, and respiration (impedance pneumography), extracted from the MIMIC-III Waveform Database.

## Goal

Test whether irregular heart rhythm (AF) can be reliably detected from **PPG alone** — without using ECG as a model input — since this is the realistic scenario for wearable devices (e.g. smartwatches) that only have a PPG sensor, not an ECG lead.

**Note on ECG usage:** ECG is present in this dataset, but is *not* used as a model input anywhere in this project. It exists in the raw files only because it was part of the original clinical recording; all features and models here are built entirely from the PPG (and, in one experiment, respiration) signal.

## Approach

1. **Feature engineering** — rather than feeding raw PPG waveform into a model, standard HRV (heart rate variability) style features are extracted from pulse-to-pulse timing: RMSSD, pNN50, Poincaré plot geometry (SD1/SD2), and Shannon entropy — the same category of features used in established clinical and wearable-device AF-detection research.
2. **Segmentation** — each patient's recording is split into 30-second windows, with features extracted per window.
3. **Model** — a Random Forest classifier trained on these engineered features.
4. **Validation** — patient-level 5-fold cross-validation (not a single train/test split), to get a reliable estimate given the small dataset size.
5. **Extension** — a second experiment adds respiration as an additional input alongside PPG, to test whether a second vital sign improves detection.

## Results

| Approach | Mean F1 (5-fold CV) | Mean Accuracy (5-fold CV) |
|---|---|---|
| PPG only | **0.972 ± 0.024** | **0.984 ± 0.008** |
| PPG + Respiration | 0.964 – 0.968 | 0.978 – 0.980 |

- **PPG alone achieves strong, consistent detection of AF** — comparable to results reported in published research using this same dataset.
- **Adding respiration did not improve results.** This makes physiological sense: AF is a cardiac rhythm disorder, and PPG alone already captures the relevant cardiac timing signal directly. This suggests multi-vital inputs help only when the added signal is physiologically linked to the specific condition being detected, not simply by adding more data.
- Sanity checks confirmed no patient overlap between train/test splits, and that the underlying feature (RMSSD ratio) shows a large, clean separation between AF (mean 0.25) and non-AF (mean 0.06) segments — supporting that the strong results are genuine, not a data leakage artifact.

## Limitations

This dataset is small (35 patients) and drawn from a single ICU population. These results are a **strong proof-of-concept**, not yet evidence that this generalizes to a larger, more diverse, or outpatient/wearable population. Validating on a larger, more diverse dataset is a natural next step.

## Files

- `mimic_perform_af.ipynb` — full notebook: data loading, feature extraction, model training, cross-validation, and both experiments (PPG-only and PPG+Respiration)
