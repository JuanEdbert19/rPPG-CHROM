## How to run
1. Create a virtual environment
2. Activate that virtual environment and select it as your notebook kernel
2. Open main.ipynb and run all cells

NOTE: the benchmarking function is currently commented out since in order to run this function, you will need to have the public video dataset to be available locally. Therefore, only the live webcam demo is available right now. If you want to have the benchmarking function, you would need to download the individual "subject" folders from https://www.kaggle.com/datasets/malekdinarito/ubfc-rppg-dataset?select=subject1 and put them in the same folder as main.ipynb.


rPPG (Remote Photoplethysmography) is a method of measuring heartrate using the imperceptible color changes in the human skin. The idea is hemoglobin absorbs certain lights more, from this we can detect the heart cycles. 

### Traditional Approach
1. ROI tracking and Colour averaging.
	- Use computer vision to mark or bound the face's region of interest (cheek, forehead)
	- In a nutshell, averages the colours of that square patch
	- Measure heartrate by watching out the changes in colour of those patches
	- Input is the pixels of several frames

### Modern Approach
One of the weaknesses from the traditional approach is that it underfits the face into a few boxes. When room lighting changes, the value of a box (average of the pixels in the box) looks like a heartbeat. Modern approach drastically is better at this case because it tracks all 3 colours (R,G,B). 

1. CNN
	- Instead of having a static colour channel averaging approach, the CNN approach usees a more dynamic way to find interactions between pixels. This approach involves multiple working parts including a spatial neural network layer and the main prediction layers.

2. Transformers
	- Very similar with the general LLM transformers architecture. The difference can be found in the input ouput part.
	- Output: instead of softmax and a linear layer, we have a regression layer (mlp)
	  
	  
## Motivation behind this unsupervised rPPG method
At first, I tried doing the modern approach which involves using CNNs. I ran into multiple issues since the open source models uses multiple old version of python libraries. This and other compatibility issues showed up because I'm using a silicon mac. After a while, I decided that its more worth it to pursue the traditional method, especially considering my designated time budget is around 3 hours.

## General Pipeline
1. Facial detection and ROI extraction
	I used google's lightweight facial computer vision model called mediapipe. ROI selection is decided scientifically from existing papers such as https://pmc.ncbi.nlm.nih.gov/articles/PMC8659899/. Based on multiple controlled experiments, it is found that the most accurate results are produced when we only include the left cheek, right cheek, and a portion of the forehead for the ROI.
	
2. Colour Channel Averaging
   There exists several methods to do the colour averaging of the facial ROIs. Some of them includes GREEN, POS, CHROM, and ICA. I decided on using CHROM, which works by projecting the three colour signals (R,G,B) into two chrominance signals. Heartrate is then predicted using the differences between these signals across time. This decision was made based on this paper https://pmc.ncbi.nlm.nih.gov/articles/PMC8659899/, which shows that CHROM is amongst the top performers.

	Additionally, my CHROM implementation was similar to this CHROM implementation on github ( https://github.com/etanak/CHROM_rPPG_implementation/tree/main).

3. Signal filtering and Peak Detection
	Once the raw pulse signal is extracted using CHROM, it still contains unwanted noise from breathing, subtle head movements, and camera flicker. To isolate the actual heartbeat, a **Bandpass Filter** is applied. The filter is configured with boundaries typically between 0.7 Hz and 3.5 Hz, restricting the data to a realistic human heart rate range (roughly 42 to 210 BPM). Finally, to calculate the exact beats per minute, the system applies a Fast Fourier Transform (FFT). Instead of attempting to count individual peaks in a noisy time-domain wave, FFT converts the signal into the frequency domain, allowing the model to simply locate the highest magnitude frequency peak and output it as the dominant heart rate.

## Results
The model is benchmarked using 3 videos chosen randomly from the UBFC-rPPG Dataset (https://www.kaggle.com/datasets/malekdinarito/ubfc-rppg-dataset?select=subject1). 

Subject         | Valid Frames | MAE (BPM)  | MAPE (%)
-----------------------------------------------------------------
subject1      	| 1344 frames        | 4.36 BPM      | 4.00%

subject12       | 1787 frames        | 3.41 BPM      | 5.22%

subject23       | 1757 frames        | 4.93 BPM      | 7.78%

FINAL RESULTS
-----------------------------------------------------------------
Total Mean Absolute Error (MAE):  4.24 BPM

Total Mean Abs Percentage Error:  5.67%
### Analysis 
The benchmark results align with my initial hypothesis about this model's main disadvantage, which is sensitivity to environmental changes. Through looking at each videos individually, "subject23" moves their head a lot. These micro movements momentarily changes the colour of the skin shadows. Therefore, measurements are more unpredictable in these cases.



## Improvements
The next steps would be to explore the use of deep learning for rPPG (CNN and transformers). This would fix the disadvantages of the model in this repo, which is sensitivity to environmental changes. 
