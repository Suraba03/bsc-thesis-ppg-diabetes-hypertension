# Screening for Type 2 Diabetes and Hypertension from Photoplethysmography

Bachelor's thesis project (Moscow Aviation Institute, 2024) by Viacheslav Chekmenev.

Type 2 diabetes and hypertension often stay unnoticed until complications appear. Photoplethysmography (PPG) is a cheap optical signal that smartwatches and fitness bands already record, and both diseases change the shape of the PPG pulse wave. This project explores whether a raw PPG recording can be turned into an estimate of **type 2 diabetes risk** and **blood pressure** using image models.

> **Paper:** a short English preprint of the thesis is included in this repo: [`paper/Chekmenev_2024_PPG_Diabetes_Hypertension.pdf`](paper/Chekmenev_2024_PPG_Diabetes_Hypertension.pdf). It is not peer reviewed, and the thesis itself was written in Russian.

## How it works

```
raw PPG  ->  band-pass filter  ->  continuous wavelet transform  ->  image model  ->  diabetes risk / blood pressure
            (0.5-8 Hz, 125 Hz)      (time-frequency image)           (VGG / Swin)
```

1. **Data.** PPG recordings with sex, age and medical history from the [MIMIC-III](https://physionet.org/content/mimiciii/1.4/) clinical database (read with `wfdb`). Signals sampled at 125 Hz are used.
2. **Filtering.** A third-order Butterworth band-pass filter (0.5-8 Hz) removes high-frequency noise and baseline wander. Synthetic healthy signals and image augmentations are used to reduce overfitting.
3. **Time-frequency image.** Each filtered segment is converted into a 2D image with the continuous wavelet transform (CWT).
4. **Models.** Two image models, both initialised from pre-trained weights, were compared for diabetes classification and for systolic/diastolic blood-pressure regression: a modified **VGG** and a **Swin Transformer**.
5. **Web prototype.** A Flask app that takes a PPG recording and shows every processing step: raw signal, filtered signal, wavelet image and the prediction.

## Results (hold-out set)

| Task | Metric | VGG | Swin Transformer |
|---|---|---|---|
| Type 2 diabetes | AUC-ROC | 0.62 | **0.72** |
| Type 2 diabetes | AUC-PRC | 0.55 | **0.68** |
| Systolic BP | MAE, mmHg | 10.5 | **8.5** |
| Diastolic BP | MAE, mmHg | **5** | 6 |

The Swin Transformer is better for diabetes and systolic pressure, VGG is slightly better for diastolic pressure. These numbers are **not accurate enough for diagnosis**: they are the best values over training epochs on a hold-out set (no separate validation set), the labels come from noisy ICU records, and MIMIC-III patients are not representative of a healthy wearable-user population. See the paper for the full discussion of limitations.

## Repository structure

| Path | Description |
|---|---|
| `app.py` | Flask web app (upload a signal, see each pipeline step and the result) |
| `signal_processing.py` | Signal loading, resampling and cleaning, wavelet image generation, plotting |
| `engine.py` | Model loading and inference |
| `notebooks/1.1-datasets-creation.ipynb` | Building the dataset from MIMIC-III |
| `data/external/2.1-model-training.ipynb` | Model training experiments |
| `data/external/targets.csv` | Per-subject labels and blood-pressure values |
| `data/test_signals/` | Example PPG signals to try in the web app (`Archive.zip`) |
| `templates/`, `static/` | HTML template, styles and images used by the web app |
| `paper/` | English preprint of the thesis (PDF) |

## Running the demo

1. Install the latest Python from <https://www.python.org/downloads/>.
2. Install dependencies:

   ```bash
   pip3 install -r requirements.txt
   ```

3. **Model weights are not publicly available.** The trained `.pt` file is not distributed with this repo. To request it, email chekmenev031@gmail.com. Once you have the file, put it into a `model/` folder in the repo root. The app expects the file name `model/efficientnet-medium.pt`.
4. Unpack `data/test_signals/Archive.zip`. It contains example signals you can upload to the app.
5. Start the app and open the printed local address in your browser:

   ```bash
   python3 app.py
   ```

## Notes

- The demo app in this repo uses an EfficientNetV2-M classifier for the diabetes score, and the blood-pressure values it shows are taken from precomputed estimates in `data/external/targets.csv` rather than computed live. The VGG vs. Swin Transformer comparison from the paper was done in the training notebook.
- MIMIC-III is distributed by PhysioNet under a credentialed-access licence and is not redistributed with this code. You need your own access to rebuild the dataset.
- This is a research prototype and **not a medical device**.

## Citation

```
V. Chekmenev. Screening for Type 2 Diabetes and Hypertension from Photoplethysmography
with Wavelet Images and Vision Transformers. Preprint, 2024.
https://github.com/Suraba03/bsc-thesis-ppg-diabetes-hypertension
```
