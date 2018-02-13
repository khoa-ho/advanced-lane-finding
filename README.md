**Advanced Lane Finding Project**
=================================

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

[image0]: ./output_images/chessboard.png "Chessboard"
[image1]: ./output_images/undistort.png "Undistorted"
[image2]: ./output_images/warp.png "Road Transformed"
[image3]: ./output_images/threshold.png "Binary Thresholded"
[image4]: ./output_images/fill_lane.png "Lane Filled"
[video1]: ./result.mp4 "Video"

## [Rubric](https://review.udacity.com/#!/rubrics/571/view) Points

### Here I will consider the rubric points individually and describe how I addressed each point in my implementation.  

---

### Writeup / README

#### 1. Provide a Writeup / README that includes all the rubric points and how you addressed each one.  You can submit your writeup as markdown or pdf. 

You're reading it!

### Camera Calibration

#### 1. Briefly state how you computed the camera matrix and distortion coefficients. Provide an example of a distortion corrected calibration image.

The code for this step is contained in the 2nd code cell of the IPython notebook located in "./P4.ipynb".

I start by preparing "object points", which will be the (x, y, z) coordinates of the chessboard corners in the world. Here I am assuming the chessboard is fixed on the (x, y) plane at z=0, such that the object points are the same for each calibration image.  Thus, `objp` is just a replicated array of coordinates, and `objpoints` will be appended with a copy of it every time I successfully detect all chessboard corners in a test image.  `imgpoints` will be appended with the (x, y) pixel position of each of the corners in the image plane with each successful chessboard detection.  

I then used the output `objpoints` and `imgpoints` to compute the camera calibration and distortion coefficients using the `cv2.calibrateCamera()` function.  I applied this distortion correction to the test image using the `cv2.undistort()` function (4th code cell). Following is an example.

![alt text][image0]

### Pipeline (single images)

#### 1. Provide an example of a distortion-corrected image.

To demonstrate this step, I will describe how I apply the distortion correction to one of the test images like this one:
![alt text][image1]

#### 2. Describe how (and identify where in your code) you used color transforms, gradients or other methods to create a thresholded binary image.  Provide an example of a binary image result.

I used color thresholds to generate a binary image (thresholding steps in the 8th code cell).  Here's an example of my output for this step. 

![alt text][image3]

#### 3. Describe how (and identify where in your code) you performed a perspective transform and provide an example of a transformed image.

The code for my perspective transform includes a function called `transform_perspective()`, which appears in the 6th code cell of the IPython notebook.  I chose the hardcode the source and destination points in the following manner:

```python
src = np.float32([[569, 478],
                  [753, 478],
                  [1139, 719],
                  [258, 719]])
dst = np.float32([[300, 0],
                  [1000, 0],
                  [1000, 719],
                  [300, 719]])
```

I verified that my perspective transform was working as expected by drawing the `src` and `dst` points onto a test image and its warped counterpart to verify that the lines appear parallel in the warped image.

![alt text][image2]

#### 4. Describe how (and identify where in your code) you identified lane-line pixels and fit their positions with a polynomial?

I first take a histogram along all the columns in the lower half of the image. Then, I am adding up the pixel values along each column in the image. In my thresholded binary image, pixels are either 0 or 1, so the two most prominent peaks in this histogram will be good indicators of the x-position of the base of the lane lines. I can use that as a starting point for where to search for the lines. From that point, I can use a sliding window, placed around the line centers, to find and follow the lines up to the top of the frame. Finally, I used a second order polynomial to fit through the pixels belonging to the lane to obtain the estimate for the lane line.

![alt text][image4]

#### 5. Describe how (and identify where in your code) you calculated the radius of curvature of the lane and the position of the vehicle with respect to center.

The radius of curvature was calculated by converting x and y values to the real world space. For the project, I have assumed that if you're projecting a section of the lane, the lane is about 30 meters long and 3.7 meters wide. These steps were implemented in the `radius_of_curvature()` function in the 10th code cell of the notebook.

To calculate the position of the vehicle with respect to the center, I have assumed that the camera is mounted at the center of the car, such that the lane center is the midpoint at the bottom of the image between the two lines detected earlier. The offset of the lane center from the center of the image (converted from pixels to meters) is the distance from the center of the lane. The implementation was in the 11th code cell (line 104 - 106).


#### 6. Provide an example image of your result plotted back down onto the road such that the lane area is identified clearly.

I implemented this step in the 11th code cell in the function `fill_lane()`.  Here is an example of my result on a test image:

![alt text][image4]

---

### Pipeline (video)

#### 1. Provide a link to your final video output.  Your pipeline should perform reasonably well on the entire project video (wobbly lines are ok but no catastrophic failures that would cause the car to drive off the road!).

Here's a [link to my video result](./result.mp4)

---

### Discussion

#### 1. Briefly discuss any problems / issues you faced in your implementation of this project.  Where will your pipeline likely fail?  What could you do to make it more robust?

One of the big challenge is to incorporate the detection of lane lines in previous frame to speed up the search for lane lines in the current frame. The solution is to use an Line object to keep tract of lane line attributes in the previous frame. Even though the current pipeline works decently on the challenge clip (but not perfectly, especially when there is car on the adjacent lane), it performs suboptimally on the harder challenge clip. This could be because the current pipeline doesn't take into account the confidence level of the polynomial fitting. In other words, as long as some lane lines are found, it will use them as the baseline for the detection in the next frame. Thus, it may be better if the pipline ignore fitting with low confidence. Furthermore, if the Line class can store lane lines further back in the video, we can use the moving average of relevant attributes to reject outlier.
