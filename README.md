# Kidney Stone Detection with YOLOv10 and YOLOv8

This repository contains a Kaggle notebook experiment for detecting kidney stones in CT/medical images with object detection models. The project fine-tunes and compares **YOLOv8m** and **YOLOv10m** on a Roboflow-style kidney stone dataset.

> **Medical disclaimer:** This project is for education and research only. It is not a clinical diagnostic tool and must not be used as a substitute for professional medical review.

## Project Goals

- Train YOLOv8 and YOLOv10 models for kidney stone localization.
- Compare model quality using precision, recall, mAP50, and mAP50-95.
- Visualize predictions on sample test images.
- Provide a reproducible baseline that can be improved with better dataset documentation, hyperparameter tuning, and test-set evaluation.

## Repository Contents

```text
.
├── README.md
├── requirements.txt
└── kidney-stone-detection-with-yolov10-and-yolov8.ipynb
```

| File | Purpose |
| --- | --- |
| `kidney-stone-detection-with-yolov10-and-yolov8.ipynb` | Main Kaggle notebook for installing dependencies, loading data, training YOLOv8/YOLOv10, running inference, and plotting metrics. |
| `requirements.txt` | Baseline Python dependencies for local experimentation. Kaggle may already provide some packages. |
| `README.md` | Project overview, setup notes, results summary, limitations, and next steps. |

## Dataset

The notebook expects a YOLO-format dataset mounted in Kaggle at:

```text
/kaggle/input/kidney-stone-images/data.yaml
/kaggle/input/kidney-stone-images/train
/kaggle/input/kidney-stone-images/valid
/kaggle/input/kidney-stone-images/test
```

The notebook description states that the dataset was downloaded from Roboflow Universe. The saved training logs show:

- Training split: **1054 images**, including **1 background image**.
- Validation split: **123 images**.
- Validation instances: **325 annotated kidney stone objects**.

If you run this project outside Kaggle, update the dataset paths in the `CFG` class inside the notebook:

```python
DATA_PATH = '/path/to/data.yaml'
SAMPLE_PATH = '/path/to/test/images/*'
```

## Environment

The notebook was originally run in a Kaggle environment with GPU support. The saved logs indicate:

- Python 3.10
- PyTorch 2.1.2
- NVIDIA Tesla T4 GPU
- Ultralytics YOLOv8.1.34 during training

For local experimentation, create an environment and install the baseline requirements:

```bash
python -m venv .venv
source .venv/bin/activate
pip install -r requirements.txt
```

YOLOv10 is installed in the notebook directly from the THU-MIG repository:

```bash
pip install -q git+https://github.com/THU-MIG/yolov10.git
```

The notebook also downloads the YOLOv10m pretrained weight:

```bash
wget -P /kaggle/working/yolov10/weights -q \
  https://github.com/THU-MIG/yolov10/releases/download/v1.1/yolov10m.pt
```

## How to Run on Kaggle

1. Create a new Kaggle notebook or open the provided notebook.
2. Attach the kidney stone dataset so that it is available as `/kaggle/input/kidney-stone-images`.
3. Enable GPU acceleration.
4. Run the cells from top to bottom.
5. Training outputs are written to:

```text
/kaggle/working/ft_models/yolo_v8
/kaggle/working/ft_models/yolo_v10
```

The notebook loads the trained weights from:

```text
/kaggle/working/ft_models/yolo_v8/weights/best.pt
/kaggle/working/ft_models/yolo_v10/weights/best.pt
```

## Experiment Configuration

The notebook uses the following main configuration:

| Parameter | Value |
| --- | --- |
| Epochs | 50 |
| Seed | 6 |
| Initial learning rate | 0.001 |
| Optimizer | Adam |
| Sample visualizations | 16 images |
| Model 1 | YOLOv8m |
| Model 2 | YOLOv10m |

> Note: `BATCH_SIZE = 32` is declared in the notebook config, but it is not passed into the `train()` calls. The saved YOLOv8 training log shows `batch=16`. Pass `batch=CFG.BATCH_SIZE` explicitly if you want the declared batch size to be used.

## Saved Results

The notebook's saved validation output reports the following results after 50 epochs:

| Model | Train time | Precision | Recall | mAP50 | mAP50-95 | Inference speed |
| --- | ---: | ---: | ---: | ---: | ---: | ---: |
| YOLOv8m | 0.509 hours | 0.779 | 0.717 | 0.765 | **0.328** | 18.1 ms/image |
| YOLOv10m | 0.598 hours | **0.822** | **0.738** | **0.783** | 0.315 | 21.5 ms/image |

Interpretation:

- YOLOv10m performs better on **precision**, **recall**, and **mAP50** in the saved run.
- YOLOv8m performs slightly better on **mAP50-95**, which is stricter about localization quality across multiple IoU thresholds.
- YOLOv10m produces smaller saved weights in the notebook output, but it is slower per image in the saved validation run.

## Notebook Workflow

The notebook is organized around these stages:

1. Clear Kaggle working files.
2. Install Ultralytics, YOLOv10, and supporting libraries.
3. Define configuration and dataset paths.
4. Display random test images.
5. Initialize YOLOv8m and YOLOv10m.
6. Train both models for 50 epochs.
7. Load `best.pt` weights and visualize predictions on 16 test images.
8. Read `results.csv` for each model.
9. Plot YOLOv8 and YOLOv10 loss curves.
10. Plot precision, recall, mAP50, and mAP50-95 comparisons.
11. Display training result images and precision-recall curves.

## Limitations

- The repository currently depends on a Kaggle-mounted dataset and does not include dataset download instructions or a dataset version identifier.
- There is no standalone `train.py`, `predict.py`, or `evaluate.py`; the workflow is notebook-only.
- The saved metrics are validation metrics, not a fully documented independent test-set evaluation.
- The experiment uses a single seed and a single hyperparameter configuration.
- No clinical validation, expert review, external validation dataset, or uncertainty analysis is included.
- The notebook does not include a detailed false-positive/false-negative review.

## Recommended Next Steps

- Add the exact Roboflow dataset link/version and license.
- Move reusable training and inference code into scripts.
- Add config files for paths and hyperparameters.
- Evaluate on a held-out test set and report confusion matrix, false positives, and false negatives.
- Run multiple seeds and report mean/std for metrics.
- Try additional model sizes such as YOLOv8n/s/l and YOLOv10n/s/l.
- Tune batch size, learning rate, augmentation, and confidence thresholds.
- Add sample prediction images to the README if the dataset license permits it.

## Related Project Materials

Class presentation slide link:

https://www.canva.com/design/DAGKhnD8FAY/CZ6fwapKJUjPqQp0rJ64wQ/edit?fbclid=IwZXh0bgNhZW0CMTAAAR1chrcCcMg85s5rlbhWaz1-LOHNPQEPrAqhMzIWTzTA0Ceqj7CBIFFaFGg_aem_Tu4VPhA7nxIuIH9zyvvDKw
