### Official Implementation of Paper: Detection of Spatially Aberrant Cells in Spatial Transcriptomics Data by Conformal Prediction

Pipeline：

![Pipeline](pipeline.png)


Here is the complete `README.md` content inside a single Markdown code block. You can click the **Copy** button in the top-right corner of the block (in most interfaces) or simply select all the text inside and paste it into your `README.md` file.

```markdown
# PDAC Spatial Transcriptomics Integration & Aberrant Cell Detection

This repository provides a complete pipeline for integrating single‑cell RNA‑seq (scRNA‑seq) and spatial transcriptomics (ST) data, predicting cell locations, and identifying aberrant spots using **conformal prediction**. It is based on the framework from Zhao et al. (2022) *Nature Computational Science* and extended with a novel uncertainty‑aware outlier detection module.

## Table of Contents
- [Overview](#overview)
- [Method](#method)
- [Installation](#installation)
- [Usage](#usage)
- [Input Data](#input-data)
- [Output Structure](#output-structure)
- [Configuration](#configuration)
- [Citation](#citation)
- [License](#license)

---

## Overview

- **Integration**: Adversarial domain translation aligns scRNA‑seq and ST data into a shared latent space.
- **Localization**: A separate encoder predicts spatial coordinates for every cell/spot.
- **Aberrant Detection**: Conformal prediction provides per‑spot confidence intervals; spots with prediction errors exceeding the interval are flagged as "aberrant" – potentially indicating technical artefacts or biologically distinct regions.

The implementation is fully modular, supports 3‑fold cross‑validation, and saves all intermediate outputs to timestamped directories to avoid overwriting.

---

## Method

1. **Preprocessing**  
   - Highly variable gene selection (Seurat v3) on both datasets.
   - Overlap of HVGs is used for dimensionality reduction (PCA).
   - Spatial coordinates are normalised to [0,1].

2. **Integration Network** (GAN‑based)  
   - Encoders (`E_A`, `E_B`) map each modality to a shared latent space.
   - Generators (`G_A`, `G_B`) reconstruct the original space from latent codes.
   - Discriminators (`D_A`, `D_B`) enforce distribution alignment.
   - Losses include reconstruction, cycle‑consistency, sliced Wasserstein distance, and a GMM‑based latent regularisation.

3. **Localization Network**  
   - An encoder (`E_s`) takes the concatenation of latent code and SVG (spatially variable gene) expression to predict 2D coordinates.
   - Trained with L1 loss and SWD between predicted and true coordinates.

4. **Conformal Prediction for Aberrant Detection**  
   - For each fold, the ST data is split into training (2/3), calibration (1/6) and test (1/6).
   - Non‑conformity score is based on the ratio of prediction error to local latent‑space density (k‑nearest neighbour average).
   - Quantile calibration yields per‑spot confidence intervals; spots outside the interval are marked aberrant.

5. **Cross‑Validation**  
   - 3‑fold split ensures every spot is evaluated exactly once.
   - Final predictions are aggregated from the three folds.

---

## Installation

### Requirements
- Python 3.8+
- PyTorch 1.10+
- ScanPy 1.9+
- scikit‑learn
- POT (Python Optimal Transport)
- NumPy, Pandas, SciPy
- tqdm

You can install all dependencies with:

```bash
pip install torch scanpy scikit-learn pot numpy pandas scipy tqdm
```

### Clone the repository

```bash
git clone https://github.com/yourusername/pdac-integration.git
cd pdac-integration
```

---

## Usage

### 1. Prepare your data

- **scRNA‑seq**: AnnData object with `X` (counts) and `obs` containing cell type annotations (optional).
- **ST**: AnnData object with `X` (counts), `obs` containing pixel coordinates (`x`, `y`), and `obsm['spatial']` with the original spatial coordinates.

### 2. Modify file paths

In the `main()` function of the script, update the following paths:

```python
adata_ST = sc.read_h5ad('path/to/your_ST.h5ad')
adata_sc = sc.read('path/to/your_sc.h5ad')
svg_list = pd.read_csv('path/to/svg_list.csv', index_col=0).index
```

### 3. Run the pipeline

```bash
python run_pdac_analysis.py
```

All outputs will be written to a new folder: `result/SCC_experiment_YYYYMMDD_HHMMSS/`

### 4. Output files

- `fold_*/` – per‑fold models, embeddings, and evaluation results.
- `adata_ST_final.h5ad` – ST AnnData with added columns: `final_aberrant`, `final_confidence`, `final_lambda`.
- `final_predict.npy` – final predicted coordinates for all spots.
- `all_results.npz` – all intermediate tensors for further analysis.
- `spot_info_PDAC.txt` – tab‑separated file with pixel coordinates, prediction error, aberrant flag, and non‑conformity score (mapped back to original ST layout).

---

## Input Data Format

| File | Description |
|------|-------------|
| `adata_sc.h5ad` | scRNA‑seq data (genes × cells) |
| `adata_ST.h5ad` | ST data (genes × spots) with `obs[['x','y']]` and `obsm['spatial']` |
| `svg_list.csv` | CSV with a single column (index) naming spatially variable genes |

**Important**: The ST `obsm['spatial']` is used for training coordinates. The script normalises it internally.

---

## Configuration

You can adjust model hyperparameters inside the `Model3` constructor:

| Parameter | Description | Default |
|-----------|-------------|---------|
| `batch_size` | Training batch size | 200 |
| `train_epoch` | Number of training epochs | 3000 |
| `n_latent` | Dimension of shared latent space | 20 |
| `lambdacos` | Weight for cosine similarity loss | 10 |
| `lambdaSWD` | Weight for sliced Wasserstein loss | 5 |
| `lambdalat` | Weight for coordinate prediction loss | 10 |
| `alpha` | Significance level for conformal prediction | 0.05 |
| `k_neighbors` | Number of neighbours for λ calculation | 15 |

---

## Citation

If you use this code in your research, please cite the original work:

> Zhao, J. et al. (2022) Adversarial domain translation networks for integrating large-scale atlas-level single-cell datasets. *Nature Computational Science* 2(5):317-330.

And this repository (if applicable):

```bibtex
@misc{pdac-integration,
  author = {Your Name},
  title = {PDAC Integration and Aberrant Detection},
  year = {2026},
  publisher = {GitHub},
  journal = {GitHub Repository},
  howpublished = {\url{https://github.com/yourusername/pdac-integration}}
}
```

---

## License

This project is licensed under the MIT License – see the [LICENSE](LICENSE) file for details.

---

## Contact

For questions or issues, please open a GitHub issue or contact [your.email@example.com](mailto:your.email@example.com).
```
