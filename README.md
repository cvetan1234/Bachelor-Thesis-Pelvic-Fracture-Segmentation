# Bachelor's Thesis Tsvetan Stanchev

This repository contains the full pipeline used for the bachelor's thesis on pelvic fracture segmentation with nnU-Net.

The pipeline is organized into five main parts:

1. environment setup
2. data preparation
3. label engineering / dataset generation
4. nnU-Net preprocessing, training, and prediction
5. restoration and evaluation

This README explains the correct execution order and the exact commands to run the project.

---

## 1. Repository structure

```text
Bachelorarbeit_Tsvetan_Stanchev/
├── custom_nnunet_trainers/
├── data_preparation_scripts/
├── evaluation_scripts/
├── label_engineering_scripts/
├── original_PENGWIN_data/
├── restoration_scripts/
├── set_up_env/
└── train_val_test_nnunet_scripts/
```

- `original_PENGWIN_data/` input dataset
- `set_up_env/` environment setup files
- `data_preparation_scripts/` initial preprocessing and split creation
- `label_engineering_scripts/` generation of all derived nnU-Net datasets
- `train_val_test_nnunet_scripts/` split creation for nnU-Net, preprocessing, training, prediction
- `restoration_scripts/` conversion of semantic predictions back to fragment instances
- `evaluation_scripts/` quantitative evaluation and CSV generation

---

## 2. Requirements before starting

The pipeline assumes:

- Linux
- Python 3 with `venv`
- a working CUDA / PyTorch installation
- the original PENGWIN training data available locally

### Expected raw data structure

Before running anything, place the original PENGWIN Challenge 2024 input data, that is available on https://zenodo.org/records/10927452 in:

```text
original_PENGWIN_data/
├── images/
└── labels/
```

Both folders must contain matching case files.

---
### Windows support
---

Although the pipeline is primarily designed for Linux, it can also be executed on Windows with some adjustments.

The core Python scripts are platform-independent and will run on Windows. However, some parts of the pipeline rely on Linux/Unix-specific behavior and therefore require modification:

- **Path handling**
  - Some scripts assume Unix-style paths (`/`).
  - On Windows, ensure compatibility with `Path` from `pathlib` or adjust hardcoded paths if necessary.

- **Environment activation**
  - Linux:
    ```bash
    source set_up_env/nnunet_env/bin/activate
    ```
  - Windows:
    ```bash
    set_up_env\nnunet_env\Scripts\activate
    ```

- **Shell commands in scripts**
  - Some wrapper scripts assume a Bash environment (e.g., `nohup`, `CUDA_VISIBLE_DEVICES`).
  - These need to be removed or replaced with Windows-compatible alternatives.

- **CUDA / GPU handling**
  - Environment variable usage may differ on Windows.
  - GPU selection logic inside scripts may need minor adjustments.

- **Git and system tools**
  - Automatic steps that assume Linux tools (e.g., Git availability, shell execution) may require manual setup on Windows.

For best compatibility, it is recommended to run the pipeline:

- either on a Linux system  
- or via **WSL (Windows Subsystem for Linux)** on Windows  

Running natively on Windows is possible but requires small manual adaptations in the scripts mentioned above.

---
## 3. Create the nnU-Net environment

Go to the repository root and run:

```bash
cd Bachelorarbeit_Tsvetan_Stanchev
python set_up_env/set_up_env.py
```

Then activate the environment:

```bash
source set_up_env/nnunet_env/bin/activate
```

What this setup script does:

- creates the virtual environment
- installs `torch`, `torchvision`, and the packages from `set_up_env/requirements.txt`
- installs `nnunetv2`
- copies the two custom trainers into the installed nnU-Net trainer directory

---

### nnU-Net environment variables
---

The setup script automatically configures the required nnU-Net environment variables:

- `nnUNet_raw`
- `nnUNet_preprocessed`
- `nnUNet_results`

These variables point to the `nnUNet/` directory inside the repository and are required for all nnU-Net operations (preprocessing, training, and prediction).

No manual configuration is necessary as long as the setup script is used.

---

## 4. Initial data preparation

Run the following scripts **from the repository root** and in this exact order:

### 4.1 Align images and labels and convert to `.nii.gz`

```bash
python data_preparation_scripts/align_and_convert_data.py
```

This creates:

```text
aligned_and_converted_data/
├── images/
└── labels/
```

### 4.2 Analyze cases and create the global train/val/test split

```bash
python data_preparation_scripts/data_splitting.py
```

This creates:

- `fracture_analysis.csv`
- `train_val_test.json`

### 4.3 Create the initial nnU-Net dataset structure

```bash
python data_preparation_scripts/create_nnunet_structure.py
```

This creates:

```text
nnUNet/
├── nnUNet_raw/
│   └── Dataset001_original_data/
├── nnUNet_preprocessed/
└── nnUNet_results/
```

`Dataset001_original_data` is the base dataset used by the later scripts.

---

## 5. Generate all derived datasets (labels engineering)

After `Dataset001_original_data` exists, generate the derived datasets.

These scripts are independent of each other and can be run in any order.

```bash
python label_engineering_scripts/generate_dataset_anatomy.py
python label_engineering_scripts/generate_dataset_singular_network.py
python label_engineering_scripts/generate_dataset_fracture_network_type1.py
python label_engineering_scripts/generate_dataset_fracture_network_type2.py
python label_engineering_scripts/generate_dataset_fracture_network_type3.py
python label_engineering_scripts/generate_dataset_fracture_network_type4.py
python label_engineering_scripts/generate_dataset_fracture_network_type5.py
```

This creates the following datasets inside `nnUNet/nnUNet_raw/`:

- `Dataset002_anatomy_network`
- `Dataset003_singular_network`
- `Dataset004_fracture_network_type1`
- `Dataset005_fracture_network_type2`
- `Dataset006_fracture_network_type3`
- `Dataset007_fracture_network_type4`
- `Dataset008_fracture_network_type5`

---

## 6. Create `splits_final.json` for each dataset

For each dataset that will be trained, create its nnU-Net split file.

Run:

```bash
python train_val_test_nnunet_scripts/create_train_val_split.py --dataset_id 2
python train_val_test_nnunet_scripts/create_train_val_split.py --dataset_id 3
python train_val_test_nnunet_scripts/create_train_val_split.py --dataset_id 4
python train_val_test_nnunet_scripts/create_train_val_split.py --dataset_id 5
python train_val_test_nnunet_scripts/create_train_val_split.py --dataset_id 6
python train_val_test_nnunet_scripts/create_train_val_split.py --dataset_id 7
python train_val_test_nnunet_scripts/create_train_val_split.py --dataset_id 8
```

This writes `splits_final.json` into each dataset folder inside `nnUNet/nnUNet_preprocessed/`.

---

## 7. nnU-Net preprocessing

Preprocess every dataset before training.

Run for each dataset:

```bash
python train_val_test_nnunet_scripts/preprocess.py --dataset_id 2
python train_val_test_nnunet_scripts/preprocess.py --dataset_id 3
python train_val_test_nnunet_scripts/preprocess.py --dataset_id 4
python train_val_test_nnunet_scripts/preprocess.py --dataset_id 5
python train_val_test_nnunet_scripts/preprocess.py --dataset_id 6
python train_val_test_nnunet_scripts/preprocess.py --dataset_id 7
python train_val_test_nnunet_scripts/preprocess.py --dataset_id 8
```

---

## 8. nnU-Net training

Train every dataset after preprocessing.

Run for each dataset:

```bash
python train_val_test_nnunet_scripts/train.py --dataset_id 2 --fold 0 --gpu 0
python train_val_test_nnunet_scripts/train.py --dataset_id 3 --fold 0 --gpu 0
python train_val_test_nnunet_scripts/train.py --dataset_id 4 --fold 0 --gpu 0
python train_val_test_nnunet_scripts/train.py --dataset_id 5 --fold 0 --gpu 0
python train_val_test_nnunet_scripts/train.py --dataset_id 6 --fold 0 --gpu 0
python train_val_test_nnunet_scripts/train.py --dataset_id 7 --fold 0 --gpu 0
python train_val_test_nnunet_scripts/train.py --dataset_id 8 --fold 0 --gpu 0
```

Trainer selection is automatic:

- anatomy dataset `nnUNetTrainer_no_rotation`
- all other datasets `nnUNetTrainer_1000`

If needed, a trainer can also be set manually:

```bash
python train_val_test_nnunet_scripts/train.py --dataset_id 4 --fold 0 --gpu 0 --trainer nnUNetTrainer_1000
```

---

## 9. nnU-Net prediction

After training, generate predictions on the test for each dataset:

```bash
python train_val_test_nnunet_scripts/predict.py --dataset_id 2 --gpu 0
python train_val_test_nnunet_scripts/predict.py --dataset_id 4 --gpu 0
python train_val_test_nnunet_scripts/predict.py --dataset_id 5 --gpu 0
python train_val_test_nnunet_scripts/predict.py --dataset_id 6 --gpu 0
python train_val_test_nnunet_scripts/predict.py --dataset_id 7 --gpu 0
python train_val_test_nnunet_scripts/predict.py --dataset_id 8 --gpu 0
```

Predictions are saved to:

```text
predictions/<dataset_name>/
```

For fracture datasets, the predictions are expected to be the per-region files such as:

- `case_LH.nii.gz`
- `case_RH.nii.gz`
- `case_S.nii.gz`

For anatomy, predictions are expected as:

- `case.nii.gz`

---

## 10. Restoration to instance labels

Restore final instance masks from the predicted outputs.

⚠️ All restoration scripts must be executed from the **project root folder**, since paths are resolved automatically inside the scripts.

The restoration combines:

- anatomy predictions from `Dataset002_anatomy_network`
- fracture predictions from the respective fracture dataset

and reconstructs final fragment instance masks.

Run the restoration scripts:

### Type 1 (Dataset004)

```bash
python restoration_scripts/restore_type1_script1.py
python restoration_scripts/restore_type1_script2.py
python restoration_scripts/restore_type1_script3.py
```

### Type 2 (Dataset005)

```bash
python restoration_scripts/restore_type2_script1.py
python restoration_scripts/restore_type2_script2.py
```

### Type 3 (Dataset006)

```bash
python restoration_scripts/restore_type3_script1.py
```

### Type 4 (Dataset007)

```bash
python restoration_scripts/restore_type4_script1.py
```

### Type 5 (Dataset008)

```bash
python restoration_scripts/restore_type5_script1.py
python restoration_scripts/restore_type5_script2.py
```

The results are saved in:

```
restorations/<script_name>/
```

Each restoration folder contains the final instance masks using the PENGWIN-style label numbering convention.


## 11. Evaluation

Evaluate the restored instance masks against the ground truth.

⚠️ All evaluation scripts must be executed from the **project root folder**, since paths are resolved automatically inside the scripts.

The ground truth labels are in:

```
nnUNet/nnUNet_raw/Dataset001_original_data/labelsTs
```

Run the evaluation:

### Evaluate all restoration folders

```bash
python evaluation_scripts/eval_folder.py
```

### Create the final comparison CSV

After all evaluation CSV files are created:

```bash
python evaluation_scripts/create_comparisson_csv.py
```

The evaluation results are saved in:

```
evaluation/
```

Each CSV file contains the metrics for one restoration approach. The final comparison CSV summarizes all results.


## 12. Recommended complete execution order

From start to finish, the full order is:

1. create the environment
2. activate the environment
3. place the raw data into `original_PENGWIN_data/images` and `original_PENGWIN_data/labels`
4. run `align_and_convert_data.py`
5. run `data_splitting.py`
6. run `create_nnunet_structure.py`
7. generate all derived datasets
8. create `splits_final.json` for each derived dataset
9. preprocess each dataset
10. train each dataset
11. predict each dataset
12. restore instance masks
13. evaluate restored masks

---

## 13. Practical notes

- Always run commands from the **repository root**.
- The wrapper scripts assume Linux shell commands and a Unix-style environment.
- The anatomy network (`Dataset002`) must be trained and predicted before restoration, because all restoration scripts require anatomy predictions.
- Restoration and evaluation should be performed only after the corresponding prediction folders exist.

---

## 14. Minimal example run

A minimal example for one full experiment could look like this:

```bash
# environment
python set_up_env/set_up_env.py
source set_up_env/nnunet_env/bin/activate

# data preparation
python data_preparation_scripts/align_and_convert_data.py
python data_preparation_scripts/data_splitting.py
python data_preparation_scripts/create_nnunet_structure.py

# dataset generation
python label_engineering_scripts/generate_dataset_anatomy.py
python label_engineering_scripts/generate_dataset_fracture_network_type1.py

# create splits
python train_val_test_nnunet_scripts/create_train_val_split.py --dataset_id 2
python train_val_test_nnunet_scripts/create_train_val_split.py --dataset_id 4

# preprocess
python train_val_test_nnunet_scripts/preprocess.py --dataset_id 2
python train_val_test_nnunet_scripts/preprocess.py --dataset_id 4

# train
python train_val_test_nnunet_scripts/train.py --dataset_id 2 --fold 0 --gpu 0
python train_val_test_nnunet_scripts/train.py --dataset_id 4 --fold 0 --gpu 0

# predict
python train_val_test_nnunet_scripts/predict.py --dataset_id 2 --gpu 0
python train_val_test_nnunet_scripts/predict.py --dataset_id 4 --gpu 0

# restore
python restoration_scripts/restore_type1_script1.py
python restoration_scripts/restore_type1_script2.py
python restoration_scripts/restore_type1_script3.py

# evaluate
python evaluation_scripts/eval_folder.py
python evaluation_scripts/create_comparisson_csv.py
```

---

## 15. Output overview

After a full run, the most important output folders are:

```text
aligned_and_converted_data/
nnUNet/
predictions/
restorations/
evaluation/
```

These correspond to:

- `aligned_and_converted_data/` — aligned and converted raw data  
- `nnUNet/` — nnU-Net datasets, preprocessing, and training results  
- `predictions/` — semantic segmentation outputs  
- `restorations/` — restored instance segmentation results  
- `evaluation/` — evaluation CSV files and comparisons  
