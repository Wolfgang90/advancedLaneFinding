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

[image1]: ./output_images/01_chess_dist.png "distorted chessboard"
[image2]: ./output_images/02_chess_dist.png "undistorted chessboard"
[image3]: ./output_images/03_original.png "original status example picture"
[image4]: ./output_images/04_undistorted.png "undistorted image"
[image12]: ./output_images/12_combined_binary.png "combined binary image"
[image13]: ./output_images/13_src_points.png "source points"
[image14]: ./output_images/14_dst_points.png "destination points"
[image15]: ./output_images/13_warped.png "warped image"
[image16]: ./output_images/14_warped_binary.png "warped binary image"
[image19]: ./output_images/19_identified_lanes_first.png "identified lane lines first"
[image20]: ./output_images/20_identified_lanes_second.png "identified lane lines subsequent"
[image21]: ./output_images/21_retransform.png "retransformed image"
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

In section "1.1 Load images and fit corners" I start by preparing "object points", which will be the (x, y, z) coordinates of the chessboard corners in the world. Here I am assuming the chessboard is fixed on the (x, y) plane at z=0, such that the object points are the same for each calibration image.  Thus, `objp` is just a replicated array of coordinates, and `obj_points` will be appended with a copy of it every time I successfully detect all chessboard corners in a test image.  `img_points` will be appended with the (x, y) pixel position of each of the corners in the image plane with each successful chessboard detection.  

I then used the output `obj_points` and `img_points` to compute the camera calibration and distortion coefficients using the `cv2.calibrateCamera()` function in section "1.2 Calibrate camera".  

Subsequently I applied the distortion correction creating the function `distortion_correction()` using `cv2.undistort()`  and applying it to the test image in section "1.3 Perform distortion correction". I obtained this result: 

Undistorted image

![alt text][image1]

Distorted image

![alt text][image2]

###Pipeline (single images)
All example images of the pipeline will be based on this initial image:
![alt text][image3]

####1. Provide an example of a distortion-corrected image.
Through`distortion_correction()` from section "1.3 Perform distortion correction" I apply the previously determined transformation matrix on all test images in section "2.1 Distortion correction". Here is an example of our undistorted test image:
![alt text][image4]

####2. Describe how (and identify where in your code) you used color transforms, gradients or other methods to create a thresholded binary image.  Provide an example of a binary image result.
I used a combination of color and gradient thresholds to generate a binary image in section "2.2 Create threshholded binary image". In section "2.2.1 Create S-channel based binary image" I apply a threshold based on the S-channel (`RGB_to_HLS()`,`HLS_to_S`,`S_to_thresh`). In section "2.2.2 Create Sobel based binary image" I apply 4 different Sobel thresholds, namely Sobel in x-direction, Sobel in y-direction (both `abs_sobel_thresh()`) , the magnitude of the gradient (`mag_thresh()`) and the direction of the gradient (`dir_threshold()`). Subsequently I combine the thresholds by counting pixels which are either activated in both x- and y-direction Sobel or both in magnitude of the gradient and the direction of the gradient (`sob_to_sobcomb()`). In section "2.2.3 Create overall combined binary image" I finally combine the overall Sobel binary with the s-channel binary, counting the pixel if it is activated in one of the two (`col_sob_comb()`). To summarize the various functions just mentioned as one callable to create the binary, I created the overall function `img_to_thresh_bin()` in section "2.2.4 Create overall function for binary creation". Examples for all stages for all test images can be found in the notebook. Here's our example picture with the final output of this step.

![alt text][image12]

####3. Describe how (and identify where in your code) you performed a perspective transform and provide an example of a transformed image.

The code for my perspective transform includes a function called `warp()`, which can be found in section "2.3 Perform perspective transform".  The `warp()` function takes an image (`img`), as well as source (`src`) and destination (`dst`) points as inputs.  I chose to hardcode the source and destination points in the following manner based on manual identification in an example image:

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

Test image:

![alt text][image13]

Warped image:

![alt text][image14]

If I apply the warp function on our test image, it looks like this:

![alt text][image15]

If I apply the warp function on the binary image of our test image it looks like this:
![alt text][image16]

####4. Describe how (and identify where in your code) you identified lane-line pixels and fit their positions with a polynomial?

I identified and fitted the lane-line pixels in section "2.4 Identify lane-line pixels and fit with polynomial". In order to identify the lane lines, I generated the two funtions `identify_lane_line_first()` and `identify_lane_line_cont()`. `identify_lane_line_first()` searches for the lane lines in the whole image, while `identify_lane_line_cont()` uses the identified lane lanes of the previous image as a starting point for the search. It will become relevant for the video processing in order to minimize computing time.

`identify_lane_line_first()` uses a histogram of the bottom half of the image to determine the starting point for the search of the 2 lanes at the bottom of the image by evaluating the peaks of the histogram (line 25 ff. in respective code cell). Subsequently, I defined windows to start at the bottom of the image and iterate upwards to perform the lane line search upwards the image (line 40 ff. in respective code cell). Afterwards I perform an initial iteration to allocate the windows to the area with most pixels found (line 62 ff. in respective code cell). For detailed documentation of the processing steps, please refer to the in-line code comments. Afterwards I determine the overall direction of the lane lines to be able to correct individual window positions (line 113 ff. in respective code cell). Based on the overall turn-direction I perform a second iteration to correct the window positions (line 144 ff. in respective code cell). If for instance the overall curve structure of a lane is to the left, but one window is shifted to the right compared to the previous image, the window will be positioned in the middle between the previous and next window. The window was probably bisased to the right due to activated pixels on the right not belonging to the lane. By the second iteration these pixels are excluded before we fit the lanes. For detailed documentation please refer to the in-line code comments. Eventually, I used `np.polyfit()` to fit a second order polynomial functions to represent the lane lines, based on the activated pixels in the previously set windows, calculate the x and y values of the lane lines and plot them on the output image (line 226 ff. in respective code cell).

`identify_lane_line_cont()` uses the polynomial fits identified in the previous image as starting point for the lane line search. Pixels are now searched in a margin area around the previous fit function. Subsequently, a line is fitted and plotted to the image based on the activated pixels in the margin area.

The output of the function `identify_lane_line_first()` for our test image is:

![alt text][image19]

If we take the polynomial fits of the previous image as basis, the output of the function `identify_lane_line_cont()` for our test image looks like this:

![alt text][image20]

####5. Describe how (and identify where in your code) you calculated the radius of curvature of the lane and the position of the vehicle with respect to center.

I created 5 functions to perform this and related tasks in section "2.5 Calculate radius of curvature and position of vehicle". `radius_and_position_warped()` defines the right and left curve radius in pixels. `convert_radpos_to_real()` performs a conversion of the radius to meters.`det_position_warped()` determines the offset from the lane center and the distance from the car center to the left and right line. `dconvert_position_to_real()` transforms the offset and the distances to meters. `print_data_to_image()` enables to output the identified values in a frame directly in the image.

####6. Provide an example image of your result plotted back down onto the road such that the lane area is identified clearly.

In section "2.6 Transform lane view back to initial image" I implemented the function `warped_to_real()`, which performs the task. Here is an example of my result on a test image:

![alt text][image21]

---

###Pipeline (video)

####1. Provide a link to your final video output.  Your pipeline should perform reasonably well on the entire project video (wobbly lines are ok but no catastrophic failures that would cause the car to drive off the road!).

In section "3.1 Define class to store previous image values" I designed the class `Line()` to hold the line values of the previous images (up to 5) in instances for the right and left line, to be able to access the values of the previous image to set thresholds if the identified values deviate too strongly from the previous image. Furthermore, the instances will be used to calculate an average of the past 5 values if a lane line was not identified to output in the video.

In section "3.2 Define function to apply processing pipeline on image" I created the full pipeline to be applied on a single image of the video stream in the function `process_image()`. The pipeline consists of the functions defined in section "2 Pipeline (test images)" (line 14 ff. in respective code field). Subsequently, I defined three conditions which result in lines not beeing detected, which are: (1) width of the lane lines is greater than 6 or smaller than 2 meters, (2) lane lines are pointing in different directions and one of the curve radiuses is smaller 1000m, (3) Strong differences in curve radius (> 100m) at small turns (< 1000m for both curve radius) (line 33 ff. in respective code field). If the lane line was identified, the class values are updated accordingly (line 52 ff. in respective code field). Else, the class value for detected lane lines is set to not detected (line 57 ff. in respective code field) and the lane values are updated with the mentioned average values of the previous 5 correctly fitted images (line 62 ff. in respective code field).

In section "3.3 Process video" the class instances for right and left lines are created (2nd code cell of section) and the pipeline of section "3.2 Define function to apply processing pipeline on image" is applied on the video.

Here's a [link to my video result](./project_video.mp4)

---

###Discussion

####1. Briefly discuss any problems / issues you faced in your implementation of this project.  Where will your pipeline likely fail?  What could you do to make it more robust?

The strongest challeng I faced during implementation was, that lines were "getting out of control", meaning, that they suddently moved away from the actual line to something nearby which might look like a line in the thresholded images. To tackle this issue. I introduced the three conditions to eliminate the lane detections which do not seem reasonable and replace them with average values from the previous images as elaborated in the "Pipeline (video)" paragraph before. Also starting off with the much more detailed lane finding approach with the function `identify_lane_line_first()` helps to identify the lanes better in the next image.

My pipeline still struggles in situations were:

1) The road color and the color of the lanes do not have much contrast, e.g. yellow lanes and concrete road

2) There is both shadow and light on the road, with strong contrast, which might be interpreted as lane by the current pipeline

3) Reflections resulting from either water on the road or reflections in the car window, which might be interpreted as lane lines by the image

4) In foggy conditions, when there is not much contrast between the lane lines and the rest of the image

To address these challenges, it would help to further experiment with the creation of the binary image. Other colorspaces and flexibel thresholding (e.g. based on overall image brightness) to receive a better binary image result in the just mentioned scenarios, which is key for the subsequent lane detection.
