---
layout: post
title: Creating Fusion of a Complementary Feature Set with MFCC using Librosa
permalink: feature-fusion
date: 2020-7-15
---


In various audio based recognition system, Mel-frequency cepstral coefficients (MFCC) is commonly used for a feature extraction purpose.
However, often MFCC features are combined with other features as well. 
The purpose of this blog post is twofold:  (i) The first is to highlight the use of Librosa library for MFCC feature extraction; (ii) The second is just a note to a small problem that I faced in creating features fusion with existing MFCC features.


### Librosa Library

Librosa is a python library used mostly for music and sound analysis. It provides the building blocks necessary to create a music information retrieval system. This library contains a wealth of sound signal processing tools such as sound reading, sample rate conversion, stft, istft and more.

### Extracting MFCC Features using Librosa

The following snippet code is Librosa functions used to extract MFCC features from an audio.

```python
# Mel-frequency cepstral coefficients (MFCCs) Mel Cepstrum Coefficient
y, sr = librosa.load(librosa.util.example_audio_file(), offset=30, duration=5)
mfcc = librosa.feature.mfcc(y=y, sr=sr)
print(mfcc)

array([[ -5.229e+02,  -4.944e+02, ...,  -5.229e+02,  -5.229e+02],
       [  7.105e-15,   3.787e+01, ...,  -7.105e-15,  -7.105e-15],
       ...,
       [  1.066e-14,  -7.500e+00, ...,   1.421e-14,   1.421e-14],
       [  3.109e-14,  -5.058e+00, ...,   2.931e-14,   2.931e-14]])
```

### Solve feature fusion problem

Using the encapsulated function for mfcc extraction, we get a framed window and a series of processed data. To add other features after mfcc of each frame, we must first get the parameter settings of the frame, then The same framing mechanism is adopted for the fused features to ensure that the fusion of the two is performed under the same frame. For example, we need to fuse the mfcc feature with the short-term energy feature. We need to ensure that the framing is the same in the process of extracting the two, and then splicing the obtained features. The code fusion of mfcc and energy is as follows:

```python
import librosa
import numpy as np

# Load Audio
y, sr = librosa.load(librosa.util.example_audio_file(), offset=30, duration=5)

# Extract MFCC
mfcc = librosa.feature.mfcc(y=y, sr=sr)


#Frame and then find the short-time Fourier transform, the framing parameters are the same as the logarithmic energy mel filter bank parameters.
stft_coff=abs(librosa.stft(y,1024,512,1024)) 

#Get the average energy of each frame
Energy = np.sum(np.square(stft_coff),0)

#Create a fusion feature of MFCC and Enerygy retrieved from each frame with the short-term energy 
MFCC_Energy = np.vstack((mfcc,Energy)) 
```
------

```python
print(mfcc.shape)
(21, 216)

print(Energy.shape)
(216,)

print(MFCC_Energy.shape)
(21, 216)
```
After combining MFCC with Energy feature, we can observe that that dimension of new feature (MFCC_Energy) become 21 features accross 216 instances (i.e frame).


### References

[1] [Librosa Library](https://github.com/librosa/librosa)
