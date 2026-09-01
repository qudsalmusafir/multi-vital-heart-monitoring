# Multi-Vital Tachycardia Detection (VitalDB)

Exploring multi-vital heart monitoring on the VitalDB surgical dataset, as part of ongoing research into heart monitoring methods that don't rely solely on ECG.

## Dataset

[VitalDB](https://physionet.org/content/vitaldb/1.0.0/) — a high-fidelity multi-parameter vital signs database from over 6,000 surgical patients, including ECG, arterial blood pressure, PPG, SpO2, and more, at up to 500Hz resolution.

## Goal

Test whether combining ECG with other continuous vital signs improves heart-condition monitoring compared to ECG alone. Tachycardia detection (heart rate > 100 bpm) was used as the target task.

## Approach

1. **Signals used:** raw ECG, arterial blood pressure (invasive, continuous), and PPG — all sampled at full 100Hz waveform resolution (not downsampled to 1 sample/second, which was found to destroy the signals' actual waveform shape in earlier attempts).
2. **Segmentation:** each case is split into 10-second windows; a window is labeled "tachycardia" if the majority of its heart-rate readings exceed 100 bpm.
3. **Model:** a 1D CNN (3 convolution + pooling blocks) trained on the 3-channel (ECG + arterial pressure + PPG) windows, with class weighting to handle the ~93%/7% class imbalance.
4. **Threshold tuning:** the default 0.5 classification threshold is tuned post-hoc to find the best precision/recall/F1 tradeoff.

## Results

| Metric | Default threshold | Tuned threshold |
|---|---|---|
| Accuracy | 90.68% | 95% |
| Tachycardia Precision | 0.49 | **0.70** |
| Tachycardia Recall | **0.90** | 0.78 |
| Tachycardia F1 | 0.64 | **0.74** |

- Using full waveform resolution (rather than 1-sample-per-second downsampling) was the key factor that made this approach work — earlier low-resolution attempts (ECG-only, and ECG + SpO2/blood pressure at 1Hz) all performed poorly (F1 in the 0.13–0.23 range).
- The default threshold favors catching more true tachycardia cases (high recall) at the cost of more false alarms; the tuned threshold trades some recall for meaningfully better precision.

## Limitations

VitalDB is a **surgical/anesthesia population** — heart rate is influenced by anesthesia, ventilation, and surgical stimulation, not purely by the underlying rhythm/condition. Results here may not directly generalize to non-surgical or wearable-device contexts.

## Files

- `vitaldb_tachycardia.ipynb` — the final working pipeline: data loading, windowing, model training, evaluation, and threshold tuning. (Earlier low-resolution experiments and dead ends from the exploration process have been removed for clarity — see the results table above for a summary of what didn't work and why.)
