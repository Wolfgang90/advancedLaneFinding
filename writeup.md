**Advanced Lane Finding Project**

The goals / steps of this project are the following:

* Compute the camera calibration matrix and distortion coefficients given a set of chessboard images.
* Apply a distortion correction to raw images.
* Use color transforms, gradients, etc., to create a thresholded binary image.
* Apply a perspective transform to rectify binary image ("birds-eye view").
* Detect lane pixels and fit to find the lane boundary.
* Determine the curvature of the lane and vehicle position with respect to center.
* Warp the detected lane boundaries back onto the original image.
* Output visual display of the lane boundaries and numerical estimation of lane curvature and vehicle position.

[//]: # (Image References)

[image1]: ./output_images/ "distorted chessboard"
[image2]: ./output_images/ "undistorted chessboard"
[image3]: ./output_images/01_original.png "Original status example picture"
[image4]: ./output_images/02_undistorted.png "Undistorted image"
[image12]: ./output_images/10_combined_binary.png ""
[image13]: ./output_images/11_warped.png ""
[image14]: ./output_images/12_warped_binary.png ""
[image15]: ./output_images/13_identified_lanes_first.png ""
[image16]: ./output_images/14_identified_lanes_second.png ""
[image17]: ./output_images/15_retransform.png ""
[video1]: ./project_video_annotated.mp4 "Video"

All code can be found in the jupyter notebook "./advanced_lane_finding.ipynb"

## [Rubric](https://review.udacity.com/#!/rubrics/571/view) Points
###Here I will consider the rubric points individually and describe how I addressed each point in my implementation.  

---
###Writeup / README

####1. Provide a Writeup / README that includes all the rubric points and how you addressed each one.

You're reading it!
###Camera Calibration

####1. Briefly state how you computed the camera matrix and distortion coefficients. Provide an example of a distortion corrected calibration image.

The code for this step is contained in section "1 Camera calibration" of the IPython notebook.  

I start by preparing "object points", which will be the (x, y, z) coordinates of the chessboard corners in the world. Here I am assuming the chessboard is fixed on the (x, y) plane at z=0, such that the object points are the same for each calibration image.  Thus, `objp` is just a replicated array of coordinates, and `obj_points` will be appended with a copy of it every time I successfully detect all chessboard corners in a test image.  `img_points` will be appended with the (x, y) pixel position of each of the corners in the image plane with each successful chessboard detection.  

I then used the output `obj_points` and `img_points` to compute the camera calibration and distortion coefficients using the `cv2.calibrateCamera()` function.  I applied this distortion correction to the test image using the `cv2.undistort()` function and obtained this result: 


Undistorted image

![alt text][image1]

Distorted image

![alt text][image2]

###Pipeline (single images)
All example images of the pipeline will be based on this initial image:
![alt text][image3]

####1. Provide an example of a distortion-corrected image.
In section "2.1 Distortion correction" of the IPython notebook I defined the function "distortion correction", which uses the `cv2.undistort()` function to apply the previously determined transformation matrix and applied it on all test images. Here is an example of our undistorted test image:
![alt text][image4]

####2. Describe how (and identify where in your code) you used color transforms, gradients or other methods to create a thresholded binary image.  Provide an example of a binary image result.
I used a combination of color and gradient thresholds to generate a binary image (section "2.2 Create threshholded binary image" in IPython notebook). First I provide all hyperparameters for the thresholding. In section "2.2.1 Create S-channel based binary image" I apply a threshold based on the S-channel (`RGB_to_HLS()`,`HLS_to_S`,`S_to_thresh`). In section "2.2.2 Create Sobel based binary image" I apply 4 different Sobel thresholds, namely Sobel in x-direction, Sobel in y-direction (both `abs_sobel_thresh()`) , the magnitude of the gradient (`mag_thresh()`) and the direction of the gradient (`dir_threshold()`). Subsequently I combine the thresholds by counting pixels which are either activated in both x- and y-direction Sobel or both in magnitude of the gradient and the direction of the gradient (`sob_to_sobcomb()`). Finally, I combine the overall Sobel binary with the s-channel binary counting the pixel if it is activated in one of the two (`col_sob_comb()`). To summarize the various functions just mentioned as one callable to create the binary, I created the overall function `img_to_thresh_bin()`. Examples for all stages for all test images can be found in the notebook. Here's our example picture with the final output of this step.

![alt text][image12]

####3. Describe how (and identify where in your code) you performed a perspective transform and provide an example of a transformed image.

The code for my perspective transform includes a function called `warp()`, which can be found in section "2.3 Perform perspective transform" in the IPython notebook).  The `warp()` function takes as inputs an image (`img`), as well as source (`src`) and destination (`dst`) points.  I chose the hardcode the source and destination points in the following manner based on manual identification in an example image:

```
src = np.float32(
    [[255,688],
     [1051,688],
     [595,452],
     [686,452]])

# Four desired coordinates
dst = np.float32(
    [[360,720],
     [940,720],
     [360,0],
     [940,0]])

```

I verified that my perspective transform was working as expected by drawing the `src` and `dst` points onto a test image and its warped counterpart to verify that the lines appear parallel in the warped image.

![alt text][###]

If I apply the warp function on our test image, it looks like this:

![alt text][image13]

If I apply the warp function on our binary test image it looks like this:
![alt text][image14]

####4. Describe how (and identify where in your code) you identified lane-line pixels and fit their positions with a polynomial?

I identified and fitted the lane-line pixels in section "2.4 Identify lane-line pixels and fit with polynomial". In order to identify the lane lines I generated the two funtions `identify_lane_line_first()` and `identify_lane_line_cont()`. `identify_lane_line_first()` searches the whole image for lane lines, while `identify_lane_line_cont()` uses the identified lane lanes of the previous image as a starting point for the search. It will become relevant for the video processing in order to minimize computing time.





Then I did some other stuff and fit my lane lines with a 2nd order polynomial kinda like this:

![alt text][image5]

####5. Describe how (and identify where in your code) you calculated the radius of curvature of the lane and the position of the vehicle with respect to center.

I did this in lines # through # in my code in `my_other_file.py`

####6. Provide an example image of your result plotted back down onto the road such that the lane area is identified clearly.

I implemented this step in lines # through # in my code in `yet_another_file.py` in the function `map_lane()`.  Here is an example of my result on a test image:

![alt text][image6]

---

###Pipeline (video)

####1. Provide a link to your final video output.  Your pipeline should perform reasonably well on the entire project video (wobbly lines are ok but no catastrophic failures that would cause the car to drive off the road!).

Here's a [link to my video result](./project_video.mp4)

---

###Discussion

####1. Briefly discuss any problems / issues you faced in your implementation of this project.  Where will your pipeline likely fail?  What could you do to make it more robust?

Here I'll talk about the approach I took, what techniques I used, what worked and why, where the pipeline might fail and how I might improve it if I were going to pursue this project further.  

