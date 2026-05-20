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
data-owner review, and receive outputs that have passed review — either
non-disclosive aggregate metrics (confusion matrices, F1, precision, recall)
or, when row-level outputs are required for the analysis, results returned
with differential privacy applied. The data owner serves as the human-in-the-loop
privacy control that enforces these output policies.

<img width="6295" height="5042" alt="Workflow"
     src="https://github.com/user-attachments/assets/736a5746-7ea7-4f68-b476-aa5e5902a40e" />

## Repository contents

| File | Purpose |
|------|---------|
| `Mock_data_faketucky.ipynb` | Generates structurally valid mock records using the Faker library (Data-Type-Aware Templates method from the paper). |
| `FaketuckySDV.ipynb` | Generates synthetic records from Faketucky using the Synthetic Data Vault (SDV) Gaussian Copula method and compares classification performance against the original data. |
| `Faketucky_classification_original_data_subset.ipynb` | End-to-end retention-prediction pipeline on the original Faketucky data: cleaning, SMOTE class balancing, XGBoost, SHAP, threshold optimization, and risk-group assignment. Serves as the baseline against which the SDV synthetic-data results are compared. |
| `Lowside_faketucky_upload_code.ipynb` | PySyft client code that logs into the low-side server, submits a remote model function for data-owner review, and retrieves results. Requires a live PySyft datasite. |
| `LICENSE` | MIT License covering this repository. |
| `.gitignore` | Excludes data files, model artifacts, credentials, and other items that should not be committed. |

Each notebook begins with an intro markdown cell explaining its purpose,
inputs, and expected outputs. Saved cell outputs in the notebooks let you
browse results without re-running.

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

The first three notebooks are designed to run in **Google Colab** against the
public Faketucky dataset. The fourth notebook (`Lowside_faketucky_upload_code.ipynb`)
is a PySyft client and requires a live PySyft datasite to connect to.

### Environment

The notebooks were developed in Colab and on the project's Docker-based
deployment. Python versions vary between notebooks (3.11.7 and 3.13.0 in the
versions saved here; 3.10.12 / 3.12.3 in the production deployment per Table III).
Package compatibility was sensitive during the project; environment-related
issues are documented in Section III.F of the paper.

### Required packages

Most dependencies install automatically in Colab. If you're running outside Colab,
install:

```bash
pip install pandas scikit-learn imbalanced-learn xgboost \
            sdv faker shap matplotlib seaborn syft
```

PySyft 0.9.2 matches the paper's deployment; newer versions of PySyft may have
API differences.

### Faketucky dataset

The first three notebooks expect to load `faketucky.dta` (33.6 MB). In Colab,
each notebook's first data-loading cell uses `files.upload()` to prompt you to
upload the file from your local machine.

To obtain Faketucky:

```bash
git lfs install
git clone https://github.com/OpenSDP/faketucky.git
```

This puts `faketucky.dta` in the `faketucky/` directory, which you can then
upload to Colab when prompted.

### Run order

1. **`Mock_data_faketucky.ipynb`** — generates Faker-based mock data that
   matches Faketucky's column structure. The output is used to populate the
   low-side PySyft datasite in the production deployment.
2. **`FaketuckySDV.ipynb`** — evaluates SDV Gaussian Copula and differentially
   private SDV synthesizers; compares XGBoost classification on synthetic vs.
   real data. Produces the results in Table VI of the paper (34.4% mean
   degradation).
3. **`Faketucky_classification_original_data_subset.ipynb`** — full classification
   pipeline on the original Faketucky data. Serves as the baseline against
   which the privacy-preserving methods are compared.
4. **`Lowside_faketucky_upload_code.ipynb`** — *requires a live PySyft server.*
   Connects to the low-side datasite, uploads a remote model function, and
   submits a code-review request to the data owner. Credentials are read from
   environment variables (`PYSYFT_URL`, `PYSYFT_EMAIL`, `PYSYFT_PASSWORD`) or
   prompted interactively via `input()` and `getpass`. The notebook does not
   contain saved outputs and cannot be reproduced without a live datasite.

## Reproducibility notes

Notebooks 1–3 are reproducible from this repository once the Faketucky data
is uploaded. The synthetic-data results in Section IV.A of the paper come
from `FaketuckySDV.ipynb`.

Notebook 4 documents the PySyft client workflow but requires the full RDS
infrastructure to execute. The private institutional results in Table VII of
the paper (Inter-Institutional PPML Results on Concordia 2021 data) are not
reproducible outside Concordia for FERPA reasons; they require both the
high-side server and access to the private dataset.

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
