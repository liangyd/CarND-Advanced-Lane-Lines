## Writeup Report

---

**Advanced Lane Finding Project**

The goals of this project are the following:

* Compute the camera calibration matrix and distortion coefficients given a set of chessboard images.
* Apply a distortion correction to raw images.
* Use color transforms, gradients, etc., to create a thresholded binary image.
* Apply a perspective transform to rectify binary image ("birds-eye view").
* Detect lane pixels and fit to find the lane boundary.
* Determine the curvature of the lane and vehicle position with respect to center.
* Warp the detected lane boundaries back onto the original image.
* Output visual display of the lane boundaries and numerical estimation of lane curvature and vehicle position.

[//]: # (Image References)

[image1]: ./output_images/chessboard/compare.jpg
[image2]: ./output_images/undistorted_image/compare2.jpg
[image3]: ./output_images/binary_image/test3.jpg
[image4]: ./output_images/perspective_transform/perspective_transform.jpg
[image5]: ./output_images/perspective_transform/test3_warped.jpg
[image6]: ./output_images/perspective_transform/lanefinding.jpg
[image7]: ./output_images/equation.jpg
[image8]: ./output_images/final_output/result_test3.jpg

---
### Here I will consider the rubric points individually and describe how I addressed each point in my implementation.  

---

### Writeup

#### 1. Provide a Writeup that includes all the rubric points and how you addressed each one.  You can submit your writeup as markdown.  [Here](https://github.com/udacity/CarND-Advanced-Lane-Lines/blob/master/writeup_template.md) is a template writeup for this project you can use as a guide and a starting point.  

You're reading it!

### Camera Calibration

#### 1. Briefly state how you computed the camera matrix and distortion coefficients. Provide an example of a distortion corrected calibration image.

The code for this step is contained in the first code cell of the IPython notebook located in "lane_detection.ipynb".

I start by preparing "object points", which will be the (x, y, z) coordinates of the chessboard corners in the world. Here I am assuming the chessboard is fixed on the (x, y) plane at z=0, such that the object points are the same for each calibration image.  Thus, `objp` is just a replicated array of coordinates, and `objpoints` will be appended with a copy of it every time I successfully detect all chessboard corners in a test image.  `imgpoints` will be appended with the (x, y) pixel position of each of the corners in the image plane with each successful chessboard detection.  

I then used the output `objpoints` and `imgpoints` to compute the camera calibration and distortion coefficients using the `cv2.calibrateCamera()` function.  I applied this distortion correction to the test image using the `cv2.undistort()` function and obtained this result: 

![alt text][image1]


### Pipeline (single images)

#### 1. Provide an example of a distortion-corrected image.

I applied the distortion correction to a test image:
![alt text][image2]

#### 2. Describe how (and identify where in your code) you used color transforms, gradients or other methods to create a thresholded binary image.  Provide an example of a binary image result.

I used a combination of color and gradient thresholds to generate a binary image.The details were written in the `binary_image` function of the "lane_detection.ipynb". 
I tried different color space to find the best way to detect the white and yellow lanes. I found out that the B channel in LAB color space and the L channel in HLS space have the best performance in lane detection. I did not apply any gradient threshold to the image.
Here's an example of my output for this step:
![alt text][image3]

#### 3. Describe how (and identify where in your code) you performed a perspective transform and provide an example of a transformed image.

The code for my perspective transform includes a function called `perspective_transform()` in the 8th code block in the "lane_detection.ipynb".  I chose the hard-coded source and destination points in the following manner:

```python
src = np.float32(
                [[570, 470],
                [210, 720],
                [1110, 720],
                [722, 470]])
dst = np.float32(
                [[(img_size[0] / 4), 0],
                [(img_size[0] / 4), img_size[1]],
                [(img_size[0] * 3 / 4), img_size[1]],
                [(img_size[0] * 3 / 4), 0]])
```

This resulted in the following source and destination points:

| Source        | Destination   | 
|:-------------:|:-------------:| 
| 570, 470      | 320, 0        | 
| 210, 720      | 320, 720      |
| 1110, 720     | 960, 720      |
| 722, 470      | 960, 0        |

I verified that my perspective transform was working as expected by drawing the `src` and `dst` points onto a test image and its warped counterpart to verify that the lines appear parallel in the warped image.

![alt text][image4]

Then I apply the perspective transform function to the test image:
![alt text][image5]

#### 4. Describe how (and identify where in your code) you identified lane-line pixels and fit their positions with a polynomial?

Then I used the sliding window method to filter out the pixels on lane lines, and fit my lane lines with a 2nd order polynomial like this:

![alt text][image6]

The lane finding method was included in the `lanefinder` function in the 16th code block in the `lane_detection.ipynb`. I took a histogram on the lower half of the image and find two peaks which were the start point of the right and the left lane lines. Then, I detected the pixels within a sliding window with a margin size of 70 and a height of 80. If more than 40 pixels were found within the window, the window would be resized to the center of all those pixels. 

Once the lane lines have been found in the previous image, the next search can start with the old lane line with a +/- margin around its line center. 

After that, I also added a sanity check which ensures that the two lane lines are almost parallel. I calculated the lane distance at the top and bottom of the image. If these two lane distances are not the same, then the lane detection result is invalid. If the polyfit coefficients changes rapidly, the lane detection result is also invalid.

If the detected lane lines fail in the sanity check, the lane finding algorithm will be reset and the lane finding will start with the histogram calculation again.

#### 5. Describe how (and identify where in your code) you calculated the radius of curvature of the lane and the position of the vehicle with respect to center.

The curvature and vehicle position was calculated in the real world space, which requires a conversion from pixel to meter. In general, the lane is about 30 meters long and 3.7 meters wide on an image. Thus, 720 pixels in y direction represents 30 meter and 700 pixels in x direction represents 3.7 meters. 

After the conversion, I calculated the curvature through the `curvature` function in my code in `lane_detection.ipynb`. The curvature calculation is based on the following equation:

![alt text][image7]

I calculated the position of the vehicle through the `offset` function by comparing the center of the image and the center of the two lanes, assuming that the camera is mounted at the center of the vehicle. 

#### 6. Provide an example image of your result plotted back down onto the road such that the lane area is identified clearly.

I implemented this step through the `draw` function in my code in `lane_detection.ipynb`.  Here is an example of my result on a test image:

![alt text][image8]

---

### Pipeline (video)

#### 1. Provide a link to your final video output.  Your pipeline should perform reasonably well on the entire project video.

Here's a [link](./project_video_output.mp4) to my video result.
When I wrote the pipeline, I also constructed a class `Line()` to temporarily store the parameters in the lane detection on each frame. I also used a moving average filter to smooth the lane detection over frames. 

---

### Discussion

#### 1. Briefly discuss any problems / issues you faced in your implementation of this project.  Where will your pipeline likely fail?  What could you do to make it more robust?

The first problem that I encountered was that the lane detection was affected by the shadows, road surface color, brightness, etc. I compared different channels in different color spaces to find out the best way to represent both lane lines. 

Another problem I had was that the detected lane lines was off from the true positions. Thus, I introduced the sanity check to find out if the detected lane lines are valid. I used the `subclip` function to debug the problematic frames instead of processing all the video frames.

The pipeline performs well on the basic video. However, in the challenging video, this pipeline does not perform well. The pipeline may fail in the following conditions:

* When the car goes through a tunnel or under a bridge, the ambient light is very dark which makes the lane lines hard to be detected. 
* When the lane lines are covered with snow, the algorithm cannot tell the difference between white lane lines and white snow.
* When the camera type or the camera position changes, it needs to recalculate the calibration matrix and the perspective tranform matrix. Otherwise, it will fail.

I have also considered some approaches to make the pipeline more robust:

* Add a region of interest in the raw image and remove the outliers 
* Use other curve fitting methods instead of the 2-order polynomial fitting
* Use other edge detection method such as the Laplace operator
* Refine the thresholds in the color space 


