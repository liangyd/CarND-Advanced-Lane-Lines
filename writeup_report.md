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
[video1]: ./
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

I used a combination of color and gradient thresholds to generate a binary image.The details were written in the `binary_image` function of the "lane_detection.ipynb". I covered the image from RGB to HLS. I applied the Sobel operator on the L channel to calculate the derivative in the x direction. I also applied the color selection to get the pixels with a large value on S channel. 
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

#### 5. Describe how (and identify where in your code) you calculated the radius of curvature of the lane and the position of the vehicle with respect to center.

I calculated the curvature through the `curvature` function in my code in `lane_detection.ipynb`. The curvature calculation is based on the following equation:

![alt text][image7]

I calculated the position of the vehicle through the `offset` function by comparing the center of the image and the center of the two lanes. 

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

The pipeline was likely to fail when there are shadows or the road color changes. I used the smoothing function and the look-ahead filter to increase the robustness. Once the lane lines is detected in the previous frame, the next search will start from the previous lane detection result. It ensures a continuity in the detected lanes. 