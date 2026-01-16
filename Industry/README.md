# Touchless Industry Worklog

## Context
This industry-facing work was completed during my internship at Samsung. The goal was to build an end-to-end EMG gesture pipeline: data collection with embedded hardware, sensor data cleaning, preprocessing, feature extraction, and model training.

## Gestures Used
Primary gestures in the internship demo:
- Rotate (label = 1)
- Pinch (label = 2)

![Pinch gesture](Pinch_gesture.png)
![Rotate gesture](Rotate_gesture.webp)

## Data Collection (Arduino/ESP32)
- Hardware: ESP32-S3 with Gravity Analog EMG Sensor (OYMotion)
- Firmware: `esp32_emg_gesture.ino`
- Sampling rate: 1000 Hz
- Analog input: EMG signal read on pin 34
- Windowing: 50-sample windows with a 10-sample step
- On-device features sent over serial as a CSV-like line:
  - Mean, Standard Deviation, RMS, Zero-Crossing Rate
  - Output format: `FEATURES:mean,std,rms,zcr`

## Data Collection Setup (Internship)
Details from the Samsung internship demo setup:
- Position: Flexor Carpi Radialis muscle
- Port: COM9
- Duration per recording: 30 seconds
- Sampling frequency: 500 Hz
- Baud rate: 1,000,000

<img src="Sensor_setup.jpg" alt="Sensor setup" width="200" height="200">
<img src="Collecting_Data.jpg" alt="Collecting data" width="100" height="200">

## Data Cleaning and Preprocessing
- Bandpass/low-pass filtering (Butterworth) to suppress noise
- DC offset removal
- Full-wave rectification
- Moving average smoothing
- Windowing of signals for feature extraction
- Standard scaling of features before model training

## Dataset Preprocessing
- Raw data filtering:
  - Notch filter: 50 Hz
  - Low-pass filter: cutoff 200 Hz
  - High-pass filter: cutoff 20 Hz
- RMS smoothing:
  - Moving RMS window size: 500 samples
  - RMS < 13 -> Rest (0)
  - RMS >= 13 -> Rotate/Pinch

Final dataset summary:
- Total samples: 210,000
- Rest (label = 0): 75,670 (36.0%)
- Rotate (label = 1): 67,774 (32.3%)
- Pinch (label = 2): 66,556 (31.7%)

## Feature Engineering
Typical features extracted from each window include:
- Time-domain: mean, std, variance, RMS, MAV, ZCR, SSC, waveform length
- Frequency-domain: mean/median/peak frequency, band power

## Model Comparison (Internship Results)
| Model | Test Accuracy | Val Accuracy | CV Std | Cost Function |
| --- | --- | --- | --- | --- |
| KNN (K=5) | 0.9907 | 0.9834 | 0.0014 | N/A (instance-based) |
| Neural Network (MLP: 64 -> 32 hidden, max_iter=500) | 0.9881 | 0.9893 | 0.0029 | Log loss |
| Gradient Boosting (n=100, learning_rate=0.1) | 0.9745 | 0.9768 | 0.0024 | Deviance |
| Random Forest (n=100, max_depth=10) | 0.9728 | 0.9728 | 0.0033 | Gini impurity |
| SVM (RBF kernel) | 0.9432 | 0.9432 | 0.0050 | Hinge loss |
| CNN-LSTM | 0.8926 | 0.8917 | 0.0000 | Categorical cross entropy |

## Modeling and Evaluation
- Classical ML: Random Forest, Gradient Boosting, SVM, KNN, MLP
- Deep learning: CNN-LSTM (optional when TensorFlow is available)
- Model selection based on validation accuracy and F1-score
- Best models and scalers saved for inference

## Real-Time Gesture Recognition Flow
Pipeline used for live inference in the demo:
1. Connect EMG sensor (COM9, 1,000,000 baud)
2. Collect raw EMG into a 500-sample buffer
3. Filter the signal (notch, low-pass, high-pass)
4. Remove DC offset (subtract mean)
5. Load pre-trained model and top 15 features
6. Predict gesture class:
   - 0: Rest
   - 1: Rotate
   - 2: Pinch

## Files
- `esp32_emg_gesture.ino`: ESP32-S3 data collection and feature streaming
- `main.ipynb`: Full pipeline (preprocessing, features, training, evaluation)
- `gesture.ipynb`: Gesture analysis and modeling
- `testing.ipynb`: Experiments and model comparison
- `frontend.html`: UI demo and signal visualization
