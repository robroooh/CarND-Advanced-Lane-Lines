## Advanced Lane Finding Project

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

[chess]: ./report_images/chess_calibration.jpg "Chessboard image for camera calibration"
[chess_corner]: ./report_images/chess_corners.png "Chessboard image for camera calibration with detected corners"
[chess_undist]: ./report_images/chess_undist.png "Undistorted Chessboard image, result for camera calibration"
[ori_undist]: ./report_images/original_undist.png "Original test images and Undistorted images"
[binary]: ./report_images/binary.png "Original test image and binary"
[warp]: ./report_images/warp.png "Original test image and Warped"
[lane_detection]: ./report_images/lane_detection.png "lane detection"
[lane_detect]: ./report_images/lane_detect.png "lane detection"

[image1]: ./examples/undistort_output.png "Undistorted"
[image2]: ./test_images/test1.jpg "Road Transformed"
[image3]: ./examples/binary_combo_example.jpg "Binary Example"
[image4]: ./examples/warped_straight_lines.jpg "Warp Example"
[image5]: ./examples/color_fit_lines.jpg "Fit Visual"
[image6]: ./examples/example_output.jpg "Output"
[video1]: ./project_video.mp4 "Video"

**Note that all the code references here are in the `examples.ipynb`. For the video pipelining I just copy all the functions to the new notebook `video_pipeline.ipynb`**

## [Rubric](https://review.udacity.com/#!/rubrics/571/view) Points
### Here I will consider the rubric points individually and describe how I addressed each point in my implementation.  

---
### Writeup / README

#### 1. Provide a Writeup / README that includes all the rubric points and how you addressed each one.  You can submit your writeup as markdown or pdf.  [Here](https://github.com/udacity/CarND-Advanced-Lane-Lines/blob/master/writeup_template.md) is a template writeup for this project you can use as a guide and a starting point.  

You're reading it!
### Camera Calibration

#### 1. Briefly state how you computed the camera matrix and distortion coefficients. Provide an example of a distortion corrected calibration image.

The code for this step is contained in the first & second code cell of the IPython notebook located in `./examples/example.ipynb`.  

I start by preparing "object points", which will be the (x, y, z) coordinates of the chessboard corners in the world. Here I am assuming the chessboard is fixed on the (x, y) plane at z=0, such that the object points are the same for each calibration image.  Thus, `objp` is just a replicated array of coordinates, and `objpoints` will be appended with a copy of it every time I successfully detect all chessboard corners in a test image.  `imgpoints` will be appended with the (x, y) pixel position of each of the corners in the image plane with each successful chessboard detection.  

I then used the output `objpoints` and `imgpoints` to compute the camera calibration and distortion coefficients using the `cv2.calibrateCamera()` function.  I applied this distortion correction to the test image using the `cv2.undistort()` function and obtained this result: 

Actually I wrote the functions to wrap those operations above which are 
 - `get_chessboard_point()`
 - `calibrate_camera()`
 - `undistort()`

The results and step from chessboard camera calibration can be seen in the following points

**Chessboard image for camera calibration**
![alt text][chess]

**Chessboard image for camera calibration with detected corners**
![alt text][chess_corner]

**Undistorted Chessboard image, result for camera calibration**
![alt text][chess_undist]

### Pipeline (single images)

#### 1. Provide an example of a distortion-corrected image.
To demonstrate this step, I will describe how I apply the distortion correction to one of the test images:

In order to do distortion correction, the camera calibration needs to be done in order to obtain the camera matrix and distance matrix
then we can use OpenCV's convenient function to undistort the image which is called `cv2.undistort()` where there are required parameters which are
images, camera matrix, and distance matrix. The original images and the distorted images can be seen as followings
![alt text][ori_undist]

Actually I wrote the functions to wrap the distortion-correction in the function called 
 - `undistort()`

#### 2. Describe how (and identify where in your code) you used color transforms, gradients or other methods to create a thresholded binary image.  Provide an example of a binary image result.
I used a combination of sobelx & S color threshold in HSL colorspace to generate a binary image (thresholding steps at the Third part in the function named `pipeline()` in `./examples/example.ipynb`).  Here's an example of my output for this step.

![alt text][binary]

#### 3. Describe how (and identify where in your code) you performed a perspective transform and provide an example of a transformed image.

The code for my perspective transform includes a function called `warped_bw()`, which appears in the fourth part of the file `./examples/example.ipynb`.  The `warped_bw()` function takes as inputs an image (`img`). There is no need to supply `src` and `dst` since the method is hardcoded with pre-defined trial-and-error src and destination. I chose the hardcode the source and destination points in the following manner (result from trial&error):


| Source        | Destination   | 
|:-------------:|:-------------:| 
| 585, 460      | 320, 0        | 
| 200, 720      | 320, 720      |
| 1130, 720     | 960, 720      |
| 695, 460      | 960, 0        |

I verified that my perspective transform was working as expected by choosing a straight lane image and warp to see that the line should be parallel and not curve to any directions. (note: the line is not perfectly parallel but it is enough to give correct result) 

![alt text][warp]

#### 4. Describe how (and identify where in your code) you identified lane-line pixels and fit their positions with a polynomial?

I identify the laneline using the histrogram of the bottom half image then start drawing windows on the 2 peaks of them and try to move windows to where value exists most.
The code for my laneline detection actually resides in a function `lane_detection()` in the 5th part of `./examples/example.ipynb`.
I just fit the 2nd order polynomial to parts where binary threshold is detected using `np.polyfit()`


![alt text][lane_detection]

#### 5. Describe how (and identify where in your code) you calculated the radius of curvature of the lane and the position of the vehicle with respect to center.

I just figure out meter per pixel value using the geometry udacity give (ex. the width of lane is 3.7m) then try to fit the line in meter space then find the radius of curvature according to http://www.intmath.com/applications-differentiation/8-radius-curvature.php

to find the position of the vehicle with respect to center is just finding the difference between the center of an image and center of lane which can be done on meter space

I wrote the function for creating radius of curvature and center offset on the 6th part of `./examples/example.ipynb`. The function is called get_roc()

#### 6. Provide an example image of your result plotted back down onto the road such that the lane area is identified clearly.

I implemented this step in the 7th part of `example.ipynb` in the function `plot_lane_boundaries()`.  Here is an example of my result on a test image:

![alt text][lane_detect]

---

### Pipeline (video)

#### 1. Provide a link to your final video output.  Your pipeline should perform reasonably well on the entire project video (wobbly lines are ok but no catastrophic failures that would cause the car to drive off the road!).

Here's a [link to my video result](./out_project_video.mp4)

---

### Discussion

#### 1. Briefly discuss any problems / issues you faced in your implementation of this project.  Where will your pipeline likely fail?  What could you do to make it more robust?

I would like to break down into bullet points
 - thresholded binary image can be improved by further adjusting the parameters to detect *only* lanelines. using these current values are okay, but it is noisy 
 and not robust for various types of surface which causes problems for detecting lane lines in the challenges video
 - a much better job on perspective transform can be done to make the photo symmetrically perfect. The current method is 90% reliable, but some parts of it are distorted or blurry which is a data loss which directly affects the radius of curvature, offset, and easiness of detecting lanelines.
 - The histrogram is calculated for every frame which is efficient, I believe it can be done only once and use that value only for the first frame and latter frame can use the windows location inferred by the earlier frame
 - center lane offset can be reliable only in the case that the camera is perfectly aligned in the center of the car.
 - line is not smooth(jittery), I believe this can be solve using the last n average of the polynomial coefficients

