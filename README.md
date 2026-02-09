# PyBARseq

A Python pipeline for BARseq spatial transcriptomics data processing, inspired by the BARseq2 analysis workflow (Chen et al., 2024) with updated registration, segmentation, and quality control methods.

## Repository Structure

```
├── barseq_v2_main_demo.ipynb # Main pipeline (all processing functions)
├── n2v_runner.py             # Noise2Void denoising (standalone, runs in n2v conda env)
├── cellpose_v3_runner.py     # Cellpose v3 segmentation (standalone, runs in cellpose env)
├── cfg.yaml                  # Multi-environment task configuration
├── export_envs.py            # Conda environment export utility
└── README.md
```
## Installation

### 1. Create conda environments

pyBARseq requires three separate conda environments due to dependency conflicts:

**Main environment** (`barseq`):
```bash
conda create -n barseq python=3.10
conda activate barseq
pip install numpy scipy scikit-image pandas matplotlib tifffile joblib natsort opencv-python-headless bardensr pyimagej
```

**Denoising** (`n2v_tf24_gpu`):
```bash
conda create -n n2v_tf24_gpu python=3.8
conda activate n2v_tf24_gpu
pip install tensorflow==2.4 n2v csbdeep
```

**Segmentation** (`cellpose_v3`):
```bash
conda create -n cellpose_v3 python=3.10
conda activate cellpose_v3
pip install cellpose torch
```

### 2. Configure paths

Edit `cfg.yaml` to point to your runner script locations:

```yaml
tasks:
  cellpose_segmentation:
    env: "conda:cellpose_v3"
    argv:
      - /path/to/cellpose_v3_runner.py
      - "{pth}"
  denoising_n2v:
    env: "conda:n2v_tf24_gpu"
    argv:
      - /path/to/n2v_runner.py
      - "{pth}"
      - "{fname}"
```

## Usage

The pipeline is organized into four sequential entry-point functions in `barseq_v2_main_demo.ipynb`. Run them in order:

```python
# 1. Preprocess all modalities
preprocessing(pth, config_pth, basedir, config_file='cfg.yaml')

# 2. Stitch tiles and segment cells
stitch_and_segment(pth, config_pth, basedir, config_file='cfg.yaml')

# 3. Basecall genes, hyb probes, and barcodes
basecalls(pth, config_pth)

# 4. Aggregate data and filter overlapping cells
data_aggregation_and_filtering(pth, config_pth)
```

### Input Data Structure

The pipeline expects raw BARseq imaging data organized by sequencing cycles, with separate folders for geneseq, bcseq, and hyb modalities. Each cycle folder contains multi-channel TIFF images for all tile position.

### Outputs

- `filt_neurons.joblib` — Final filtered cell data (expression matrix, coordinates, cell IDs, FOV assignments)
- `processed/` — Intermediate results per tile (aligned images, segmentation masks, basecall results)
- `check_registration/` — QC visualizations for registration quality

## Quality Control

pyBARseq includes built-in QC visualization functions:

- `check_geneseq_cycle_alignment()` — Red-green overlays of cycle-to-cycle registration
- `check_bcseq_cycle_alignment()` — Same for barcode cycles
- `check_hyb_basecall()` — Scatter plot of hybridization calls colored by gene identity
- `check_stitched_output()` — RGB composites of stitched whole-section images
- `check_cellpose_output()` — Input image vs. segmentation mask comparison
- `check_final_overlap_removal()` — Before/after spatial maps
- `check_gene_expression()` — Single-gene spatial expression maps
- `check_cycle_registration()` — NMI-based registration quality across all tiles

## Features Not Yet Ported

The following MATLAB features are under development:

- Cortical depth estimation
- PSF deconvolution in hybridization basecalling
- Nuclear background profile subtraction for soma barcoding
- Barcode error correction (collapsing, mismatch-tolerant matching)
- Soma barcode quality filtering (complexity/score/signal thresholds)

## References

- Chen, Xiaoyin, et al. "Whole-cortex in situ sequencing reveals input-dependent area identity." Nature (2024): 1-10.
- Stringer, Carsen, et al. "Cellpose: a generalist algorithm for cellular segmentation." Nature methods 18.1 (2021): 100-106.
- Krull, Alexander, Tim-Oliver Buchholz, and Florian Jug. "Noise2void-learning denoising from single noisy images." Proceedings of the IEEE/CVF conference on computer vision and pattern recognition. 2019.
- Chen, Shuonan, et al. "Barcode demixing through non-negative spatial regression (bardensr)." PLoS computational biology 17.3 (2021): e1008256.
- Chalfoun, Joe, et al. "MIST: accurate and scalable microscopy image stitching tool with stage modeling and error minimization." Scientific reports 7.1 (2017): 4988.
- Muhlich, Jeremy L., et al. "Stitching and registering highly multiplexed whole-slide images of tissues and tumors using ASHLAR." Bioinformatics 38.19 (2022): 4613-4621.



