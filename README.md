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

[image1]: ./assets/undistort_output.png "Undistorted"
[image2]: ./assets/undistort_road.png "Road Transformed"
[image3]: ./assets/grad_thresh.png "Grad Thresh"
[image4]: ./assets/color_thresh.png "Color Thresh"
[image5]: ./assets/combined_thresh.png "Combined Thresh"
[image6]: ./assets/perspective.png "Warp Example"
[image7]: ./assets/color_fit_lines.png "Fit Visual"
[image8]: ./assets/output.png "Output"
[image9]: ./assets/challenge.png "Challenge"
[video1]: ./project_video_annotated.mp4 "Video"

## [Rubric](https://review.udacity.com/#!/rubrics/571/view) Points

### Here I will consider the rubric points individually and describe how I addressed each point in my implementation.  

---

### Writeup / README

#### 1. Provide a Writeup / README that includes all the rubric points and how you addressed each one.  You can submit your writeup as markdown or pdf.  [Here](https://github.com/udacity/CarND-Advanced-Lane-Lines/blob/master/writeup_template.md) is a template writeup for this project you can use as a guide and a starting point.  

You're reading it!

### Camera Calibration

#### 1. Briefly state how you computed the camera matrix and distortion coefficients. Provide an example of a distortion corrected calibration image.

The code for this step is contained in the Camera Calibration section of the IPython notebook located in "./advanced_lane_finding.ipynb".  

I start by preparing "object points", which will be the (x, y, z) coordinates of the chessboard corners in the world. Here I am assuming the chessboard is fixed on the (x, y) plane at z=0, such that the object points are the same for each calibration image.  Thus, `objp` is just a replicated array of coordinates, and `objpoints` will be appended with a copy of it every time I successfully detect all chessboard corners in a test image.  `imgpoints` will be appended with the (x, y) pixel position of each of the corners in the image plane with each successful chessboard detection.  

I then used the output `objpoints` and `imgpoints` to compute the camera calibration and distortion coefficients using the `cv2.calibrateCamera()` function.  I applied this distortion correction to the test image using the `cv2.undistort()` function and obtained this result: 

![alt text][image1]

### Pipeline (single images)

#### 1. Provide an example of a distortion-corrected image.

Since distortion coefficients are obtained in Camera Calibration step, I simply use `cv2.undistort()` to undistort a road image.

![alt text][image2]

#### 2. Describe how (and identify where in your code) you used color transforms, gradients or other methods to create a thresholded binary image.  Provide an example of a binary image result.

Gradient threshold:
I use sobel filter to compute the gradient on x and y axis for each pixel. Then I applied threshold on the absolute value, magnitude, and slope for the gradient. The code for this step is contained in the Gradient Threshold section of the IPython notebook.

![alt text][image3]

Color threshold:
I converted the image to HLS space, LUV space and LAB space. Then I applied threshold on the S, L, B channels, correspondingly. The code for this step is contained in the Color Threshold section of the IPython notebook.

![alt text][image4]

This is the combined output of graident thresholding and color thresholding.

![alt text][image5]

#### 3. Describe how (and identify where in your code) you performed a perspective transform and provide an example of a transformed image.

The perspective transform matrix (M) is derived using `cv2.getPerspectiveTransform()` with hard coded source (`src`) and destination (`dst`) points. The code for this step is contained in the Perspective Transform section of the IPython notebook.

Here are source and destination points:

| Source        | Destination   | 
|:-------------:|:-------------:| 
| 700, 460      | 1070, 0       | 
| 1070, 690     | 1070, 690     |
| 240, 690      | 240, 690      |
| 580, 460      | 240, 0        |

I verified that my perspective transform was working as expected by drawing the `src` and `dst` points onto a test image and its warped counterpart to verify that the lines appear parallel in the warped image.

![alt text][image6]

#### 4. Describe how (and identify where in your code) you identified lane-line pixels and fit their positions with a polynomial?

I fit a second order polynomial into lane line pixels with following steps:

First, divide a single image into 9 vertical layers, each layer has a height of 80 pixels.
In each layer, I find out left lane pixels and right lane pixels using sliding window. I also provide a way to search for lane pixels given parameters from previous frame.

Second, I fit a second order polynomial to each lane line using `np.polyfit()`.

The code for this step is contained in the Finding Lane Lines section of the IPython notebook.

![alt text][image7]

#### 5. Describe how (and identify where in your code) you calculated the radius of curvature of the lane and the position of the vehicle with respect to center.

For calculating the radius of the curvature. I first fit a polynomial with pixels converted to meters for each lane. And then I calculate the radius using Radius of Curvature formula.

I calculate the postition of the vehicle with respect to center using this formula: `offset_in_meters = ((left_lane_pixel + right_lane_pixel) / 2 - image_width / 2) * xm_per_pix`.

Where left_lane_pixel, right_lane_pixel are the lane pixels closest to the vehicle.

The code for this step is contained in the Measuring Curvature & Offset section of the IPython notebook.

#### 6. Provide an example image of your result plotted back down onto the road such that the lane area is identified clearly.

The code for this step is contained in the Pipeline section of the IPython notebook. Here is an example of my result on a test image:

![alt text][image8]

---

### Pipeline (video)

#### 1. Provide a link to your final video output.  Your pipeline should perform reasonably well on the entire project video (wobbly lines are ok but no catastrophic failures that would cause the car to drive off the road!).

Here's a [link to my video result](./project_video_annotated.mp4)

---

### Discussion

#### 1. Briefly discuss any problems / issues you faced in your implementation of this project.  Where will your pipeline likely fail?  What could you do to make it more robust?

The algorithm is likely to fail if there are other "lines" on the road. For example, if the road color is different in the lane, the color bounary will be caught by gradient thresholding and generate false positives. I can leverge color information on both sides of the gradient to eliminate this kind of false positives.

![alt text][image9]
