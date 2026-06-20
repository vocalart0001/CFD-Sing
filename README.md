# CFD-Sing

**A Counterfactual Latent Diffusion Approach to Singing Voice Correction for Vocal Music Teaching**

Official reference implementation, dataset preparation guides, and full reproduction pipeline for every table and figure reported in Section 4 of the manuscript.

---

## Repository contents

This repository hosts two self-contained archives:

| Archive | Size | Purpose |
|---|---|---|
| [`cfdsing_code.zip`](#cfdsing_codezip) | ≈ 5 MB | Model, training, inference, evaluation, and figure-generation code |
| [`cfdsing_datasets.zip`](#cfdsing_datasetszip) | ≈ 30 KB | Dataset reference document, per-corpus preparation guides, format specifications, and the phoneme vocabulary |

Both archives are designed to be downloaded together. Unpack them side-by-side:

```
cfdsing/
├── cfdsing_code/           ← from cfdsing_code.zip
└── cfdsing_datasets/       ← from cfdsing_datasets.zip
```

---

## Quick start

```bash
# 1. Get the archives
unzip cfdsing_code.zip
unzip cfdsing_datasets.zip

# 2. Install Python dependencies
cd cfdsing_code
pip install -r requirements.txt

# 3. Reproduce all tables and figures
bash scripts/run_all.sh
```

The reproduction script aggregates the per-recording experimental records under `results/per_sample/` into the six summary tables (`results/tables/`) and renders the six figures (`results/figures/`), printing per-method summary lines that match the values reported in the paper.

---

## cfdsing_code.zip

```
cfdsing_code/
├── README.md, requirements.txt
├── configs/cfd_sing.yaml              # Hyperparameters (Table 1)
├── cfdsing/
│   ├── models/
│   │   ├── encoder.py                 # Triple-condition encoder (Eq. 3–5)
│   │   ├── unet.py                    # Three-branch cross-attention U-Net (Eq. 6–8)
│   │   ├── diffusion.py               # Gaussian diffusion process (Eq. 2)
│   │   ├── hifigan.py                 # HiFi-GAN codec (Eq. 1)
│   │   └── cfd_sing.py                # End-to-end model (51.8 M parameters)
│   ├── losses/objectives.py           # L_diff (Eq. 9) + L_disent (Eq. 14) + total (Eq. 15)
│   ├── metrics/
│   │   ├── t_star.py                  # Algorithm 1 — binary-search noise-step metric
│   │   ├── mel_diff.py                # Algorithm 2 — phoneme-level attribution
│   │   └── objective.py               # MCD, F0 RMSE, V/UV F1, PMOS
│   ├── data/datasets.py               # Loaders for the five public datasets
│   ├── eval/                          # Six evaluation entry points (one per table)
│   ├── figures/                       # Six figure-generation scripts
│   └── train.py                       # Training loop
├── inference/correct.py               # Apply CFD-Sing to a singing recording
├── scripts/run_all.sh                 # Master reproduction script
└── results/
    ├── per_sample/                    # Per-recording experimental records
    ├── tables/                        # Aggregated summary tables (CSV)
    └── figures/                       # Rendered figures (PNG + PDF)
```

### Module ↔ equation mapping

| Equation / Algorithm | Implementation |
|---|---|
| Eq. (1) waveform → latent | `cfdsing/models/hifigan.py` |
| Eq. (2) forward diffusion | `cfdsing/models/diffusion.py::add_noise` |
| Eq. (3) phoneme condition | `cfdsing/models/encoder.py::PhonemeEncoder` |
| Eq. (4) melody condition | `cfdsing/models/encoder.py::MelodyEncoder` |
| Eq. (5) technique condition | `cfdsing/models/encoder.py::TechniqueEncoder` |
| Eq. (6) single cross-attention | `cfdsing/models/unet.py::CrossAttentionBranch` |
| Eq. (7) three-branch fusion | `cfdsing/models/unet.py::TripleCrossAttention` |
| Eq. (8) denoiser ε<sub>θ</sub> | `cfdsing/models/unet.py::TripleCrossAttnUNet` |
| Eq. (9) diffusion loss | `cfdsing/losses/objectives.py::diffusion_loss` |
| Eq. (10), (11) t* metric | `cfdsing/metrics/t_star.py` |
| Eq. (12), (13) Mel-diff | `cfdsing/metrics/mel_diff.py` |
| Eq. (14) disentanglement | `cfdsing/losses/objectives.py::disentanglement_loss` |
| Eq. (15) total loss | `cfdsing/losses/objectives.py::total_loss` |
| Algorithm 1 | `cfdsing/metrics/t_star.py::binary_search_t_star` |
| Algorithm 2 | `cfdsing/metrics/mel_diff.py::phoneme_level_heatmap` |

---

## cfdsing_datasets.zip

```
cfdsing_datasets/
├── README.md
├── CFD-Sing_Public_Datasets_Download_Reference.docx
├── per_dataset/
│   ├── popbutfy_prep.md
│   ├── opencpop_prep.md
│   ├── vocalset_prep.md
│   ├── nhss_prep.md
│   └── damp_vpb_prep.md
├── format_specs/
│   ├── audio_format.md
│   ├── textgrid_format.md
│   ├── lyric_format.md
│   └── midi_format.md
└── phoneme_vocab.txt
```

The package consolidates references, per-corpus preparation guides, format specifications, and the 110-token phoneme vocabulary used by the encoder. It does **not** ship audio — each corpus must be obtained from its original provider under the terms of its own licence.

---

## Public datasets

All five datasets used in Section 4 of the manuscript are publicly released:

| Dataset | Role in CFD-Sing | Singers | Duration | Access | URL |
|---|---|---|---|---|---|
| **PopBuTFy** | Main training | 32 | ≈ 28.5 h | Apply | https://github.com/MoonInTheRiver/NeuralSVB |
| **OpenCpop** | Main training (Mandarin) | 1 | 5.2 h | Apply | https://wenet-e2e.github.io/opencpop/ |
| **VocalSet** | Zero-shot technique | 20 | 10.1 h | Open | https://zenodo.org/records/1442513 |
| **NHSS** | Zero-shot cross-dataset | 10 | 4.75 h | Open + EULA | https://hltnus.github.io/NHSSDatabase/ |
| **DAMP-VPB** | Zero-shot diversity | 1,200+ | 80+ h | Apply | https://zenodo.org/records/2616690 |

VocalSet and NHSS are directly downloadable; PopBuTFy, OpenCpop, and DAMP-VPB require an application. Detailed access notes, licence summaries, and a per-corpus file-composition matrix live in `CFD-Sing_Public_Datasets_Download_Reference.docx`.

After download, follow the matching guide under `cfdsing_datasets/per_dataset/<name>_prep.md` to lay the files out where the data loaders expect them, then run MFA on the lyric transcripts to produce TextGrids (for PopBuTFy, OpenCpop, and NHSS).

---

## Training and inference

### Train from scratch

```bash
cd cfdsing_code
python -m cfdsing.train \
    --config configs/cfd_sing.yaml \
    --out_dir runs/cfd_sing
```

Training settings follow Table 1 of the paper: AdamW with base learning rate 2 × 10⁻⁴, cosine schedule with 4 k warm-up steps, batch size 16, 5 × 10⁵ iterations, λ<sub>d</sub> = 0.1, T = 1000 diffusion steps with a cosine variance schedule. Checkpoints are written every 10 k steps to `runs/cfd_sing/step_*.pt`, with the final model at `runs/cfd_sing/final.pt`.

### Apply CFD-Sing to a singing recording

```bash
python -m inference.correct \
    --config configs/cfd_sing.yaml \
    --ckpt   runs/cfd_sing/final.pt \
    --in     path/to/student.wav \
    --lyrics path/to/student.TextGrid \
    --score  path/to/target_score.mid \
    --tau    "vibrato=0.4,breathiness=0.1,resonance=0.8" \
    --out    out/
```

Outputs:

- `corrected.wav` — counterfactual singing waveform
- `feedback.npz` — per-phoneme attribution heatmap (Algorithm 2)
- `summary.json` — top-5 attributed phonemes and the t* value (Algorithm 1)

---

## Tables and figures reproduced

| Result | Script | Topic |
|---|---|---|
| Table 3 | `cfdsing.eval.run_objective` | MCD / F0 RMSE / V-UV F1 / PMOS across 5 methods |
| Table 4 | `cfdsing.eval.run_subjective` | MOS-N / SMOS / pairwise preference (50 listeners) |
| Table 5 | `cfdsing.eval.run_t_star` | Correlation of t* with expert ratings on 5 datasets |
| Table 6 | `cfdsing.eval.run_attribution` | Precision / Recall / F1 of phoneme-level attribution |
| Table 7 | `cfdsing.eval.run_ablation` | Ablation of the four design choices |
| §4.7 | `cfdsing.eval.run_cross_dataset` | Cross-dataset MCD on NHSS / VocalSet / DAMP-VPB |
| Figure 6 | `make_fig6_radar` | Normalised radar across MCD / F0 / V-UV F1 / PMOS |
| Figure 7 | `make_fig7_preference` | Stacked-bar preference distribution |
| Figure 8 | `make_fig8_scatter` | t* vs expert rating (n = 1,716) |
| Figure 9 | `make_fig9_heatmap` | Phoneme-level attribution heatmap |
| Figure 10 | `make_fig10_violin` | PMOS violin plot across 5 ablation variants |
| Figure 11 | `make_fig11_tsne` | 2-D t-SNE of the latent space (N = 1,500) |

Each figure is written as both PNG (180 dpi) and PDF for inclusion in publications.

---

## Environment

- Python ≥ 3.10
- PyTorch ≥ 2.0
- NumPy, SciPy, scikit-learn, matplotlib, PyYAML, soundfile, mido

A CUDA-enabled GPU with ≥ 24 GB VRAM is required for training. Evaluation and figure generation run comfortably on CPU. See `cfdsing_code/requirements.txt` for the full dependency list.

External tooling required for training only:

- **MFA** (Montreal Forced Aligner) — https://montreal-forced-aligner.readthedocs.io/
- **HiFi-GAN-Singing** pretrained vocoder — released with NeuralSVB at https://github.com/MoonInTheRiver/NeuralSVB

---

## Citation

If you use this code or the materials in this repository, please cite the manuscript:

```bibtex
@article{cfdsing,
  title  = {A Counterfactual Latent Diffusion Approach to Singing Voice
            Correction for Vocal Music Teaching},
  author = {...},
  year   = {...},
  journal= {...},
}
```

When you also use any of the five public datasets, please cite their original releases:

- **PopBuTFy** — Liu et al., *Learning the Beauty in Songs: Neural Singing Voice Beautifier*, ACL 2022.
- **OpenCpop** — Wang et al., *Opencpop: A High-Quality Open Source Chinese Popular Song Corpus for Singing Voice Synthesis*, INTERSPEECH 2022.
- **VocalSet** — Wilkins et al., *VocalSet: A Singing Voice Dataset*, ISMIR 2018.
- **NHSS** — Sharma et al., *NHSS: A Speech and Singing Parallel Database*, *Speech Communication*, 2021.
- **DAMP-VPB** — Smule, Inc., *Digital Archive of Mobile Performances — Vocal Performances Balanced*, Zenodo, 2019.

---

## Licence

Code: MIT (see `cfdsing_code/LICENSE` once added).
Datasets: each public corpus retains its own licence — see the matching file in `cfdsing_datasets/per_dataset/`.

---

## Contact

Issues and pull requests are welcome through the repository's issue tracker.
