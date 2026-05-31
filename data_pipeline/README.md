# G1 Motion Dataset Pipeline

End-to-end scripts for retargeting AMASS / HumanML3D motions to the **Unitree G1** humanoid robot and publishing the resulting dataset.

## Overview

```
AMASS raw data
    │
    ├── build_curated_yaml.py       # Filter AMASS to HumanML3D clips
    ├── build_pt_parts.py           # Convert AMASS → .pt motion library chunks
    │
    └── [Vast.ai GPU instance]
            │
            ├── vastai_retarget_pipeline.py     # Full pipeline orchestration
            ├── setup_vastai_envs.sh            # Environment setup
            └── pyroki/batch_retarget_to_g1_from_keypoints.py  # Core retargeter
                    │
                    └── retargeted_g1/*.npz     (one .npz per source sequence)

retargeted_g1/*.npz
    │
    ├── retry_hf_push.py            # Push .npz files to HuggingFace
    ├── create_professional_dataset.py   # Publish clean research-ready HF repo
    │
    ├── build_text_sidecar.py       # Build text_annotations.json from HumanML3D
    ├── build_dataset_excel.py      # Join retargeted files + text → Excel/CSV
    │
    ├── translate_captions.py       # Translate captions to Hindi/Bengali/Tamil/Telugu
    ├── split_csv.py                # Split CSV for parallel translation workers
    └── merge_chunks.py             # Merge parallel translation outputs
```

## Prerequisites

```bash
pip install huggingface_hub deep-translator pandas openpyxl
export HF_TOKEN=hf_...   # HuggingFace write token (required for all HF uploads)
```

## Quick Start

### 1. Build the AMASS motion library

```bash
# Filter AMASS to HumanML3D-curated sequences
python data_pipeline/build_curated_yaml.py

# Build .pt motion library parts
python data_pipeline/build_pt_parts.py
```

### 2. Retarget on GPU (Vast.ai recommended)

```bash
# Full pipeline (keypoint extraction + retargeting)
python data_pipeline/vastai_retarget_pipeline.py

# Or retarget a single dataset directly
python pyroki/batch_retarget_to_g1_from_keypoints.py \
    --keypoints-folder-path /workspace/keypoints/CMU \
    --output-dir /workspace/retargeted_g1/CMU \
    --no-visualize --skip-existing --subsample-factor 2 \
    --source-type smpl --input-fps 30
```

### 3. Push to HuggingFace

```bash
export HF_TOKEN=hf_...
python data_pipeline/retry_hf_push.py --all
```

### 4. Build annotated dataset

```bash
# Create text annotation sidecar from HumanML3D
python data_pipeline/build_text_sidecar.py

# Join retargeted files + captions → Excel + CSV
python data_pipeline/build_dataset_excel.py

# Translate captions (parallel workers)
python data_pipeline/split_csv.py
# Run in 4 terminals:
python data_pipeline/translate_captions.py --csv g1_chunk_0.csv
python data_pipeline/translate_captions.py --csv g1_chunk_1.csv
python data_pipeline/translate_captions.py --csv g1_chunk_2.csv
python data_pipeline/translate_captions.py --csv g1_chunk_3.csv
python data_pipeline/merge_chunks.py
```

## Documentation

- [`HUMANML3D_TO_G1_PIPELINE.md`](HUMANML3D_TO_G1_PIPELINE.md) — Full pipeline walkthrough
- [`VASTAI_GPU_GUIDE.md`](VASTAI_GPU_GUIDE.md) — Vast.ai GPU setup and tips
- [`PROTOMOTIONS_CODEBASE_REFERENCE.md`](PROTOMOTIONS_CODEBASE_REFERENCE.md) — ProtoMotions internals reference

## Dataset Stats (AMASS → G1, HumanML3D subset)

| Source | Sequences |
|--------|----------:|
| KIT | 4,231 |
| BioMotionLab_NTroje | 2,958 |
| CMU | 2,082 |
| BMLmovi | 1,801 |
| Eyes_Japan_Dataset | 750 |
| BMLhandball | 649 |
| EKUT | 348 |
| ACCAD | 252 |
| MPI_HDM05 | 215 |
| Transitions_mocap | 110 |
| MPI_mosh | 77 |
| DFaust_67 | 129 |
| SFU | 44 |
| TotalCapture | 37 |
| SSM_synced | 30 |
| HumanEva | 28 |
| **Total** | **13,741** |
