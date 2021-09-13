# Gia's Noises
Simple audio classifier for the different noises my dog makes. Built with Keras. 

My dog is very vocal and friendly. Not everyone realizes that most of her barks are friendly. 

I collected the data by duct taping an audio recorder to her collar when we went on walks/drives/played. 

The following explains my thought process in data collection, feature extraction, and model building. 

## Major hurdles: 
- wav files obscured with background noise (either from other audio events or transducer noise)
- limited number of samples per label
- unbalanced data sets 
- inconsistent data: varying loudness and time durations

## Overall steps: 
1. Data generation 

2. Data cleanup/preprocessing 
- Split m4a files: https://audiotrimmer.com/

3. Feature extraction 

4. Model building 

5. Model evaluation 

## Data generation: 
What will my labels be? 
1. she needs to use the bathroom 
2. excited to play 
3. friendly bark (happy to see you) 
4. alert bark (protective) 
5. separation anxiety 


How much data do I need for each label? 
https://www.kaggle.com/mmoreaux/audio-cats-and-dogs has 164 8-10sec cat clips and 113 5-6sec dog clips (only two labels). Sampled at 16 kHz 

Kaggle Environmental Sound Classification has 50 5 sec samples for each class. Sampled at 16kHz 

UrbanSound8K has 8800 4sec samples for 10 classes

GOAL: 50 5 sec samples for each label. ~21 min of data total (4min/label) at 44.1 kHz and 64 kbps (i think). Would be about 20-30 MB total 

The main problems are data noisiness and variable time durations. Should be able to split a long clip into multiple smaller clips. 

To address noise, whenever sampling data, also take a sample of the noise (sound when Gia isn't making a sound, e.g. when she's playing outside, in the house, etc.) 

## Preprocessing: 
- Noisereduce python library uses spectral gating to suppress noise 

### Spectral gating: 
Use fourier transforms to get the frequency spectrum (pure tones) of the noise
For each short segment of sound in the signal, take the fourier transform. If any of pure tones (or freq spikes) are not significantly higher (based on sensitivity param) than their average in the noise, reduce them (it means they’re probably still noise). 
Apply time smoothing (so gain across bands moves gradually) and frequency smoothing (so one frequency isn’t reduced/boosted in isolated) 
Apply inverse fourier transform to get the new output signal with reduced noise 
 
Fourier transforms is a way to represent a wave in its constituent pure freqs. This new form is a sum of sines/cosines (the pure frequencies that are the constituents of the observed waveform) 

Attributes of sound: 
Amplitude — Loudness of the sound
Frequency — The pitch of the sound
Timbre — Quality of the sound or the identity of the sound (e.g. the Sound difference between a piano and a violin) → how is this quantified? 


## Feature extraction: 

Fourier transform: snapshot of a waveform represented as its decomposed frequencies. 

Discrete fourier transform: The fourier transform equation is originally bounded from -inf to inf. In computing, audio typically combines in discrete samples, so it’s more reasonable to use the discrete (summation) FT rather than the continuous integral 

Fast fourier transform: A fast way to compute DFT and iDFT. Creates a sparse fourier representation of the waveform (mostly zeros with some spikes), so it’s quick and can be used for compression. For n samples, DFT takes O(N^2), but FFT is O(N logN). It’s a divide and conquer algo, so works well with ^2 sample sizes. 

Chroma stft: Short time fourier transform 
Tracks how the frequency spectrum changes over time. Take the FFT for overlapping windows over time. Can adjust the window size and hop. 
The window size is the a sliding window over the wave form on which the FFT is calculated. Since the delta(t) of the time graph = delta(f) of the frequency graph (bidirectional bandwidth), as the window size increases, it increases frequency resolution, but decreases time precision. The window size should be a power of two for divide and conquer FFT. 
Hop is how much the window size translates. Small hop = more overlap, so smoother output, but more compute time. 
Chromagram is a representation of the STFT that aggregates the pitch info into one of 12 classes (12 keys on a piano). This gives pitch info independent of timbre. Yields 12 features. 

Mel frequency: another way of measuring a waveform instead of Hz. Problem with Hz is that as freq increases, need to be at farther difference to perceive the pitch change. Mel somehow accounts for this; it better incorporates how humans perceive pitch. The equation is a log relationship 

MFCC: Mel Frequency Cepstrum Coefficients (link) 
Cepstrum is a transformation on the frequency spectrum generated by FT. Cepstrum = FT(log(FT(waveform) in mels)). The cepstrum coefficients describe the cepstrum. This is used as a feature set because these coefficients describe in the cepstrum graph, there’s a peak wherever there’s a periodic element in the og signal. The domain is called the quefrency domain. Since mel frequencies better represent how humans perceive sound, overall the MFCCs represent sounds as the shape of the vocal tract that generated it. There are 40 coeffs that are features. 


Melspectrogram: (link) (great explanation of FFT, spectrogram, and mel spectrogram) 
Spectrogram - a picture of sound. It shows frequency over time. Think FFTs overlapped over time. The x axis is time, y axis is frequency (usually log), brightness is amplitude (log amplitude is decibels). 
Compute FFT, convert to mel scale, make a spectrogram of it with mel as the y axis. 


Spectral contrast: 
Classify narrow vs wide frequency band signals. Split signal into 7 frequency sub-bands. Take difference between spectral peak and valley. High contrast means narrow band and vice versa. Calculate this over time. Makes 7 more features when averaged over time. 



## References 

CNN for audio: https://towardsdatascience.com/cnns-for-audio-classification-6244954665ab
Music feature extraction: https://towardsdatascience.com/extract-features-of-music-75a3f9bc265d





