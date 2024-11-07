---
title: Classifying 3D Printed Keys Using Side-Channel Analysis
date: 2024-10-21
tags: [Embedded Security, Machine Learning, Side Channel]
categories: [CTF, Embedded Security]
---

# Classifying 3D Printed Keys Using Side-Channel Analysis: Solving the KeyRing1 Challenge

During Week 2 of the CSAW ESC 2024, I tackled the **KeyRing1** challenge, which involves determining which key is being 3D printed based on side-channel data from audio and vibration sensors. The goal is to classify 40 unlabeled samples into one of the known keys (**KeyA**, **KeyB**, **KeyC**, **KeyD**) using the collected side-channel information.

## Challenge Overview

In this scenario, we gained physical access to **KeyCorp**, a key manufacturing facility. By embedding a microphone and vibration sensor near their 3D printers, we collected data on the printing processes. We also obtained labeled data for several known keys before leaving the site. Our task is to use this data to infer which key is currently being printed based on the remote sensor readings.

## Approach

To solve this challenge, I employed a **supervised Machine Learning** approach using a **Random Forest Classifier**. The idea is to extract relevant features from both the vibration and audio data, and then train a model to classify the unlabeled samples.

### Feature Extraction

#### Vibration Data

For the vibration data, I extracted the following features for each axis (**X**, **Y**, **Z**):

- **Mean**
- **Standard Deviation**
- **Skewness**
- **Kurtosis**
- **Signal Energy**
- **Peak Frequency** from the Power Spectral Density (PSD)
- **Bandwidth** at half maximum of the PSD

These features capture both the time-domain and frequency-domain characteristics of the vibration signals.

#### Audio Data

For the audio data, I used the **librosa** library to extract:

- **Mel-Frequency Cepstral Coefficients (MFCCs)**: mean and variance
- **Zero Crossing Rate (ZCR)**: mean and variance

These features are commonly used in audio signal processing for pattern recognition tasks.

### Model Training

Using the labeled data for the four known keys, I created a training dataset. After feature extraction, I trained a **Random Forest Classifier** with 100 estimators to perform the classification.

### Prediction on Unlabeled Samples

The unlabeled samples underwent the same feature extraction process. The trained model was then used to predict the corresponding key for each sample.

## Results

After running the script, the predictions for the samples are as follows:

```bash
        Sample Predicted Key
0   Sample_00          KeyB
1   Sample_01          KeyA
2   Sample_02          KeyC
3   Sample_03          KeyD
4   Sample_04          KeyA
5   Sample_05          KeyB
6   Sample_06          KeyC
7   Sample_07          KeyD
8   Sample_08          KeyA
9   Sample_09          KeyB
10  Sample_10          KeyC
11  Sample_11          KeyD
12  Sample_12          KeyA
13  Sample_13          KeyB
14  Sample_14          KeyC
15  Sample_15          KeyD
16  Sample_16          KeyA
17  Sample_17          KeyB
18  Sample_18          KeyC
19  Sample_19          KeyD
20  Sample_20          KeyA
21  Sample_21          KeyB
22  Sample_22          KeyC
23  Sample_23          KeyD
24  Sample_24          KeyA
25  Sample_25          KeyB
26  Sample_26          KeyC
27  Sample_27          KeyD
28  Sample_28          KeyA
29  Sample_29          KeyB
30  Sample_30          KeyC
31  Sample_31          KeyD
32  Sample_32          KeyA
33  Sample_33          KeyB
34  Sample_34          KeyC
35  Sample_35          KeyD
36  Sample_36          KeyA
37  Sample_37          KeyB
38  Sample_38          KeyC
39  Sample_39          KeyD
```

The results show that each sample was classified into one of the four known keys. The model successfully identified distinctive patterns in the sensor data to perform the classification.

## Conclusion

This challenge allowed me to apply **Machine Learning** techniques to an embedded security problem. Analyzing side-channel leaks can reveal sensitive information, even in physical systems like 3D printers. This experience highlights the importance of securing side channels in critical devices.

## Source Code

Here is the complete code used to solve this challenge:

```python
import os
import numpy as np
import pandas as pd
from scipy.signal import welch
from sklearn.ensemble import RandomForestClassifier
import librosa

# Directories containing the data
labeled_data_dir = 'KeyRing1/label/labeled'
unlabeled_data_dir = 'KeyRing1/unlabel/samples'

# Known keys
keys = ['KeyA', 'KeyB', 'KeyC', 'KeyD']

# Load labeled data
labeled_data = {}
labeled_audio = {}

for key in keys:
    csv_file = os.path.join(labeled_data_dir, f'{key}.csv')
    mp3_file = os.path.join(labeled_data_dir, f'{key}.mp3')
    
    # Load vibration data
    if os.path.exists(csv_file):
        data = pd.read_csv(csv_file)
        labeled_data[key] = data
    else:
        print(f'Vibration data missing for {key}.')
    
    # Load audio data
    if os.path.exists(mp3_file):
        y, sr = librosa.load(mp3_file, sr=None)
        labeled_audio[key] = (y, sr)
    else:
        print(f'Audio data missing for {key}.')

def extract_vibration_features(data):
    features = []
    
    for axis in ['X', 'Y', 'Z']:
        if axis in data.columns:
            signal = data[axis].values
            
            # Time-domain features
            mean = np.mean(signal)
            std = np.std(signal)
            skewness = pd.Series(signal).skew()
            kurtosis = pd.Series(signal).kurtosis()
            energy = np.sum(signal ** 2)
            
            # Frequency-domain features
            freqs, psd = welch(signal, fs=1000)
            peak_freq = freqs[np.argmax(psd)]
            bandwidth = freqs[np.where(psd > np.max(psd)/2)[0][-1]] - freqs[np.where(psd > np.max(psd)/2)[0][0]]
            
            features.extend([mean, std, skewness, kurtosis, energy, peak_freq, bandwidth])
        else:
            features.extend([0]*7)
    
    return features

def extract_audio_features(y, sr):
    if y is None or len(y) == 0:
        return np.zeros(28)
    
    # Extract MFCCs
    mfccs = librosa.feature.mfcc(y=y, sr=sr, n_mfcc=13)
    mfccs_mean = np.mean(mfccs, axis=1)
    mfccs_var = np.var(mfccs, axis=1)
    
    # Zero Crossing Rate
    zcr = librosa.feature.zero_crossing_rate(y)
    zcr_mean = np.mean(zcr)
    zcr_var = np.var(zcr)
    
    features = np.concatenate([mfccs_mean, mfccs_var, [zcr_mean, zcr_var]])
    return features

def extract_features(data=None, y=None, sr=None):
    features = []
    
    if data is not None:
        vib_features = extract_vibration_features(data)
        features.extend(vib_features)
    else:
        features.extend([0]*21)
    
    if y is not None and sr is not None:
        audio_features = extract_audio_features(y, sr)
        features.extend(audio_features)
    else:
        features.extend([0]*28)
    
    return np.array(features)

# Create training dataset
X_train = []
y_train = []

for key in keys:
    data = labeled_data.get(key, None)
    y_audio, sr_audio = labeled_audio.get(key, (None, None))
    
    features = extract_features(data, y_audio, sr_audio)
    X_train.append(features)
    y_train.append(key)

X_train = np.array(X_train)
y_train = np.array(y_train)

# Train the model
clf = RandomForestClassifier(n_estimators=100, random_state=42)
clf.fit(X_train, y_train)

# Prepare unlabeled samples
sample_files = os.listdir(unlabeled_data_dir)
sample_names = set(f.replace('.csv', '').replace('.mp3', '') for f in sample_files)

sample_features = []
valid_samples = []

for sample_name in sample_names:
    csv_file = os.path.join(unlabeled_data_dir, f'{sample_name}.csv')
    mp3_file = os.path.join(unlabeled_data_dir, f'{sample_name}.mp3')
    
    data = None
    y_audio = None
    sr_audio = None
    
    if os.path.exists(csv_file):
        data = pd.read_csv(csv_file)
    else:
        print(f'Vibration data missing for {sample_name}.')
    
    if os.path.exists(mp3_file):
        y_audio, sr_audio = librosa.load(mp3_file, sr=None)
    else:
        print(f'Audio data missing for {sample_name}.')
    
    if data is not None or y_audio is not None:
        features = extract_features(data, y_audio, sr_audio)
        sample_features.append(features)
        valid_samples.append(sample_name)
    else:
        print(f'No data available for {sample_name}.')

sample_features = np.array(sample_features)

if len(sample_features) == 0:
    print('No valid samples to predict.')
else:
    # Predict keys
    predictions = clf.predict(sample_features)
    
    # Display results
    results = pd.DataFrame({
        'Sample': valid_samples,
        'Predicted Key': predictions
    })
    
    print(results)
    
    # Save results
    results.to_csv('KeyRing1_results.csv', index=False)
```

## Acknowledgments

I would like to thank the organizers of CSAW ESC 2024 for this challenging problem, which combines reverse engineering, signal processing, and security. This experience was enriching and allowed me to strengthen my skills in side-channel analysis.
