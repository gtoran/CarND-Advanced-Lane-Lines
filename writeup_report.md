## Writeup Template
### You can use this file as a template for your writeup if you want to submit it as a markdown file, but feel free to use some other method and submit a pdf if you prefer.

---

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

## [Rubric](https://review.udacity.com/#!/rubrics/571/view) Points
### Here I will consider the rubric points individually and describe how I addressed each point in my implementation.  

---
### Writeup / README

#### 1. Provide a Writeup / README that includes all the rubric points and how you addressed each one.  You can submit your writeup as markdown or pdf.  [Here](https://github.com/udacity/CarND-Advanced-Lane-Lines/blob/master/writeup_template.md) is a template writeup for this project you can use as a guide and a starting point.  

You're reading it!

### Camera Calibration

#### 1. Briefly state how you computed the camera matrix and distortion coefficients. Provide an example of a distortion corrected calibration image.

The code for this step is contained in the first code cell of the IPython notebook located in "./advanced-lane-lines.ipynb".

I start by preparing "object points", which will be the (x, y, z) coordinates of the chessboard corners in the world. Here I am assuming the chessboard is fixed on the (x, y) plane at z=0, such that the object points are the same for each calibration image.  Thus, `objp` is just a replicated array of coordinates, and `objpoints` will be appended with a copy of it every time I successfully detect all chessboard corners in a test image.  `imgpoints` will be appended with the (x, y) pixel position of each of the corners in the image plane with each successful chessboard detection.  

I then used the output `objpoints` and `imgpoints` to compute the camera calibration and distortion coefficients using the `cv2.calibrateCamera()` function.  I applied this distortion correction to the test image using the `cv2.undistort()` function and obtained this result: 

At this point, I looped through all images in the `camera_cal` directory, converting them to grayscale and running the `cv2.findChessboardCorners` routine to determine if the image has a detectable chessboard pattern. If it does, I append it to `objpoints` and `imgpoints`.

Finally, I output all chessboard images to a grid, as show below.

![Chessboard Corner Detection](https://gtoran.github.io/repository-assets/CarND-Advanced-Lane-Lines-P4/chessboard-corner.png)

### Pipeline (single images)

#### 1. Provide an example of a distortion-corrected image.

To demonstrate this step, I will describe how I apply the distortion correction to one of the test images like this one:

![Distortion Correction Image](https://gtoran.github.io/repository-assets/CarND-Advanced-Lane-Lines-P4/distortion-correction-image.png)

#### 2. Describe how (and identify where in your code) you used color transforms, gradients or other methods to create a thresholded binary image. Provide an example of a binary image result.

I implemented a pipeline with the following steps:

1. Gamma equalization.
2. Take a derivative of the image using the Sobel operator.
3. Transform image from RGB to HLS, filter each channel.
4. Combine filtered HLS channels.
5. Combine Sobel + HLS, generate binary image.

![Binary Pipeline Test](https://gtoran.github.io/repository-assets/CarND-Advanced-Lane-Lines-P4/binary-pipeline-test.png)

#### 3. Describe how (and identify where in your code) you performed a perspective transform and provide an example of a transformed image.

After masking the image by reusing the `region_of_interest` routine from Project 1, I chose to hard code my src and dst points after some manual calculations. This was due to the fact that my P1 project calculated pixels depending on the image size but the region masking was a little too tight for comfort. To ensure no problems arose at this point and to focus on the task at hand, I decided to proceed in this manner.

```
src_points = np.float32(
        [[0,720],
         [578,450],
         [702,450],
         [1280,720]])

dst_points = np.float32(
        [[320,720],
         [320,0],
         [960,0],
         [960,720]])

```

I ran a visual confirmation by outputting the transformed images along with their binary counterparts.

![Perspective Transform](https://gtoran.github.io/repository-assets/CarND-Advanced-Lane-Lines-P4/perspective-transform.png)

#### 4. Describe how (and identify where in your code) you identified lane-line pixels and fit their positions with a polynomial?

I defined a `calc_lane` function (block 13) that carries out lane-line pixel identification. After some initial testing, I used the code proposed in Chapter 33 of the Advanced Lane Lines project with the following tweaks:

* Histogram on the bottom quarter of the image.
* Reduced margin
* Implement a `first_run` workflow that conditionally determines the correct value for `left_lane_inds` and `right_lane_inds` depending on if it's the first run or not. If not first run, then add `left_fit` and `right_fit`. 

By applying this workflow, I obtain the following result plotting (block 14) one of the pipeline images:

![Lane Line Pixel Identification](https://gtoran.github.io/repository-assets/CarND-Advanced-Lane-Lines-P4/lane-line-pixel-transformation.png)

#### 5. Describe how (and identify where in your code) you calculated the radius of curvature of the lane and the position of the vehicle with respect to center.

I calculated the radius of the curvature in block 17 using the following workflow:

1. Obtain real distance (meters) from pixels using a xm_per_pix and ym_per_pix ratio. These numbers are obtained assuming a distance of 3.7 meters between lane lines on the X axis and 3 meters as the length of a dashed line. 
2. Use these ratios in the curve equation provided in the course material

![Curvature Equation](https://gtoran.github.io/repository-assets/CarND-Advanced-Lane-Lines-P4/curvature-equation.png)

3. Combine curvature calculations for both left and right lane lines and determine an average (`curvature_factor`).

#### 6. Provide an example image of your result plotted back down onto the road such that the lane area is identified clearly.

I implemented this step in block 20 & 21 by using lane curvature and vehicle offset to determine the polyfills that need to be warped back to the original image. I also overlaid the radius and offset on the image.

![Warped Image](https://gtoran.github.io/repository-assets/CarND-Advanced-Lane-Lines-P4/warp-overlay-map-lane.png)

I wanted to tackle the wobbly line issue I experienced in Project 1. After researching, a colleague suggested that I output a mean average of all polys as a way of smoothing out the graphical representation. This took quite a bit to achieve since I had to modify my workflow slightly, but I was able to achieve this in a satisfactory manner. Smoothing takes place in the `poly_smooth` method, defined in block 22.

---

### Pipeline (video)

#### 1. Provide a link to your final video output.  Your pipeline should perform reasonably well on the entire project video (wobbly lines are ok but no catastrophic failures that would cause the car to drive off the road!).

Here's a [link to my video result](./output_images/project_video.mp4).

I also ran my algorithm this on the harder challenge version with some interesting results. My initial thoughts were that the motorcycle overtaking the car would interrupt the shading immediately, but this wasn't the case. Shortly afterwards though the process destabilizes completely and I suspect that my smoothing method renders it useless.

For reference, here's a [link to the challenging video result](./output_images/challenge_project_output.mp4).

On a final note, I tried running the algorithm on one of my self-made driving clips but I was unable to get it to start due to the system not being able to identify the lane at all. My assumption is that I would have to review and tweak the parameters to calibrate for my camera and adjust region masking.

---

### Discussion

#### 1. Briefly discuss any problems / issues you faced in your implementation of this project.  Where will your pipeline likely fail?  What could you do to make it more robust?

Even though the pipeline works flawlessly on the project video, I'm intrigued by how to handle extreme situations. For example:

1. Bumper to bumper traffic. My gut feeling is that an anomaly like this would throw off the algorithm.
2. Theory tells us that using the HLS color space allows a broad range of lighting conditions. For example, if the algorithm is driving at night and a headlight were to go out, thus reducing road illumination, how would the algorithm handle this?
3. Other vehicles overtaking.
4. Worn out paint - could we work out a dead-reckoning algorithm for partial wear outs? Could this be considered safe?
5. Related to the previous point, how can we handle intersections, roundabouts or other non-lane-line road areas?