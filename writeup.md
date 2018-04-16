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
[image1]: ./report_images/HOG1.jpg
[image2]: ./output_images/test1.jpg
[image3]: ./output_images/test2.jpg
[image4]: ./output_images/test3.jpg
[image5]: ./output_images/test4.jpg
[image6]: ./output_images/test5.jpg
[image7]: ./output_images/test6.jpg
[image8]: ./output_images/test1_heatmap.jpg
[image9]: ./output_images/test2_heatmap.jpg
[image10]: ./output_images/test3_heatmap.jpg
[image11]: ./output_images/test4_heatmap.jpg
[image12]: ./output_images/test5_heatmap.jpg
[image13]: ./output_images/test6_heatmap.jpg


## [Rubric](https://review.udacity.com/#!/rubrics/513/view) Points
### Here I will consider the rubric points individually and describe how I addressed each point in my implementation.  

---
### Writeup / README

#### 1. Provide a Writeup / README that includes all the rubric points and how you addressed each one.  You can submit your writeup as markdown or pdf.  [Here](https://github.com/udacity/CarND-Vehicle-Detection/blob/master/writeup_template.md) is a template writeup for this project you can use as a guide and a starting point.  

You're reading it!

### Histogram of Oriented Gradients (HOG)

#### 1. Explain how (and identify where in your code) you extracted HOG features from the training images.

The first step was to load all the available data into dataset lists using the "glob" library. This has been done in Cell 2. The implementation of the HOG feature extraction is done in Cell 3.

The initial experiment with HOG was done with the following parameters:
- Orientation = 9
- Pixels per cell = 8
- Cells per block = 2
- Color space = Gray

![alt text][image1]


#### 2. Explain how you settled on your final choice of HOG parameters.

For the final parameter selection I made a series of tests using the same classifier. The selection was then done based on the classifier accuracy, of course, and the processing time.

The first decision to be made was about which colorspace to use and on which channel(s) is the HOG extraction to be done. The results are summarized in the table below. The generation of the table can be seen in cell 11.

| Colorspace | Orient | P/C | C/B | Channel | Extraction time | Training time | Accuracy |
|:---:|:---:|:---:|:---:|:---:|:---:|:---:|:---:|
| RGB | 9 | 8 | 2 | 0 | 13.64s | 6.19s | 0.90 |
| RGB | 9 | 8 | 2 | 1 | 13.70s | 5.50s | 0.91 |
| RGB | 9 | 8 | 2 | 2 | 13.70s | 5.85s | 0.91 |
| RGB | 9 | 8 | 2 | ALL | 36.13s | 17.59s | 0.91 |
| HSV | 9 | 8 | 2 | 0 | 14.73s | 6.03s | 0.90 |
| HSV | 9 | 8 | 2 | 1 | 13.77s | 6.41s | 0.87 |
| HSV | 9 | 8 | 2 | 2 | 13.91s | 5.61s | 0.90 |
| HSV | 9 | 8 | 2 | ALL | 36.80s | 13.11s | 0.95 |
| LUV | 9 | 8 | 2 | 0 | 14.41s | 5.89s | 0.92 |
| HLS | 9 | 8 | 2 | 0 | 14.97s | 6.46s | 0.90 |
| HLS | 9 | 8 | 2 | 1 | 13.95s | 6.18s | 0.89 |
| HLS | 9 | 8 | 2 | 2 | 15.07s | 6.92s | 0.88 |
| HLS | 9 | 8 | 2 | ALL | 36.53s | 13.38s | 0.95 |
| YUV | 9 | 8 | 2 | 0 | 14.54s | 5.66s | 0.92 |
| YUV | 9 | 8 | 2 | 1 | 14.07s | 4.20s | 0.93 |
| YCrCb | 9 | 8 | 2 | 0 | 14.23s | 5.75s | 0.92 |
| YCrCb | 9 | 8 | 2 | 1 | 13.44s | 3.80s | 0.93 |
| YCrCb | 9 | 8 | 2 | 2 | 13.87s | 3.38s | 0.95 |
| YCrCb | 9 | 8 | 2 | ALL | 35.56s | 5.04s | 0.98 |

A few combinations are missing, they triggered an exception during the processing of the data. Based on the table values, we have a winner for the colorspace and channel decision: YCrCb and all channels.

But the next open question is, how does the choice of the orientation, pixels per cells and cells per block plays an impact on the accuracy of the classifier. Code is available in Cell 12.

| Colorspace | Orient | P/C | C/B | Channel | Extraction time | Training time | Accuracy |
|:---:|:---:|:---:|:---:|:---:|:---:|:---:|:---:|
| YCrCb | 8 | 10 | 4 | ALL | 19.88s | 2.87s | 0.99 |
| YCrCb | 8 | 10 | 6 | ALL | 19.58s | 0.67s | 0.99 |
| YCrCb | 8 | 12 | 4 | ALL | 15.58s | 1.13s | 0.99 |
| YCrCb | 10 | 10 | 4 | ALL | 19.22s | 3.60s | 0.99 |
| YCrCb | 10 | 10 | 6 | ALL | 14.99s | 0.93s | 0.99 |
| YCrCb | 10 | 12 | 4 | ALL | 16.98s | 1.56s | 0.99 |


We can, only using a HOG based linear SVM classifier, reach an accuracy of 99%. The most accurate an  time efficient combination of parameters is the following: 
- Orientation = 10
- Pixels per cell = 10
- Cells per block = 6
- HOG channels = ALL
- Colorspace = YCrCb

#### 3. Describe how (and identify where in your code) you trained a classifier using your selected HOG features (and color features if you used them).

The linear SVM training can be found in the Cells 8 and 9. The data was normalized using the standard scaler method of scikit. Default parameters for C and gamma were used at first.

### Sliding Window Search

#### 1. Describe how (and identify where in your code) you implemented a sliding window search.  How did you decide what scales to search and how much to overlap windows?

I decided to split the lower half of the image in two regions, far and near. For these regions the scale is different, as we expect cars in the near region to be bigger that in the far region. An acceptable performance on the test images set was obtained using:
- far region: y ranges from 400 to 550, scale factor 1.2
- near region: y ranges from 550 to 656, scale factor 2.0

The implementation can be found in Cells 17 and 18. 


#### 2. Show some examples of test images to demonstrate how your pipeline is working.  What did you do to optimize the performance of your classifier?

Here is the result on the set of test images:
![alt text][image2]
![alt text][image3]
![alt text][image4]
![alt text][image5]
![alt text][image6]
![alt text][image7]

the pipeline performs well enough in the test images, all vehicles are hit at least once, and there are no false positives. Nevertheless it is necessary to improve this pipeline in order to be robust towards false positives, that will anyways be detected on the video.

#### 3. Overlapping windows

As we can see, for a single car on the image we might get multiple hits. We need to combine those hits in order to have a single object, and it will also help getting a bounding box that best fits. For this purpose we use the heat map approach from the lesson. Code is to be seen in cells 19 and 20. A low threshold for the heatmap was chosen in order to have a positive detection of the car on the image test3.jpg.

![alt text][image8]
![alt text][image9]
![alt text][image10]
![alt text][image11]
![alt text][image12]
![alt text][image13]


### Video Implementation

#### 1. Provide a link to your final video output.  Your pipeline should perform reasonably well on the entire project video (somewhat wobbly or unstable bounding boxes are ok as long as you are identifying the vehicles most of the time with minimal false positives.)
Here's a [link to my video result](./output_videos/project_video_initial.mp4)

The pipeline works pretty well, with a low amount of false positives. However there is a problem with false negatives, the white overtaking car on the right side is not detected for a very long time. In order to improve the situation, and since the classification performance is pretty good, I decided to add some scaled windows for the search. My suspicion is that the white car is no longer detected because the window size in that area is not appropriate. After a few trials, I ended up using:
- hits 2 far region: y ranges from 410 to 466, scale factor 1.2
- hits 2 near region: y ranges from 400 to 515, scale factor 2.0
- hits 1 far region: y ranges from 410 to 500, scale factor 1.2
- hits 1 near region: y ranges from 400 to 564, scale factor 2.0

this combination improves a lot the final result, as can be seen in this video [link to my video result](./output_videos/project_video_version2.mp4)


This output is much better, but we still need to improve the performance a bit. The next possibility would be to keep the heat map in memory instead of re-initializing it at each cycle.
For this purpose we just store in a list the last 5 heatmaps, and compute the average for each cell over the last 5 cycles.

The final result can be seen in this video: [link to my video result](./output_videos/project_video_final.mp4)

The performance looks in my opinion good, the vehicles are indeed detected most of the time and the number of false positives is very low. One of the false positives happens on an oncoming car, which is maybe not to be treated as a false positive. For a few cycles the white car on the right is lost, but since the car is on the overnext lane this should not be a problem.


#### 2. Describe how (and identify where in your code) you implemented some kind of filter for false positives and some method for combining overlapping bounding boxes.

Please refer to 3. Overlapping windows above. I used the heat map method from the class for reducing the amount of false positives.


---

### Discussion

#### 1. Briefly discuss any problems / issues you faced in your implementation of this project.  Where will your pipeline likely fail?  What could you do to make it more robust?

one of the main issues was finding a proper set of scales and windows for the generation of the HOG. The following points could improve in my opinion the functionality of the detection function:

- Use different scales in Y and X, as objects appear to have a different size depending on their lateral displacement compared to the ego car.
- Including another feature set, like the color histogram could improve the performance of the classifier.
- Detecting oncoming vehicles should be a goal, since on a secondary road we do not want to crash on oncoming traffic
- Trying around different classifiers (decision tree for instance) or adjusting the c parameter of the linear SVC

