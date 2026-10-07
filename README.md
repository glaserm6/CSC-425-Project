
# Bye-Bye Lanternfly: AI Image Recognition and Counting

**CSC 425 AI Project — Milestone 1**

## Project Overview

**Bye-Bye Lanternfly** is an AI computer-vision project designed to identify and count spotted lanternflies (*Lycorma delicatula*) in user-provided images. The goal is to build a lightweight object-detection system that draws a bounding box around each detected lanternfly and uses the number of detections as the estimated count.

### Input → AI Task → Output

- **Input:** An image supplied by the user
- **AI task:** Object detection using a YOLO model
- **Output:** Bounding boxes around detected spotted lanternflies, confidence scores, and the total number detected

The project proposal specifies approximately 500 labeled images, split into training, validation, and test sets. Preprocessing will include image sizing, augmentation, and bounding-box annotation.

## Why This Matters

Spotted lanternflies are an invasive species with environmental and agricultural impacts. Automated image-based identification could make it easier for individuals to identify lanternflies and record observations. A counting system could also support future monitoring of lanternfly abundance and distribution.

## Proposed Approach

The initial model will use **YOLOv8**, trained in Google Colab with a free GPU. The model will be fine-tuned on a custom spotted-lanternfly dataset.

> **Important data note:** the Hugging Face lanternfly datasets currently discoverable include useful classification/field-image resources, but not all contain YOLO bounding-box annotations. Before training, the project must verify annotation availability and, if necessary, manually or semi-automatically create bounding boxes. The repository therefore treats the dataset as a preparation task rather than assuming the downloaded images are already YOLO-ready.

## Dataset Plan

Primary candidate resources:

1. [Hugging Face — rlogh/lanternfly-images](https://huggingface.co/datasets/rlogh/lanternfly-images)
2. [Hugging Face — rlogh/lanternfly-data](https://huggingface.co/datasets/rlogh/lanternfly-data)
3. Additional open-source images from sources such as iNaturalist/Kaggle, subject to licensing and annotation availability.

The target dataset is approximately 500 useful labeled images. The final number may change after checking image quality, duplicates, licensing, and bounding-box availability.

### YOLO Dataset Layout

```text
data/
├── images/
│   ├── train/
│   ├── val/
│   └── test/
└── labels/
    ├── train/
    ├── val/
    └── test/
```

Each label file uses YOLO object-detection format:

```text
class_id x_center y_center width height
```

All coordinates are normalized from 0 to 1.

## Repository Structure

```text
Bye-Bye-Lanternfly-CSC425/
├── data/                  # Dataset folders; large data is NOT committed
├── docs/
│   ├── milestone1.md
│   └── data_notes.md
├── notebooks/
│   └── 01_data_exploration.ipynb
├── scripts/
│   └── download_hf_dataset.py
├── src/
│   ├── train.py
│   └── predict_count.py
├── models/                # Final .pt weights
├── runs/                  # Training outputs (ignored by Git)
├── data.yaml              # YOLO dataset configuration
├── requirements.txt
├── .gitignore
└── README.md
```

## Installation

Python 3.10 or newer is recommended. Install the dependencies:

```bash
pip install -r requirements.txt
```

## Prepare Hugging Face images

The `rlogh/lanternfly-data` dataset is the closest match for field photographs. The dataset card lists 234 images and a CC BY 4.0 license. Its images have image level species labels and metadata, but no verified YOLO bounding boxes. `rlogh/lanternfly-images` is a smaller 33 original image classification set with augmented copies; those copies are not independent examples and it also has no boxes. These datasets cannot train the object detector until you draw and check a box for every lanternfly.

Download the field images (first run may take a few minutes):

```bash
python scripts/download_hf_dataset.py
```

To use the smaller classification dataset instead:

```bash
python scripts/download_hf_dataset.py --dataset rlogh/lanternfly-images --split original --output data/raw_classification
```

Open the photographs in a bounding-box annotation tool (for example, LabelImg), choose YOLO format, and save each annotation as a `.txt` file with the same base name as its image. Put images and labels into `data/images/train`, `data/images/val`, and `data/images/test` and matching `data/labels/...` folders. Use separate original photo groups in each split; don't put augmented versions of a photo in different splits. Empty label files are needed for images confirmed to contain no lanternflies. Keep the test split untouched until final evaluation. The repository intentionally does not turn an image-level `has_lanternfly` tag into a made-up bounding box.

## Train YOLOv8

Google Colab is recommended for training to avoid overheating a local laptop.

```bash
python src/train.py
```

Training checks that images and labels exist before starting. It uses `yolov8n.pt` as the lightweight pretrained starting point and writes the best trained checkpoint to `runs/lanternfly_yolov8/weights/best.pt`. Copy that file to `models/best.pt` for the phone app. Optional settings include `--epochs`, `--imgsz`, `--batch`, and `--model`.

## Try one image on your computer

```bash
python src/predict_count.py --source path/to/photo.jpg
```

The count is the number of YOLO boxes. Annotated photos are saved under `runs/predict/`.

## Use the phone camera

Start the local web app on your computer after placing trained weights at `models/best.pt`:

```bash
python src/phone_app.py
```

Keep the computer and phone on the same Wi-Fi network. Find the computer's local IP address (`ipconfig` on Windows), then open `http://COMPUTER-IP:5000` on the phone. Tap the photo chooser and use the camera option. The result shows the count and annotated image. Photos are processed locally by the computer and are not saved by the app. Keep this demo on a trusted private network; it is intended for local classroom use, not public hosting.

If no weights exist yet, the page explains the setup needed. The app does not claim predictions from generic pretrained weights.

## Evaluation

The project will evaluate:

- Precision
- Recall
- mAP when available from the YOLO validation process
- Detection/counting accuracy on held-out test images
- Error cases such as small insects, occlusion, poor lighting, cluttered backgrounds, and multiple lanternflies

For counting, the primary application-level measure will compare the predicted number of lanternflies with the manually verified number.

## Timeline

| Period | Target |
|---|---|
| Now–Oct. 11 | Project overview and GitHub setup |
| Oct. 11–25 | Set up/preprocess data and start a baseline |
| Oct. 26–Nov. 15 | Test AI approach and image-classification/detection errors |
| Nov. 16–29 | Refine system and test counting |
| Nov. 30–Dec. 11 | Complete report, demo, and presentation |

## Expected Final Deliverable

- Lightweight trained YOLO weights (`.pt`)
- Python demonstration/inference program
- GitHub repository
- Dataset/setup instructions
- Final report
- Presentation/demo

## Current model status

The repository contains the training pipeline and mobile camera demo. A trained model is not checked in because the Hugging Face candidates currently provide image-level labels, not the object bounding boxes YOLO needs. After boxes are annotated and training finishes, put `best.pt` in `models/` locally; model weights and datasets are excluded from Git by `.gitignore`.

## Team

**Mackenzie Glaser** — Project lead; dataset preparation, preprocessing, model development, evaluation, counting system, documentation, and presentation.

## References

- Jena, Biswajit, et al. “Artificial intelligence-based hybrid deep learning models for image classification: The first narrative review.” *Computers in Biology and Medicine*, 137 (2021): 104803.
- Kräter, M., et al. “AIDeveloper: Deep Learning Image Classification in Life Science and Beyond.” *Advanced Science*, 8 (2021): 2003743. https://doi.org/10.1002/advs.202003743
- Urban, Julie M. “Perspective: shedding light on spotted lanternfly impacts in the USA.” *Pest Management Science*, 76.1 (2020): 10–17.
- Urban, Julie M., and Heather Leach. “Biology and management of the spotted lanternfly, Lycorma delicatula (Hemiptera: Fulgoridae), in the United States.” *Annual Review of Entomology*, 68.1 (2023): 151–167.
- Zhang, Yanlong, et al. “The biology and management of the invasive pest spotted lanternfly, Lycorma delicatula White (Hemiptera: Fulgoridae), in the United States.” *Journal of Plant Diseases and Protection*, 130.6 (2023): 1155–1174.
- Ultralytics YOLO documentation: https://docs.ultralytics.com/
- Hugging Face lanternfly dataset resources: https://huggingface.co/datasets
