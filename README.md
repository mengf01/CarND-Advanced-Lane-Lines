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

[image1]: ./output_images/undistortion.jpg "Undistorted"
[image2]: ./output_images/undistortion_test2.jpg "Road Undistorted"
[image3]: ./output_images/thresholding_test2.jpg "Road Thresholded"
[image4]: ./output_images/get_rect_on_straight_lanes.jpg "Transformation"
[image5]: ./output_images/warped_test2.jpg "Road Warped"
[image6]: ./output_images/binary_warped_test2.jpg "Binary Image Warped"
[image7]: ./output_images/sliding_windows_test2.jpg "Sliding Window Example"
[image8]: ./output_images/output_mask_test2.jpg "Out mask"
[image9]: ./output_images/final_image_test2.jpg "Final image"
[video1]: ./project_video_output.mp4 "Video"

### Introduction
#### The project code is in `P2.ipynb` notebook. There are 3 parts:
#### 1, I wrote a draft pipeline function by function to test each of the images in `test_images` folder. The images in this report has been generated during this process.
#### 2, I organized above pipeline to a single class called `Pipeline`. Some refactoring work has been done to make it clean and effective. (I will go over details below based on functions in `Pipeline` class.)
#### 3, The `process_frame` function of `Pipeline`has been used to process each frame image of the input video.

### Camera Calibration

#### 1. Briefly state how you computed the camera matrix and distortion coefficients. Provide an example of a distortion corrected calibration image.

The code for this step is in `Pipeline.calibrate_camera()`.

I start by preparing "object points", which will be the (x, y, z) coordinates of the chessboard corners in the world. Here I am assuming the chessboard is fixed on the (x, y) plane at z=0, such that the object points are the same for each calibration image.  Thus, `objp` is just a replicated array of coordinates, and `objpoints` will be appended with a copy of it every time I successfully detect all chessboard corners in a test image.  `imgpoints` will be appended with the (x, y) pixel position of each of the corners in the image plane with each successful chessboard detection.  

I then used the output `objpoints` and `imgpoints` to compute the camera calibration and distortion coefficients using the `cv2.calibrateCamera()` function.  I applied this distortion correction to the test image using the `cv2.undistort()` function and obtained this result: 

![][image1]

### Pipeline (single images)

#### 1. Provide an example of a distortion-corrected image.

To demonstrate this step,  I applied the distortion correction (using camera calibration parameters) to the test image using the `cv2.undistort()` function and obtained this result: 
![][image2]

#### 2. Describe how (and identify where in your code) you used color transforms, gradients or other methods to create a thresholded binary image.  Provide an example of a binary image result.

The code for this step is in `Pipeline.apply_color_transforms_and_gradients()`.

I used a combination of color and gradient thresholds to generate a binary image.  The color threshold is on `s` channel while the gradient threshold is on `l` channel. Here's an example of my output for this step.

![][image3]

#### 3. Describe how (and identify where in your code) you performed a perspective transform and provide an example of a transformed image.

The code for this step is in `Pipeline.get_perspective_trans_matrices()`.

This function takes inputs from source (`src`) and destination (`dst`) points.  I chose the hardcode the source and destination points in the following manner:

I chose source and destination points from a test image with straight lanes. The four source points are drawn on the image below:

| Source        | Destination   | 
|:-------------:|:-------------:| 
| 578, 460      | 320, 0        | 
| 203, 720      | 320, 720      |
| 1106, 720     | 960, 720      |
| 702, 460      | 960, 0        |
![Select 4 source points][image4]

I verified that my perspective transform was working as expected by drawing the `src` and `dst` points onto the test image and its warped counterpart to verify that the lines appear parallel in the warped image. Warping is done by using `cv2.warpPerspective()`.

![][image5]
![warped binary image][image6]

#### 4. Describe how (and identify where in your code) you identified lane-line pixels and fit their positions with a polynomial?

The code for this step are in the 3 functions below:

`Pipeline.find_lane_pixels()`: I used sliding window method to find lane pixels. First, I found the histogram of points on the warped binary image along x axis and selected the peaks as my starting points for the left and right lanes. Then, I used sliding window upward to constrain lane pixels within each window and at the same time update my window.

`Pipeline.search_around_poly()`: I used cached fitting results with margin to find lane pixels if the fitting in the previous frame is good (I'll explain below).

`Pipeline.fit_polynomial()`:  I fit a 2nd order polynomial to the detected lane pixels (either from 		`find_lane_pixels()` or `search_around_poly()`). I also did quality control with this logic: we should have left and right curvatures in a similar range of magnitude (ratio within `0.05-20`), and we should have left and right curves have the same sign for `a` for `ax^2+bx+c`. Otherwise, we reset to use sliding window for the next frame.

An example of the sliding window output is shown below:
![][image7]

#### 5. Describe how (and identify where in your code) you calculated the radius of curvature of the lane and the position of the vehicle with respect to center.

The code for this step is in `Pipeline.measure_curvature_real()`.

#### 6. Provide an example image of your result plotted back down onto the road such that the lane area is identified clearly.

The code for this step is in `Pipeline.setup_visualization()`.
An example of the detected road region is shown below:
![][image8]
Here is a final output of my result on the test image:
![final result][image9]

---

### Pipeline (video)

#### 1. Provide a link to your final video output.  Your pipeline should perform reasonably well on the entire project video (wobbly lines are ok but no catastrophic failures that would cause the car to drive off the road!).

Here's a [link to my video result](./project_video_output.mp4))

---

### Discussion

#### 1. Briefly discuss any problems / issues you faced in your implementation of this project.  Where will your pipeline likely fail?  What could you do to make it more robust?

The project video output looks good while the pipeline failed to handle the challenge video, especially where the dark background appears. If given time, I could tune further on the color masking part to make it better.