# Machine Learning-Based Prediction of Molecular Lipophilicity (logD₇.₄): An Evaluation of Predictive Performance Across the 500 Da Molecular Weight Threshold

CHE1147 Final Project — University of Toronto

Authors: Chengrui Liu and Xiaochen (Helen) Liu

## Project Overview

This project predicts experimental molecular lipophilicity, expressed as
logD at pH 7.4, using molecular descriptors calculated from SMILES.

The project addresses two research questions:

1. Which candidate model provides the best predictive performance?
2. Does prediction error differ between compounds with molecular weight
   at or below 500 Da and compounds above 500 Da?

Error trends near 500 Da are also explored. A difference between the two
groups does not, by itself, establish a sharp performance boundary at
500 Da.

## Dataset

The project uses the DeepChem Lipophilicity dataset, containing 4,200
compound records with ChEMBL identifiers, SMILES strings, and experimental
logD values at pH 7.4.

Dataset URL:
https://deepchemdata.s3-us-west-1.amazonaws.com/datasets/Lipophilicity.csv

Required columns:

- `CMPD_CHEMBLID`: compound identifier
- `smiles`: molecular structure representation
- `exp`: experimental logD at pH 7.4

The notebook checks missing values, invalid SMILES, and duplicate
canonical molecular structures before modeling. The number of retained
records is reported during execution.

## Molecular Descriptors

Descriptors are calculated using RDKit:

| Descriptor | Unit |
|---|---|
| Molecular weight | Da |
| Topological polar surface area | Å² |
| Hydrogen bond donor count | Count |
| Aromatic ring count | Count |
| Fraction of sp³ carbons | Dimensionless |
| Rotatable bond count | Count |
| Calculated MolLogP | Dimensionless |

Each model family is evaluated with and without calculated MolLogP.

Calculated MolLogP is derived from molecular structure. It is related
to lipophilicity but is not equivalent to experimental logD at pH 7.4.

## Methods

### Data Splitting

- 85% development set and 15% held-out test set.
- Stratification by molecular-weight group: ≤500 Da and >500 Da.
- Five-fold cross-validation within the development set, also stratified
  by molecular-weight group.
- Identical folds are used across candidate models.
- Random seeds are fixed for reproducibility.

### Models

- Dummy mean predictor
- Ridge regression
- RBF Kernel Ridge regression
- Random Forest regression

Missing descriptor values are imputed within the modeling pipeline.
Ridge and Kernel Ridge use standardization fitted within each
cross-validation training fold.

Hyperparameters and the final model are selected using development-set
cross-validation RMSE. Test results are not used to select the model.

### Evaluation

Performance is reported using:

- Mean absolute error (MAE)
- Root mean squared error (RMSE)
- Coefficient of determination (R²)

The CV-selected model is evaluated separately for the two MW groups.

The group comparison includes:

- Difference in MAE: >500 Da minus ≤500 Da
- Within-group bootstrap 95% confidence interval with 10,000 resamples
- Two-sided Welch's test on absolute prediction errors
- Exploratory LOWESS error trends
- Descriptive analysis within the 400–600 Da range

### Exploratory Analysis

The notebook generates:

- Overall and MW-group summary statistics
- Descriptor and target distribution plots
- MW-tail and IQR outlier summaries
- Spearman correlation plots
- Feature–target scatterplots
- Test prediction and error plots

Valid high-MW observations are retained. IQR flags are used for
inspection rather than automatic deletion.

## Repository Structure

- `README.md`: project description and execution instructions
- `requirements.txt`: pinned package versions
- `.gitignore`: files excluded from version control
- `notebooks/CHE1147_project.ipynb`: complete analysis workflow
- `notebooks/README.md`: notebook overview

When the notebook is run from the `notebooks` directory, generated
outputs are saved in `notebooks/final_project_results/`.

## Environment Setup

The recorded development environment uses Python 3.10.21.
Package versions are listed in `requirements.txt`.

### 1. Download the Repository

On GitHub, select **Code → Download ZIP**, then extract the archive.

Open a terminal in the extracted repository root, where
`requirements.txt` is located.

### 2. Create an Environment

Using Conda:

```bash
conda create -n che1147-logd python=3.10.21 -y
conda activate che1147-logd
```

### 3. Install Dependencies

```bash
python -m pip install -r requirements.txt
```

### 4. Register the Notebook Kernel

```bash
python -m ipykernel install --user \
  --name che1147-logd \
  --display-name "Python (CHE1147 logD)"
```

## Running the Project

From the repository root:

```bash
conda activate che1147-logd
cd notebooks
python -m notebook
```

1. Open `CHE1147_project.ipynb`.
2. Select the **Python (CHE1147 logD)** kernel.
3. If an initial package-installation cell is present, skip it after
   installing `requirements.txt` to preserve the pinned environment.
4. Restart the kernel and execute the remaining cells from top to bottom.
5. Wait for all model searches and analyses to finish.

An internet connection is needed to download the dataset if a local
`Lipophilicity.csv` is not available in the notebook working directory.

Model tuning may take several minutes or longer depending on the
available CPU and memory.

## Generated Outputs

The output directory contains:

- Shared data split and CV-fold assignments
- Development-set descriptive statistics
- Outlier summaries
- Distribution and correlation figures
- Hyperparameter-search results
- CV model comparisons
- Fitted model files
- Test predictions and performance tables
- Bootstrap confidence intervals and Welch-test results
- Error-trend figures

The notebook is the source for preprocessing, model training,
statistical analysis, and figure generation.

## Interpretation and Limitations

The high-MW group contains fewer compounds, which limits precision.
Differences in target distributions can affect group-level prediction
metrics.

Random compound splitting does not establish generalization to unseen
chemical scaffolds. Statistical inference assumes independent test
observations and is conditional on the fitted model.

The test set was inspected during project development, including before
the standardization correction. Results should therefore be interpreted
as an internal evaluation rather than independent external validation.

The dependency versions record the development environment. Installation
and full execution in a clean environment should be verified before final
submission.

## References

- DeepChem Lipophilicity dataset:
  https://deepchemdata.s3-us-west-1.amazonaws.com/datasets/Lipophilicity.csv
- RDKit: https://www.rdkit.org/
- scikit-learn: https://scikit-learn.org/
- SciPy: https://scipy.org/
- statsmodels: https://www.statsmodels.org/
