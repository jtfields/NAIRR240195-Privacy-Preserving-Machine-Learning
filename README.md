**Note:** Notebooks are being updated to match the workflow described
> below. Some cells may still contain Colab-specific paths or placeholder
> credentials; see comments in each notebook.

# Privacy-Preserving Machine Learning for Student Retention

This repository accompanies the IEEE IRI 2026 paper *"A Privacy-Preserving
Framework Using Remote Data Science for Inter-Institutional Student Retention
Prediction"* and supports the **NAIRR240195** grant from the National
Artificial Intelligence Research Resource (NAIRR) initiative.
[Learn more about NAIRR](https://nairrpilot.org/projects/awarded?_requestNumber=NAIRR240195).

The project develops a privacy-preserving framework that enables researchers
from different institutions to train and validate student-retention models on
sensitive educational data without that data ever leaving the data-owning
institution. The framework is built on [OpenMined's PySyft](https://github.com/OpenMined/PySyft)
and aligns with FERPA (US) and GDPR (EU) privacy requirements.

## Citation

If this work is useful to you, please cite:

> J. Fields, K. M. S. Islam, R. Thota, V. Chen, and P. Madiraju,
> "A Privacy-Preserving Framework Using Remote Data Science for
> Inter-Institutional Student Retention Prediction," in *Proc. 2026 IEEE 27th
> International Conference on Information Reuse and Integration for Data
> Science (IRI)*, Seattle, WA, USA, 2026.

## Architecture

The framework uses a dual-server, semi-air-gapped architecture. The **low-side**
server hosts synthetic and mock data and is accessible to external researchers;
the **high-side** server holds the real institutional data and accepts only
data-owner-approved code submissions.

<img width="1830" height="1275" alt="NAIRR Architecture"
     src="https://github.com/user-attachments/assets/125b7db3-23d8-4db8-b97e-9b73338d1f20" />

### Server specifications (as deployed)

| Component | High Side                                        | Low Side                |
|-----------|--------------------------------------------------|-------------------------|
| Hardware  | Dell Precision T7610 with encrypted RAID drives  | Microsoft Azure cloud   |
| OS        | Ubuntu 24.04                                     | Ubuntu 24.04            |
| Python    | 3.12.3                                           | 3.10.12                 |
| PySyft    | 0.9.2                                            | 0.9.2                   |
| Docker    | 24.0.9                                           | 28.0.4                  |
| Network   | Semi air-gapped; SSH-only via dedicated laptop   | Public, behind PySyft auth |

## Workflow

External researchers prototype models on the low-side, submit code for
data-owner review, and receive only non-disclosive aggregate metrics
(confusion matrices, F1, precision, recall) back from execution on
high-side private data.

<img width="6295" height="5042" alt="Workflow"
     src="https://github.com/user-attachments/assets/736a5746-7ea7-4f68-b476-aa5e5902a40e" />

## Repository contents

| File | Purpose |
|------|---------|
| `Mock_data_faketucky.ipynb` | Generates structurally valid mock records using the Faker library (Data-Type-Aware Templates method from the paper). |
| `FaketuckySDV.ipynb` | Generates synthetic records from Faketucky using the Synthetic Data Vault (SDV) Gaussian Copula method and compares classification performance against the original data. |
| `Faketucky Classification Original Data Subset.ipynb` | End-to-end retention-prediction pipeline on the real Faketucky data: cleaning, SMOTE class balancing, XGBoost, SHAP, threshold optimization, and risk-group assignment. |
| `Lowside_faketucky_upload_code.ipynb` | PySyft client code that logs into the low-side server, submits a remote model function for data-owner review, and retrieves aggregate results. |

## Data sources

This repository does **not** include any data files. There are three datasets
relevant to the project, each handled differently:

- **Faketucky (public).** A synthetic college-going dataset (~85,000 records)
  published by OpenSDP at the Center for Education Policy Research, Harvard
  University. Used for low-side development and validation. Available at
  https://github.com/OpenSDP/faketucky under the MIT License.
- **Mock Faketucky.** Faker-generated records produced by
  `Mock_data_faketucky.ipynb` that preserve the column structure and valid
  ranges of Faketucky but contain no real statistical relationships. Used as
  the low-side mock asset for the PySyft workflow.
- **Concordia 2021 cohort (private).** De-identified institutional data
  (N=720) from Concordia University Wisconsin and Ann Arbor. Resides only on
  the high-side server, protected by FERPA. Not in this repository.

## Getting started

### Prerequisites

- Python 3.10+ (the project was deployed on 3.10.12 low-side and 3.12.3 high-side)
- ~4 GB free disk
- Git LFS (for the Faketucky dataset download)
- A PySyft datasite, if you want to exercise the full RDS workflow

### Install Python dependencies

The notebooks were run with the versions below. Pinning is recommended for
reproducibility; later versions may work but PySyft APIs in particular have
churned between point releases.

```bash
pip install syft==0.9.2 \
            pandas scikit-learn imbalanced-learn xgboost \
            sdv faker shap matplotlib seaborn
```

### Obtain the Faketucky dataset

```bash
git lfs install
git lfs clone https://github.com/OpenSDP/faketucky.git
export FAKETUCKY_PATH="$(pwd)/faketucky/faketucky.dta"
```

The file is 33.6 MB. If your network blocks `media.githubusercontent.com`
(common in corporate or air-gapped environments), download `faketucky.dta`
on a machine that has access and copy it into the working directory.

### Update notebook paths

Each notebook currently has a hardcoded placeholder where the data is loaded.
Replace those with your local path. Recommended pattern (drop into the first
code cell of each notebook):

```python
import os
DATA_PATH = os.environ.get("FAKETUCKY_PATH", "./faketucky.dta")
```

Then change `pd.read_stata("file path")` (or the Colab path in
`FaketuckySDV.ipynb`) to `pd.read_stata(DATA_PATH)`.

### Run order

1. **`Mock_data_faketucky.ipynb`** — generates Faker-based mock data that
   matches Faketucky's column structure. Output is used to populate the
   low-side PySyft datasite.
2. **`FaketuckySDV.ipynb`** — evaluates the SDV Gaussian Copula synthesizer
   and compares XGBoost classification performance on synthetic vs. real data.
3. **`Faketucky Classification Original Data Subset.ipynb`** — full
   classification pipeline on the real Faketucky data. Serves as the baseline
   the privacy-preserving methods are compared against.
4. **`Lowside_faketucky_upload_code.ipynb`** — *requires a live PySyft
   server.* Connects to the low-side datasite, uploads a remote model
   function, and submits a code-review request to the data owner. Replace
   the placeholder credentials at the top with values supplied by the data
   owner, or set:

   ```python
   my_client = sy.login(
       url=os.environ["PYSYFT_URL"],
       email=os.environ["PYSYFT_EMAIL"],
       password=os.environ["PYSYFT_PASSWORD"],
   )
   ```

## Reproducibility notes

Notebooks 1–3 can be run standalone after the Faketucky data is in place.
Notebook 4 requires a running PySyft datasite. The institutional private
data is not reproducible outside Concordia for FERPA reasons; the
synthetic-data results in the paper (Section IV.A) are reproducible from
this repository.

For full operational deployment, the high-side server requires additional
configuration (dedicated hardware, restricted network, IRB-approved data
ingestion) that is outside the scope of this code repository.

## License

MIT License. See `LICENSE` for details. The Faketucky dataset used here is
separately distributed under MIT by OpenSDP.

## Acknowledgments

This work was supported by NAIRR grant NAIRR240195, *Privacy-Preserving
Machine Learning for Improving University Student Retention*. We thank
OpenMined for cloud infrastructure and engineering support, and OpenSDP at
the Center for Education Policy Research at Harvard for publishing
Faketucky.

## Contact

For questions about the framework or institutional collaboration, please
open an issue or contact the corresponding author (see paper).
