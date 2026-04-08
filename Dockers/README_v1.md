# BrainSeg Docker v1

This repository provides a production-ready `brainseg:v1` Docker runtime for BrainSeg preprocessing, inference, and metadata generation.

The v1 image is intended for end users who want a one-command workflow on their own data without manual environment setup.

## 1. What this release includes

- Fully packaged runtime environment
- Fully packaged offline NLP dependencies for BiomedCLIP text encoder
- Bundled checkpoints:
  - `BCLIP.pt`
  - `BrainSeg_tissue.pt`
  - `BrainSeg_parc.pt`
  - `BrainSeg_lesion.pt`
- Bundled template:
  - `MNI152_T1_1mm_Brain.nii.gz`
- Built-in tasks:
  - `preprocess`
  - `metadata-template`
  - `metadata`
  - `tissue`
  - `parc` (`dk` alias)
  - `lesion`

All outputs are written under `<input>/work/`.  
No internet is required at inference time.

## 2. Image names

- Local tag in this repository: `brainseg:v1`
- Docker Hub tag (recommended for users): `brainseg/brainseg:v1`

Pull from Docker Hub:

```powershell
docker pull brainseg/brainseg:v1
```

## 3. System requirements

- Docker Desktop 4.x or Docker Engine (Linux containers)
- NVIDIA GPU and compatible driver (recommended)
- NVIDIA Container Toolkit / Docker GPU support (`--gpus all`)

CPU-only mode is also supported by removing `--gpus all` and passing `--device cpu`.

## 4. Input data requirements

### 4.1 Core assumption

Skull stripping must be done by the user before running BrainSeg.

### 4.2 Supported modality filenames

- `brain.nii.gz`
- `T2-brain.nii.gz`
- `CT-brain.nii.gz`
- `Flair-brain.nii.gz`
- `DWI-brain.nii.gz`
- `US-brain.nii.gz`
- `PD-brain.nii.gz`
- `SWI-brain.nii.gz`
- `T2s-brain.nii.gz`
- `AV45-brain.nii.gz`
- `FDG-brain.nii.gz`
- `TAU-brain.nii.gz`
- `Dynamic-brain.nii.gz`
- `PIB-brain.nii.gz`
- `CTAC-brain.nii.gz`
- `Flumetamol-brain.nii.gz`
- `NAV4694-brain.nii.gz`
- `SUV-brain.nii.gz`
- `SUM-brain.nii.gz`

### 4.3 Optional ground truth labels

- `tissue.nii.gz`
- `dk-struct.nii.gz`
- `lesion.nii.gz`

These are optional. If present, Dice metrics are computed.

### 4.4 Recommended input structure

```text
YourData/
  sub001/
    brain.nii.gz
    T2-brain.nii.gz
    Flair-brain.nii.gz
    AV45-brain.nii.gz
    tissue.nii.gz
    dk-struct.nii.gz
    lesion.nii.gz
  sub002/
    brain.nii.gz
    DWI-brain.nii.gz
```

## 5. Output structure

BrainSeg writes all runtime outputs to `<input>/work/`:

```text
YourData/
  work/
    preprocess_steps/
      01_n4/
      02_registered/
    preprocessed/
    metadata/
      brainseg_metadata_template.xlsx
      brainseg_auto_tissue.xlsx
      brainseg_auto_parc.xlsx
      brainseg_auto_lesion.xlsx
    tissue/
    parc/
    lesion/
```

## 6. Preprocessing behavior in v1

Pipeline order:

1. N4 bias correction
2. Rigid registration to `MNI152_T1_1mm_Brain.nii.gz`
3. Background cleanup trick during registration:
   - nearest-neighbor interpolation -> foreground mask
   - linear interpolation -> registered image
   - mask application -> remove interpolation halo outside the brain
4. Finalize outputs to `work/preprocessed`

Important notes:

- The bundled MNI template is already `224 x 256 x 224`
- The template has copied physical metadata from `Sample/sub001/brain.nii.gz`
- No extra crop/pad is needed after registration
- `--step crop` is preserved only as a compatibility alias and now only finalizes outputs

## 7. Command reference and quick start

### 7.1 Help

```powershell
docker run --rm brainseg/brainseg:v1 --help
docker run --rm brainseg/brainseg:v1 preprocess --help
docker run --rm brainseg/brainseg:v1 metadata-template --help
docker run --rm brainseg/brainseg:v1 metadata --help
docker run --rm brainseg/brainseg:v1 tissue --help
docker run --rm brainseg/brainseg:v1 parc --help
docker run --rm brainseg/brainseg:v1 lesion --help
```

### 7.2 Full preprocessing

```powershell
docker run --rm --gpus all `
  -v D:\YourData:/workspace/data `
  brainseg/brainseg:v1 preprocess `
  --input /workspace/data
```

### 7.3 Registration only (skip N4)

```powershell
docker run --rm --gpus all `
  -v D:\YourData:/workspace/data `
  brainseg/brainseg:v1 preprocess `
  --input /workspace/data `
  --step register
```

### 7.4 Custom registration template

```powershell
docker run --rm --gpus all `
  -v D:\YourData:/workspace/data `
  -v D:\Templates:/workspace/templates `
  brainseg/brainseg:v1 preprocess `
  --input /workspace/data `
  --step register `
  --template /workspace/templates/my_template.nii.gz
```

### 7.5 Metadata template generation

```powershell
docker run --rm `
  -v D:\YourData:/workspace/data `
  brainseg/brainseg:v1 metadata-template `
  --input /workspace/data
```

Default output:

`/workspace/data/work/metadata/brainseg_metadata_template.xlsx`

Editable columns include:

- `age`
- `age_stage`
- `gender`
- `diagnosis`
- `preferred_pet`
- `caption_override`

### 7.6 Generate metadata workbook

```powershell
docker run --rm `
  -v D:\YourData:/workspace/data `
  brainseg/brainseg:v1 metadata `
  --input /workspace/data `
  --task tissue
```

### 7.7 Tissue segmentation

```powershell
docker run --rm --gpus all `
  -v D:\YourData:/workspace/data `
  brainseg/brainseg:v1 tissue `
  --input /workspace/data
```

### 7.8 Parcellation

```powershell
docker run --rm --gpus all `
  -v D:\YourData:/workspace/data `
  brainseg/brainseg:v1 parc `
  --input /workspace/data
```

If tissue prior is missing, `parc` auto-runs tissue first.

### 7.9 Lesion task

```powershell
docker run --rm --gpus all `
  -v D:\YourData:/workspace/data `
  brainseg/brainseg:v1 lesion `
  --input /workspace/data
```


## 8. How to test Vis_Sample data (paper visualization raw data)

This repository includes the raw visualization samples used for paper figures:

- Subjects: `Vis_Sample/test/`
- Captions workbook: `Vis_Sample/test.xlsx`
- One-click script: `Vis_Sample.sh`

Design of `Vis_Sample.sh`:

- No preprocessing is performed
- Raw sample data is used directly
- Samples are grouped by name prefix:
  - `fig3_*` -> tissue only
  - `fig5_*` -> tissue then parc
  - `fig6_*` -> lesion only
  - `s.fig4_*` -> tissue only

### 8.1 Run Vis_Sample using Docker v1 image

```powershell
docker run --rm --gpus all `
  -v D:\BrainSeg:/workspace `
  -w /workspace `
  --entrypoint bash `
  brainseg/brainseg:v1 -lc "bash ./Vis_Sample.sh"
```

`--entrypoint bash` is required because `brainseg:v1` defaults to BrainSeg CLI entrypoint.

### 8.2 Default Vis_Sample outputs

```text
Vis_Sample/runs/
  fig3_tissue/
  fig5_tissue_parc/
  fig6_lesion/
  sfig4_tissue/
```

Example prediction files:

- `Vis_Sample/runs/fig3_tissue/input/work/tissue/<subject>/pred_tissue.nii.gz`
- `Vis_Sample/runs/fig5_tissue_parc/input/work/parc/<subject>/pred_dk-struct.nii.gz`
- `Vis_Sample/runs/fig6_lesion/input/work/lesion/<subject>/pred_lesion.nii.gz`

### 8.3 Customize Vis_Sample output root

```powershell
docker run --rm --gpus all `
  -e VIS_SAMPLE_RUN_ROOT=/workspace/Vis_Sample/runs_custom `
  -v D:\BrainSeg:/workspace `
  -w /workspace `
  --entrypoint bash `
  brainseg/brainseg:v1 -lc "bash ./Vis_Sample.sh"
```


## 9. Additional notes

- One-time `transformers` cache migration logs may appear on first run
- Some PyTorch/cudnn warnings may appear and do not block inference
