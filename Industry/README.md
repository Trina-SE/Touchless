# Touchless Industry Worklog

## Context
This industry-facing work was completed during my internship at Samsung. The goal was to build an end-to-end EMG gesture pipeline: data collection with embedded hardware, preprocessing, feature extraction, and model training.

## Data Collection (Arduino/ESP32)
- Hardware: ESP32-S3 with Gravity Analog EMG Sensor (OYMotion)
- Firmware: `Industry/esp32_emg_gesture.ino`
- Sampling rate: 1000 Hz
- Analog input: EMG signal read on pin 34
- Windowing: 50-sample windows with a 10-sample step
- On-device features sent over serial as a CSV-like line:
  - Mean, Standard Deviation, RMS, Zero-Crossing Rate
  - Output format: `FEATURES:mean,std,rms,zcr`

## Data Cleaning and Preprocessing
Based on notebooks in this folder (see `Industry/main.ipynb`, `Industry/gesture.ipynb`, `Industry/testing.ipynb`):
- Bandpass/low-pass filtering (Butterworth) to suppress noise
- DC offset removal
- Full-wave rectification
- Moving average smoothing
- Windowing of signals for feature extraction
- Standard scaling of features before model training

## Feature Engineering
Typical features extracted from each window include:
- Time-domain: mean, std, variance, RMS, MAV, ZCR, SSC, waveform length
- Frequency-domain: mean/median/peak frequency, band power

## Modeling and Evaluation
- Classical ML: Random Forest, Gradient Boosting, SVM, KNN, MLP
- Deep learning: CNN-LSTM (optional when TensorFlow is available)
- Model selection based on validation accuracy and F1-score
- Best models and scalers saved for inference

## Files
- `Industry/esp32_emg_gesture.ino`: ESP32-S3 data collection and feature streaming
- `Industry/main.ipynb`: Full pipeline (preprocessing, features, training, evaluation)
- `Industry/gesture.ipynb`: Gesture analysis and modeling
- `Industry/testing.ipynb`: Experiments and model comparison
- `Industry/frontend.html`: UI demo and signal visualization

