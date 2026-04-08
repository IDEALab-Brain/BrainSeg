# BrainSeg Environment Image (`brainseg:env`)

This document describes the developer-focused environment image:

- Docker Hub tag: `brainseg/brainseg:env`

Unlike `brainseg:v1`, this image is intended for users who want a ready-to-use runtime environment and default source code for custom development, retraining, and debugging.

## 1. What `brainseg:env` is for

Use `brainseg:env` when you want to:

- avoid manual dependency setup
- modify BrainSeg code
- run custom training workflows
- run manual preprocessing/inference scripts from source

`brainseg:env` is an environment convenience image. It does not rely on one-click entrypoint wrappers.

## 2. What is included

- Preconfigured Python/runtime environment
- Default BrainSeg codebase under `/opt/brainseg`
- Bundled model and runtime resources
- Main packages available (for direct import):
  - `torch`
  - `monai`
  - `SimpleITK`
  - `ants`
  - `openpyxl`
  - and other dependencies from the packaged env

## 3. Runtime defaults inside container

`brainseg:env` container defaults:

- `Entrypoint`: `/bin/bash`
- `Cmd`: `-l`
- `Working directory`: `/opt/brainseg`

That means running the image opens a shell directly.

## 4. Pull and start

Pull from Docker Hub:

```powershell
docker pull brainseg/brainseg:env
```

Run with your data mounted:

```powershell
docker run --rm -it --gpus all `
  -v D:\YourData:/workspace/data `
  brainseg/brainseg:env
```

After entering the container:

```bash
pwd
python -V
python -c "import torch; print(torch.cuda.is_available())"
```

## 5. Important paths in the container

- Source code: `/opt/brainseg`
- Environment binaries are already on `PATH`
- Suggested data mount: `/workspace/data`

Because `PATH` is preconfigured, no manual environment activation is required.

## 6. Typical commands in `brainseg:env`

From inside the container:

```bash
cd /opt/brainseg
python train.py --help
python brainseg_cli.py --help
python brainseg_cli.py preprocess --help
python brainseg_cli.py tissue --help
python brainseg_cli.py parc --help
python brainseg_cli.py lesion --help
```

Run preprocessing:

```bash
python brainseg_cli.py preprocess --input /workspace/data
```

Run segmentation tasks:

```bash
python brainseg_cli.py tissue --input /workspace/data
python brainseg_cli.py parc --input /workspace/data
python brainseg_cli.py lesion --input /workspace/data
```

## 7. Develop on your own codebase

If you want code edits to persist on host, mount your own repository:

```powershell
docker run --rm -it --gpus all `
  -v D:\MyBrainSeg:/workspace/BrainSeg `
  -v D:\YourData:/workspace/data `
  -w /workspace/BrainSeg `
  brainseg/brainseg:env
```

Inside container:

```bash
python train.py
python brainseg_cli.py preprocess --input /workspace/data
```

This workflow is recommended for custom training and long-term development.

## 8. Recommended data structure

```text
YourData/
  sub001/
    brain.nii.gz
    T2-brain.nii.gz
    Flair-brain.nii.gz
    tissue.nii.gz
    dk-struct.nii.gz
    lesion.nii.gz
  sub002/
    brain.nii.gz
```

The same modality naming rules used by `brainseg:v1` apply here.

## 9. Output behavior

When using `python brainseg_cli.py ...`, outputs are written under:

```text
/workspace/data/work/
```

Typical output folders:

- `/workspace/data/work/preprocessed`
- `/workspace/data/work/metadata`
- `/workspace/data/work/tissue`
- `/workspace/data/work/parc`
- `/workspace/data/work/lesion`

## 10. `brainseg:v1` vs `brainseg:env`

Use `brainseg:v1` if you need:

- direct end-user commands (`brainseg preprocess`, `tissue`, `parc`, `lesion`, `ui`)
- one-click workflow for deployment/use

Use `brainseg:env` if you need:

- editable development workspace
- manual script-level control
- retraining and experimentation

## 11. Quick validation checklist

From host:

```powershell
docker run --rm --entrypoint bash brainseg/brainseg:env -lc "python -c 'import torch, monai, SimpleITK; print(\"ok\")'"
```

If output prints `ok`, core runtime dependencies are available.

## 12. Notes

- GPU access requires `--gpus all`
- If you only need inference delivery, use `brainseg/brainseg:v1`
- `brainseg:env` is best treated as a stable research/development base image
