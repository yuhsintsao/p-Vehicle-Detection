## Writeup Template
### You can use this file as a template for your writeup if you want to submit it as a markdown file, but feel free to use some other method and submit a pdf if you prefer.

---

**Vehicle Detection Project**

The goals / steps of this project are the following:

* Perform a Histogram of Oriented Gradients (HOG) feature extraction on a labeled training set of images and train a classifier Linear SVM classifier
* Optionally, you can also apply a color transform and append binned color features, as well as histograms of color, to your HOG feature vector. 
* Note: for those first two steps don't forget to normalize your features and randomize a selection for training and testing.
* Implement a sliding-window technique and use your trained classifier to search for vehicles in images.
* Run your pipeline on a video stream (start with the test_video.mp4 and later implement on full project_video.mp4) and create a heat map of recurring detections frame by frame to reject outliers and follow detected vehicles.
* Estimate a bounding box for vehicles detected.

[//]: # (Image References)
[image1]: ./images/car_not_car_hog_set1.png
[image2]: ./images/car_not_car_hog_set2.png
[image3]: ./images/car_not_car_hog_set3.png

[image4]: ./images/slide_windows_01.png
[image5]: ./images/slide_windows_02.png
[image6]: ./images/slide_windows_03.png
[image7]: ./images/slide_windows_04.png
[image8]: ./images/slide_windows_05.png
[image9]: ./images/slide_windows_06.png

[image10]: ./images/heat_01.png
[image11]: ./images/heat_02.png
[image12]: ./images/heat_03.png
[image13]: ./images/heat_04.png
[image14]: ./images/heat_05.png
[image15]: ./images/heat_06.png

[image16]: ./images/out_01.png
[image17]: ./images/out_02.png
[image18]: ./images/out_04.png
[image19]: ./images/out_05.png
[image20]: ./images/out_06.png

[video1]: ./project_video_final.mp4

## [Rubric](https://review.udacity.com/#!/rubrics/513/view) Points
### Here I will consider the rubric points individually and describe how I addressed each point in my implementation.  

---

### Writeup / README

#### 1. Provide a Writeup / README that includes all the rubric points and how you addressed each one.  You can submit your writeup as markdown or pdf.  [Here](https://github.com/udacity/CarND-Vehicle-Detection/blob/master/writeup_template.md) is a template writeup for this project you can use as a guide and a starting point.  

You're reading it!

### Histogram of Oriented Gradients (HOG)

#### 1. Explain how (and identify where in your code) you extracted HOG features from the training images.

The code for this step is contained in the second code cell of the IPython notebook called `project.ipynb`.  

I started by reading in a `vehicle` and `non-vehicle` images. I then explored different color spaces and different `skimage.hog()` parameters (i.e. `orientations`, `pixels_per_cell`, and `cells_per_block`).  I grabbed images from each of the two classes and displayed them to get a feel for what the `skimage.hog()` output looks like.

Here is an example using the `YCrCb` color space and HOG parameters of `orientations=12`, `pixels_per_cell=(8, 8)` and `cells_per_block=(2, 2)`:

![Hog Transformed 01][image1]

Here is an example using the `YCrCb` color space and HOG parameters of `orientations=12`, `pixels_per_cell=(16, 16)` and `cells_per_block=(2, 2)`:

![Hog Transformed 02][image2]

Here is an example using the `YCrCb` color space and HOG parameters of `orientations=8`, `pixels_per_cell=(16, 16)` and `cells_per_block=(2, 2)`:

![Hog Transformed 03][image3]

#### 2. Explain how you settled on your final choice of HOG parameters.

I tried various combinations of three main parameters, i.e. `hog_channel`, `orientations` and `pixels_per_cell`. `orientations` is selected as 8, 11, or 12, and `pixels_per_cell` as 8, 16, or 24. Based on the accuracy rate of the SVM model, I settled on the choice of these three parameters as below:

`hog_channel` was ALL.\\
`orientations` was 12.\\
`pixels_per_cell` was 16.

with HOG cells per block being 2.

#### 3. Describe how (and identify where in your code) you trained a classifier using your selected HOG features (and color features if you used them).

The classification accuracy was used to tune all selected parameters. I trained a linear SVM using 3 kinds of features, i.e. HOG features, binned color features, and color histogram features. I settled on the choice of key parameters as below:
  
The color space was `YCrCb`.\\   
HOG orientations was 12.\\
HOG pixels per cell was 16.\\
HOG cells per block was 2.\\
All hog channels were included.\\
Spatial binning dimensions were (16, 16).\\
Number of histogram bins was 16.

With the above setting, the highest classifiction accuracy rate of the SVM model was achieved. The test accuracy of SVC was 0.9882.

The final feature vector length was 2112.

### Sliding Window Search

#### 1. Describe how (and identify where in your code) you implemented a sliding window search.  How did you decide what scales to search and how much to overlap windows?

I searched window positions starting from x = (385, 400, 415, 430, 445, 460) to x = 600, and set the window's size as (64, 64). The selected region on the image was rescaled by being divided by (1.0, 1.2, 1.5, 1.8, 2.0, 2.2, 2.5). 

The difference between two starting points was 15 pixels which was nearly one forth of the window's size. This implied that the overlapping region was aournd 0.75. I rescaled the selected region up to 7 times with different scales respectively in order to fit different cars' sizes on the images and reduce the number of false positive detections. 

![Sliding Windows 01][image4]
![Sliding Windows 02][image5]
![Sliding Windows 03][image6]
![Sliding Windows 04][image7]
![Sliding Windows 05][image8]
![Sliding Windows 06][image9]

#### 2. Show some examples of test images to demonstrate how your pipeline is working.  What did you do to optimize the performance of your classifier?

I tuned three main parameters to improve the performance of my model: `hog_channel`, `orientations`, and `pixels_per_cell`.

Ultimately I used all hog channels, spatially binned color, and histograms of color in the feature vector, which identified distinguishable characteristics among vehicles and non-vehicles and helped achieving a nice model performance. Here are some example images:

![Pipeline output 01][image16]
![Pipeline output 02][image17]
![Pipeline output 04][image18]
![Pipeline output 05][image19]
![Pipeline output 06][image20]

---

### Video Implementation

#### 1. Provide a link to your final video output.  Your pipeline should perform reasonably well on the entire project video (somewhat wobbly or unstable bounding boxes are ok as long as you are identifying the vehicles most of the time with minimal false positives.)
Here's a [link to my video result](./project_video.mp4)

#### 2. Describe how (and identify where in your code) you implemented some kind of filter for false positives and some method for combining overlapping bounding boxes.

I recorded the positions of positive detections in each frame of the video. For all positive detections, I created a heatmap and then thresholded that map to identify the vehicles' positions. I then used `scipy.ndimage.measurements.label()` to identify individual blobs in the heatmap.  I assumed each blob corresponded to a vehicle. I constructed bounding boxes to cover the area of each blob detected. 

The final video output was created with 2 steps. At the first step, I run the pipeline with three kinds of loop design: (1) recording all blobs detectd with different starting points of y and scales; (2) filtering the detected blobs with a low threshold to avoid false positive outcomes for each setting of the combination of a start point of y and a scale; (3) for a specific scale, recording all blobs detected with different starting points of y. Moreover, to trace each detected vehicle across frames, I recorded the blobs detected in the latest 13 frames. At the second step, I combinded three outputs of the blobs' positions from the previous step and recored the blobs detected in the latest 17 frames.  

Here's an example result showing the heatmap from a series of frames of video, the result of `scipy.ndimage.measurements.label()` and the bounding boxes then overlaid on the last frame of video:

### Here are six frames' corresponding heatmaps and the outputs of `scipy.ndimage.measurements.label()` on the integrated heatmaps :

![Heat map 01][image10]
![Heat map 02][image11]
![Heat map 03][image12]
![Heat map 04][image13]
![Heat map 05][image14]
![Heat map 06][image15]

### Here the resulting bounding boxes are drawn onto the last frame in the series:

![Pipeline output 01][image16]
![Pipeline output 02][image17]
![Pipeline output 04][image18]
![Pipeline output 05][image19]
![Pipeline output 06][image20]

---

### Discussion

#### 1. Briefly discuss any problems / issues you faced in your implementation of this project.  Where will your pipeline likely fail?  What could you do to make it more robust?

I encountered two main problems when doing the project. First, false positive detections popped up every now and then, and made it hard to set the threshold for the heatmap. Second, too many sizes of cars were shown across the video frames, and made it difficult to choose a set of rescaling sizes. If I rescaled the same image too many times with too many different scales, the number of false positive detections increased. That was a dilemma. 

I suggested three ways to make my pipeline more robust. First, increasing the amount of training images for cars and non-cars. This is the most intuitive way to make the model more robust. Second, selecting more features into the feature vector, e.g. including more color spaces like HLS. A longer feature vector might be more representative of the target object. Third, I could try more settings of parameters, record all the outputs separately, and then combine all of them. By setting a new threshold for the combined result, vehicles across frames could be detected accurately and precisely. 









