# Multimodal Skin Lesion Classification Using Clinical Images and Structured Patient Information

## Overview

Skin cancer is a significant healthcare concern, and early identification of suspicious skin lesions is important. Deep learning models can analyse clinical images, while structured patient information can provide additional context for classification.

This project investigates a multimodal approach to skin lesion classification by combining clinical images with structured patient information. It compares image-only, clinical-only, and multimodal classification approaches using the PAD-UFES-20 dataset.

The primary objective is to evaluate whether combining visual and structured clinical information improves classification performance compared with using either modality independently.

**Disclaimer:** This project is intended for academic research and educational purposes only. It is not a clinically validated diagnostic system and must not be used for medical decision-making.

---

## Research Questions

The project investigates four research questions:

- **RQ1:** How well can skin lesions be classified using clinical images alone?
- **RQ2:** How well can skin lesions be classified using structured clinical data alone?
- **RQ3:** Does combining image and clinical information improve classification performance compared with either modality independently?
- **RQ4:** What challenges and limitations arise when developing multimodal skin-lesion classification models using a small and imbalanced medical dataset, including incomplete clinical information?

---

## Dataset

**Dataset:** [PAD-UFES-20](https://doi.org/10.17632/zr7vgbcyr2.1)

PAD-UFES-20 is a publicly available dataset containing smartphone clinical images of skin lesions and associated patient and lesion information.

### Dataset Summary

| Attribute | Details |
|---|---|
| Original records | 2,298 |
| Image modality | Smartphone clinical images |
| Structured modality | Patient and lesion metadata |
| Original diagnostic categories | 6 |
| Unique patients in original metadata | 1,373 |
| Image format | PNG |

### Target Class Mapping

The original six diagnostic categories were consolidated into three target classes:

| Target Class | Original Diagnostic Labels |
|---|---|
| Malignant | BCC, SCC, MEL |
| Precancerous | ACK |
| Benign | NEV, SEK |

### Data Cleaning and Preprocessing

The dataset underwent several preprocessing and quality-checking steps:

- Verified image availability and metadata-to-image matching.
- Checked image integrity and dimensions.
- Identified exact duplicate images using image hashing.
- Excluded 24 records belonging to conflicting-label duplicate groups.
- Removed three redundant records from same-label duplicate groups.
- Preserved the original dataset files and metadata.
- Created the three-class target variable.

After cleaning, **2,271 records** remained.

### Cleaned Dataset Distribution

| Class | Records | Percentage |
|---|---:|---:|
| Benign | 477 | 21.00% |
| Malignant | 1,071 | 47.16% |
| Precancerous | 723 | 31.84% |
| **Total** | **2,271** | **100%** |

The dataset is imbalanced, with Malignant being the largest class and Benign the smallest. This motivates the use of macro-averaged evaluation metrics.

---

## Data Splitting

A patient-level data splitting strategy was used to prevent records belonging to the same patient from appearing across different partitions.

The final dataset was divided into:

| Partition | Records | Percentage |
|---|---:|---:|
| Training | 1,608 | 70.81% |
| Validation | 305 | 13.43% |
| Testing | 358 | 15.76% |
| **Total** | **2,271** | **100%** |

The split proportions are approximate because patient-level grouping takes priority over exact partition percentages.

The same saved partitions were used across all three experiments.

Patient identifiers, lesion identifiers, image identifiers, filenames, diagnostic labels and biopsy-status information were excluded from model inputs.

---

## Methodology

Three classification experiments were conducted.

### 1. Image-Only Classification

A pretrained ResNet50 convolutional neural network was used for image classification.

- **Input:** Clinical skin lesion images.
- Image size: 224 × 224 pixels.
- Transfer learning using ImageNet-pretrained weights.
- Initial training with the convolutional backbone frozen.
- Fine-tuning of the final residual block and classification layer.
- Three-class output layer.

### 2. Clinical-Only Classification

Structured clinical information was used independently of the images.

Six clinical features were selected:

- Age
- Body region
- Itching
- Pain
- Bleeding
- Elevation

Preprocessing included:

- StandardScaler for numerical age values.
- OneHotEncoder for categorical features.
- Preprocessing fitted using training data only.

Several classical machine learning models were evaluated, including:

- Logistic Regression
- Random Forest
- Support Vector Machine (SVM)
- XGBoost

The original RBF-kernel SVM was selected as the clinical model based on validation Macro-F1.

### 3. Multimodal Classification

A feature-level fusion architecture was developed to combine image and structured clinical information.

The architecture consists of:

1. **Image branch:** Fine-tuned ResNet50 encoder producing a 2,048-dimensional feature representation.
2. **Clinical branch:** Multilayer perceptron transforming 27 preprocessed clinical features into a 32-dimensional representation.
3. **Fusion layer:** Concatenation of image and clinical feature representations.
4. **Classification head:** Fully connected layers producing predictions for the three target classes.

The fused representation contains 2,080 features.

The image encoder was frozen during multimodal training, while the clinical branch and fusion classifier were trained.

---

## Experimental Results

All three models were evaluated on the same untouched test partition containing 358 records.

### Test Performance Comparison

| Metric | Clinical-Only SVM | Image-Only ResNet50 | Multimodal |
|---|---:|---:|---:|
| Accuracy | 68.72% | 74.30% | **75.98%** |
| Macro Precision | 67.29% | 76.08% | **76.26%** |
| Macro Recall | 69.62% | 73.32% | **76.17%** |
| Macro-F1 | 67.93% | 74.48% | **76.21%** |
| Malignant Recall | 65.70% | **79.07%** | 77.33% |
| Macro ROC-AUC | 0.8540 | 0.8788 | **0.8931** |

### Key Findings

- The multimodal model achieved the highest overall accuracy and Macro-F1.
- Multimodal fusion improved Macro-F1 by approximately 1.73 percentage points compared with image-only classification.
- The image-only model achieved the highest malignant recall at 79.07%.
- The clinical-only model achieved the lowest overall performance among the three approaches.
- Combining both modalities improved overall classification performance, but did not improve every individual metric.

These findings provide evidence of a modest benefit from multimodal feature fusion on the evaluated test partition.

The results are based on a single held-out test split and do not establish statistical significance or universal superiority of multimodal models.

---

## Evaluation Metrics

The following metrics were used:

- Accuracy
- Macro Precision
- Macro Recall
- Macro-F1
- Per-class Precision, Recall and F1-score
- Malignant Recall
- Multiclass ROC-AUC

Macro-F1 was treated as an important evaluation metric because the dataset contains imbalanced target classes.

Malignant recall was also examined separately because failing to identify malignant lesions is an important limitation in skin lesion classification.

---

## Visual Results

The final evaluation figures are available in `results/final/figures/`.

The figures include:

- Cleaned dataset class distribution.
- Class distribution across training, validation and test partitions.
- Clinical-only confusion matrix.
- Image-only confusion matrix.
- Multimodal confusion matrix.
- Combined ROC curve comparison.
- Model performance comparison.
- Image-only training and validation curves.
- Multimodal training and validation curves.

These visualisations support the quantitative results and help analyse class-wise performance and model generalisation.

---

## Limitations

The project has several limitations:

1. **Limited dataset size:** PAD-UFES-20 contains a relatively small number of records for training deep learning models.
2. **Class imbalance:** The three target classes are not equally represented.
3. **Incomplete clinical information:** Several structured metadata fields contain missing values, and the availability of clinical information varies across diagnostic groups.
4. **Generalisation gap:** Validation performance was higher than test performance across all three models.
5. **Malignant classification errors:** Some malignant cases were incorrectly classified as benign or precancerous.
6. **Single dataset:** The models were evaluated using one dataset and one patient-level test split. External validation is still required.
7. **Clinical applicability:** This is a retrospective research experiment and not a clinically validated diagnostic tool.

---

## Project Structure

```text
Multimodal-Skin-Lesion-Classification/
│
├── app/
│   └── components/
│
├── configs/
│   └── config.yaml
│
├── data/
│   ├── raw/
│   ├── interim/
│   │   └── cleaned_metadata_with_splits.csv
│   └── processed/
│
├── docs/
│
├── models/
│   ├── clinical/
│   │   └── best_clinical_svm.pkl
│   ├── image/
│   │   └── best_resnet50.pth
│   └── multimodal/
│       └── best_multimodal_model.pth
│
├── notebooks/
│   ├── 01_dataset_audit.ipynb
│   ├── 02-image-classification.ipynb
│   ├── 03-multimodal-classification.ipynb
│   └── 04-final-evaluation.ipynb
│
├── results/
│   ├── clinical/
│   ├── image/
│   ├── multimodal/
│   └── final/
│       ├── figures/
│       ├── final_model_comparison.csv
│       ├── clinical_only_test_report.json
│       ├── image_only_test_report.json
│       └── multimodal_test_report.json
│
├── src/
│   ├── data/
│   ├── models/
│   ├── training/
│   └── utils/
│
├── .gitignore
├── AGENTS.md
├── README.md
└── requirements.txt
```

**Note:** The current experimental implementations are documented in Jupyter notebooks. The `src/` directories are reserved for future modularisation.

The original dataset and large trained model checkpoints are maintained separately and are not included in the Git repository.

---

## Installation

### 1. Clone the Repository

```bash
git clone https://github.com/MannuGaurav/Multimodal-Skin-Lesion-Classification.git

cd Multimodal-Skin-Lesion-Classification
```

### 2. Create a Virtual Environment

```bash
python -m venv .venv
```

### 3. Activate the Environment

**Windows:**

```powershell
.venv\Scripts\activate
```

### 4. Install Dependencies

```bash
pip install -r requirements.txt
```

For GPU-based deep learning experiments, install a compatible PyTorch and Torchvision build for your CUDA environment. Kaggle provides a GPU-enabled environment for the deep learning notebooks.

---

## Execution Environment

The experiments were conducted using:

- **Clinical-only experiment:** Python and Scikit-learn.
- **Image-only experiment:** Kaggle Notebook with GPU acceleration (NVIDIA Tesla T4).
- **Multimodal experiment:** Kaggle Notebook with GPU acceleration (NVIDIA Tesla T4).
- **Final evaluation:** Kaggle Notebook using the saved model checkpoints and test partition.

The original PAD-UFES-20 dataset and trained model checkpoints must be obtained separately.

Kaggle dataset paths and notebook configurations may need to be adjusted for other environments.

---

## Reproducibility

The experiments were conducted using Python and the libraries listed in `requirements.txt`.

The original dataset and trained model checkpoints are not included in the Git repository. They must be obtained separately to reproduce the experiments.

The cleaned metadata with patient-level partitions is saved as:

```text
data/interim/cleaned_metadata_with_splits.csv
```

The saved metadata contains the final training, validation and test assignments used across all three experiments.

The notebooks document the dataset audit, individual experiments, multimodal fusion and final test evaluation.

For reproducible model loading, use compatible library versions, particularly the Scikit-learn version used to save the clinical SVM pipeline.

The saved split should be reused for reproducing the reported comparison. Generating a new patient-level split may produce different results.

---

## Future Improvements

Potential future work includes:

- Evaluation on larger and more diverse skin lesion datasets.
- External validation across different patient populations.
- Improved handling of incomplete clinical information.
- More extensive hyperparameter optimisation.
- Regularisation and training strategies to reduce overfitting.
- Investigation of class-specific performance and malignant false negatives.
- Model interpretability using Grad-CAM and feature importance methods.

---

## Disclaimer

This project is developed for academic and educational research. Its predictions are experimental and must not be interpreted as medical diagnoses or used as a substitute for professional medical advice.

---

## Author

**Mannu Gaurav**

B.Tech – Computer Science and Engineering  
KIIT, Bhubaneswar, India

GitHub: [MannuGaurav](https://github.com/MannuGaurav)
