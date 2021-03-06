**Vehicle Detection Project**

The goals / steps of this project are the following:

* Perform a Histogram of Oriented Gradients (HOG) feature extraction on a labeled training set of images and train a classifier Linear SVM classifier
* Optionally, you can also apply a color transform and append binned color features, as well as histograms of color, to your HOG feature vector. 
* Note: for those first two steps don't forget to normalize your features and randomize a selection for training and testing.
* Implement a sliding-window technique and use your trained classifier to search for vehicles in images.
* Run your pipeline on a video stream (start with the test_video.mp4 and later implement on full project_video.mp4) and create a heat map of recurring detections frame by frame to reject outliers and follow detected vehicles.
* Estimate a bounding box for vehicles detected.

[//]: # (Image References)
[image1]: ./output_images/data_example.png
[image2]: ./output_images/falsepositvie1.png
[image3]: ./output_images/falsepositvie2.png
[image4]: ./output_images/hog_example.png
[image5]: ./output_images/hog_subsample1.png
[image6]: ./output_images/hog_subsample2.png
[image7]: ./output_images/hog_subsample3.png
[image8]: ./output_images/hog_subsample4.png
[image9]: ./output_images/hog_subsample5.png
[image10]: ./output_images/hog_subsample6.png
[image11]: ./output_images/scan_roi.png
[image12]: ./output_images/sliding_detect_example1.png
[image13]: ./output_images/sliding_detect_example2.png
[image14]: ./output_images/sliding_detect_example2.png
[image15]: ./output_images/sliding_detect_example2.png

[video1]: ./processed_project_video.mp4

## [Rubric](https://review.udacity.com/#!/rubrics/513/view) Points
### Here I will consider the rubric points individually and describe how I addressed each point in my implementation.  

---
### Writeup / README

**All code for project and results are included in Vehicle_Detection_and_Tracking.html/Vehicle_Detection_and_Tracking.ipynb**

### Histogram of Oriented Gradients (HOG)

#### 1. Explain how (and identify where in your code) you extracted HOG features from the training images.

The code for this step is contained in these functions `get_hog_features`,`bin_spatial`,`color_hist`,`image_features_fun`,`extract_features`.  

I started by reading in all the `vehicle` and `non-vehicle` images.  Here is an example of one of each of the `vehicle` and `non-vehicle` classes:

![alt text][image1]

I then explored different color spaces and different `skimage.hog()` parameters (`orientations`, `pixels_per_cell`, and `cells_per_block`).  I grabbed random images from each of the two classes and displayed them to get a feel for what the `skimage.hog()` output looks like.

Here is an example using the cars/non-cars image and HOG parameters of `orientations=8`, `pixels_per_cell=(8, 8)` and `cells_per_block=(2, 2)`:


![alt text][image4]

#### 2. Explain how you settled on your final choice of HOG parameters.

I tried various combinations of parameters and test with svc classifier found that the LUV color space and hog channel 0 made better prediction accuracy in detection pipeline! The hog feature extraction speed showed below:

`color space:  RGB  HOG channel: 0
1.65 Seconds to extract featrues
color space:  RGB  HOG channel: 1
0.99 Seconds to extract featrues
color space:  RGB  HOG channel: 2
0.91 Seconds to extract featrues
color space:  HSV  HOG channel: 0
0.93 Seconds to extract featrues
color space:  HSV  HOG channel: 1
0.93 Seconds to extract featrues
color space:  HSV  HOG channel: 2
0.92 Seconds to extract featrues
color space:  LUV  HOG channel: 0
1.06 Seconds to extract featrues
color space:  LUV  HOG channel: 1
0.94 Seconds to extract featrues
color space:  LUV  HOG channel: 2
0.93 Seconds to extract featrues
color space:  HLS  HOG channel: 0
0.91 Seconds to extract featrues
color space:  HLS  HOG channel: 1
0.93 Seconds to extract featrues
color space:  HLS  HOG channel: 2
0.92 Seconds to extract featrues
color space:  YUV  HOG channel: 0
0.96 Seconds to extract featrues
color space:  YUV  HOG channel: 1
0.92 Seconds to extract featrues
color space:  YUV  HOG channel: 2
0.92 Seconds to extract featrues
color space:  YCrCb  HOG channel: 0
0.93 Seconds to extract featrues
color space:  YCrCb  HOG channel: 1
0.92 Seconds to extract featrues
color space:  YCrCb  HOG channel: 2
0.93 Seconds to extract featrues`

But even the fastest feature extraction of color space and hog channel did not make good performence in detection pipeline.

#### 3. Describe how (and identify where in your code) you trained a classifier using your selected HOG features (and color features if you used them).

I nomalized train data features by the method sklearn.StandardScaler(). The data is splitted into thaining and testing subsets using train_test_split(80% and 20%). I use combined features of Spatial features,Histogram features,HOG features and with other parameters settings:

`color_space = 'LUV' # Can be RGB, HSV, LUV, HLS, YUV, YCrCb
orient = 8  # HOG orientations
pix_per_cell = 8 # HOG pixels per cell
cell_per_block = 2 # HOG cells per block
hog_channel = 0 # Can be 0, 1, 2, or "ALL"
spatial_size = (16, 16) # Spatial binning dimensions
hist_bins = 32    # Number of histogram bins
spatial_feat = True # Spatial features on or off
hist_feat = True # Histogram features on or off
hog_feat = True # HOG features on or off`

The length of feture vector is 2432!

### Sliding Window Search

#### 1. Describe how you implemented a sliding window search.  How did you decide what scales to search and how much to overlap windows?

I decide search cars in some ROI area of image, like this:

![alt text][image11]

The code is following:

`windows = slide_window(image, x_start_stop=[800, 1280], y_start_stop=[400, 650], 
                    xy_window=(128, 128), xy_overlap=(0.85, 0.85))
windows += slide_window(image, x_start_stop=[0, 600], y_start_stop=[400, 650], 
                    xy_window=(128, 128), xy_overlap=(0.85, 0.85))
window_img = draw_boxes(image, windows, color=(0, 0, 255), thick=6) 
windows = slide_window(image, x_start_stop=[400, 1000], y_start_stop=[400, 500], 
                    xy_window=(64, 64), xy_overlap=(0.75, 0.75))
window_img = draw_boxes(window_img, windows, color=(0, 255, 0), thick=6) `

The green area use small scale windows and the red area use bigger scale windows, because cars object in the middle of image are far cars which are small, left and right side of image are nearby cars which are bigger in pixels.

#### 2. Show some examples of test images to demonstrate how your pipeline is working.  What did you do to optimize the performance of your classifier?

Ultimately I searched on two scales using LUV 0-channel HOG features plus spatially binned color and histograms of color in the feature vector, which provided a nice result.  Here are some example images:

![alt text][image5] ![alt text][image6]
---

### Video Implementation

#### 1. Provide a link to your final video output.  Your pipeline should perform reasonably well on the entire project video (somewhat wobbly or unstable bounding boxes are ok as long as you are identifying the vehicles most of the time with minimal false positives.)
Here's a [link to my video result](./processed_project_video.mp4)


#### 2. Describe how you implemented some kind of filter for false positives and some method for combining overlapping bounding boxes.

I recorded the positions of positive detections in each frame of the video.  From the positive detections I created a heatmap and then thresholded that map to identify vehicle positions.  I then used `scipy.ndimage.measurements.label()` to identify individual blobs in the heatmap.  I then assumed each blob corresponded to a vehicle.  I constructed bounding boxes to cover the area of each blob detected.  

Here's an example result showing the heatmap from a series of frames of video, the result of `scipy.ndimage.measurements.label()` and the bounding boxes then overlaid on the last frame of video:

### Here are two frames and their corresponding heatmaps and output of `scipy.ndimage.measurements.label()`:

![alt text][image2]

### Here the resulting bounding boxes are drawn onto the last frame in the series:
![alt text][image5]

---

### Discussion

#### 1. Briefly discuss any problems / issues you faced in your implementation of this project.  Where will your pipeline likely fail?  What could you do to make it more robust?

In this project,I use the HOG feature extraction, svm classifier to predtict car object , hog-susampling to speed up feature extraction, heatmap filtering and labeling to remove false positive boxes. 

The todo list include following:
1. Use deep learning detect model to classify the car objects.
2. Caculate the distance between the detected cars and camera.
3. Transform project to C++ code and made some code  optimize to speed up pipeline to get 10fps performance.

### Reference

My code refer to code from this github https://github.com/NikolasEnt/Vehicle-Detection-and-Tracking which was helpful for me to complete my porject , thanks.
