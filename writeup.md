##Writeup Template
###You can use this file as a template for your writeup if you want to submit it as a markdown file, but feel free to use some other method and submit a pdf if you prefer.

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
[image1]: ./examples/car_not_car.png
[image2]: ./examples/HOG_example.png
[image3]: ./examples/sliding_windows.jpg
[image4]: ./examples/sliding_window.png
[image5]: ./examples/bboxes_and_heat.png
[image6]: ./examples/labels_map.png
[image7]: ./examples/output_bboxes.png
[video1]: ./project_video.mp4

## [Rubric](https://review.udacity.com/#!/rubrics/513/view) Points
### Here I will consider the rubric points individually and describe how I addressed each point in my implementation.  

---
### Writeup / README

#### 1. Provide a Writeup / README that includes all the rubric points and how you addressed each one.  You can submit your writeup as markdown or pdf.  [Here](https://github.com/udacity/CarND-Vehicle-Detection/blob/master/writeup_template.md) is a template writeup for this project you can use as a guide and a starting point.  

This project implementation closely followed the instructions and code from the Ryan's Q/A [video](https://www.youtube.com/watch?v=P2zwrTM8ueA). The code is in the jupyter notebook file named `vehicle_detection.ipynb`

### Histogram of Oriented Gradients (HOG)

#### 1. Explain how (and identify where in your code) you extracted HOG features from the training images.

The feature extraction code is in section 2 of the notebook. HOG features code is defined by function `get_hog_features`, which is used in `extract_features` (for processing multiple images together) and `single_img_features` (for processing single image).

I started by reading in all the `vehicle` and `non-vehicle` images.  Here is an example of one of each of the `vehicle` and `non-vehicle` classes:

![alt text][image1]

I then explored different color spaces and different `skimage.hog()` parameters (`orientations`, `pixels_per_cell`, and `cells_per_block`).  I grabbed random images from each of the two classes and displayed them to get a feel for what the `skimage.hog()` output looks like.

Here is an example using the `YCrCb` color space and HOG parameters of `orientations=9`, `pixels_per_cell=(8, 8)` and `cells_per_block=(2, 2)`:

![alt text][image2]

#### 2. Explain how you settled on your final choice of HOG parameters.

I tried various combinations of parameters starting with the values recommended by Ryan in the QnA video. And I've settled on the following parameter values:

````````
color_space = 'YCrCb' # can be RGB, HSV, LUV, HLS, YUV, YCrCb
orient = 9 
pix_per_cell = 8
cell_per_block = 2
hog_channel = 'ALL' # 0, 1, 2 or 'ALL'
````````


#### 3. Describe how (and identify where in your code) you trained a classifier using your selected HOG features (and color features if you used them).

The code in training the classifier is in section 2.3. I've used LinearSVC from `sklearn`. In addition, I've also included color histogram and spatial binning in the feature vector used for training. 

### Sliding Window Search

#### 1. Describe how (and identify where in your code) you implemented a sliding window search.  How did you decide what scales to search and how much to overlap windows?

Sliding window search is defined in section 3 of the notebook. In section 3.1, I tried a few different windows sizes (64x64, 96x96, 128x128). And 96x96 gives me the best results. I used window overlap of 50%. Also instead of searching the whole image, I added y_start_stop to only search the vertical area where I expect cars to appear (not in the sky or on the trees).


#### 2. Show some examples of test images to demonstrate how your pipeline is working.  What did you do to optimize the performance of your classifier?

Following image is the result of search with a window size of 96x96 and 50% overlap.

![alt text][image4]
---

### Video Implementation

#### 1. Provide a link to your final video output.  Your pipeline should perform reasonably well on the entire project video (somewhat wobbly or unstable bounding boxes are ok as long as you are identifying the vehicles most of the time with minimal false positives.)
Here's a [link to my video result](https://youtu.be/DINRPTuetE4)


#### 2. Describe how (and identify where in your code) you implemented some kind of filter for false positives and some method for combining overlapping bounding boxes.

The code is in section 3.4 in the notebook. I recorded the positions of positive detections in each frame of the video.  From the positive detections I created a heatmap and then thresholded that map to identify vehicle positions.  I then used `scipy.ndimage.measurements.label()` to identify individual blobs in the heatmap.  I then assumed each blob corresponded to a vehicle.  I constructed bounding boxes to cover the area of each blob detected.  

Here's an example result showing the heatmap from a series of frames of video, the result of `scipy.ndimage.measurements.label()` and the bounding boxes then overlaid on the last frame of video:

To smooth out the detection and make the bounding boxes less jumpy. I used a queue of size 10 to accumulate the heatmaps of past 10 frames and used a higher threshold of 8. 

### Here are the bounding boxes and their corresponding heatmaps for the 6 test images:

![alt text][image5]

In `process_image`, I tried using different scales (1, 1.5 and 2) and combine the heatmap and use a higher threshold. But it didn't result in less false positives and it runs much slower. So I switched back to use just one scale (1.5, the one that works the best).


---

### Discussion

#### 1. Briefly discuss any problems / issues you faced in your implementation of this project.  Where will your pipeline likely fail?  What could you do to make it more robust?

1. The pipeline sometime is detecting cars from the upcoming traffic. I thought about restricting the search area to the right half of the screen. But then the pipeline won't work if the car is driving on the right lanes as it might fail to detect the cars on the left lanes.

2. Even using the queue of size 10 trying to smooth out the bounding boxes, I still noticed the bounding box sometimes would disappear and reappear again. I tried various window sizes, different queue sizes and thresholds, adjust the y-axis start and stop position, etc. But haven't been able to fix it.

