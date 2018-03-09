## Writeup Template

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

[//]: # (Image References)

[image1]: ./output_images/calibration1_undistorted.jpg "Undistorted"
[image2]: ./output_images/distortion_corrected_test3.jpg "Road Transformed"
[image3]: ./output_images/thresholded_binary_test3.jpg "Binary Example"
[image4]: ./output_images/warped_img_test3.jpg "Warp Example"
[image5]: ./output_images/lanes_img_test3.jpg "Fit Visual"
[image6]: ./output_images/pipeline_processed_test3.jpg "Output"
[video1]: ./project_video_output.mp4 "Video"

## [Rubric](https://review.udacity.com/#!/rubrics/571/view) Points

### Here I will consider the rubric points individually and describe how I addressed each point in my implementation.  

---

### Writeup / README

#### 1. Provide a Writeup / README that includes all the rubric points and how you addressed each one.   [Here](https://github.com/ssthind/CarND-Advanced-Lane-Lines/blob/master/writeup.md) is a template writeup for this project you can use as a guide and a starting point.  

You're reading it!
And the entire code for this project is in IPython notebook located in [Here](https://github.com/ssthind/CarND-Advanced-Lane-Lines/blob/master/CarND-Advanced-Lane-Lines.ipynb)

### Camera Calibration

#### 1. Briefly state how you computed the camera matrix and distortion coefficients. Provide an example of a distortion corrected calibration image.

The code for this step is contained in the first code cell of the IPython notebook located in [Here](https://github.com/ssthind/CarND-Advanced-Lane-Lines/blob/master/CarND-Advanced-Lane-Lines.ipynb).  

I start by preparing "object points", which will be the (x, y, z) coordinates of the chessboard corners in the world. Here I am assuming the chessboard is fixed on the (x, y) plane at z=0, such that the object points are the same for each calibration image.  Thus, `objp` is just a replicated array of coordinates, and `objpoints` will be appended with a copy of it every time I successfully detect all chessboard corners in a test image.  `imgpoints` will be appended with the (x, y) pixel position of each of the corners in the image plane with each successful chessboard detection.  

I then used the output `objpoints` and `imgpoints` to compute the camera calibration and distortion coefficients using the `cv2.calibrateCamera()` function.  I applied this distortion correction to the test image using the `cv2.undistort()` function and obtained this result: 

![alt text][image1]

### Pipeline (single images)

#### 1. Provide an example of a distortion-corrected image.

To demonstrate this step, I will describe how I apply the distortion correction to one of the test images like this one(defined a function[code cell 5 of IPython notebook] for calling this functionality in pipeline):
![alt text][image2]



#### 2. Describe how (and identify where in your code) you used color transforms, gradients or other methods to create a thresholded binary image.  Provide an example of a binary image result.

I used a combination of color and gradient thresholds to generate a binary image which is combination of thresholding saturation channel(from HLS color space) and red channel(from RGB color space) and soble x gradient on l-channel(from HLS color space). 
Thresholding function is defined as `detect_color_gradient()` at code cell 7 in IPython notebook).  Here's an example of my output for this step.  

![alt text][image3]
Note the upper half of the image seems to have the mountains, however this is not a problem, as infurther processing during warping this part of the image is ignored.

#### 3. Describe how (and identify where in your code) you performed a perspective transform and provide an example of a transformed image.

The code for my perspective transform includes a function called `prespective_transform()`, (defined in the 9th code cell of the IPython notebook).  The `prespective_transform()` function takes as inputs an image (`img`), as well as source (`src`) and destination (`dst`) points.  I chose the hardcode the source and destination points in the following manner:

```python
src = np.float32([ [418, 570], [272, 674], [1052, 674], [882, 570]])
dst = np.float32( [[320, 610], [320, 700], [1060, 700], [1060, 610]])
```

This resulted in the following source and destination points:

| Source        | Destination   | 
|:-------------:|:-------------:| 
| 418, 570      | 320, 610      | 
| 272, 674      | 320, 700      |
| 1052, 674     | 1060, 700     |
| 882, 570      | 1060, 610     |

I verified that my perspective transform was working as expected by drawing the `src` and `dst` points onto a test image and its warped counterpart to verify that the lines appear parallel in the warped image.

![alt text][image4]

#### 4. Describe how (and identify where in your code) you identified lane-line pixels and fit their positions with a polynomial?

Then I did some other stuff and fit my lane lines with a 2nd order polynomial using function `detect_lane()` in code cell 13 of IPython notebook. Which uses sliding window technique, and resulted in below output:

![alt text][image5]

#### 5. Describe how (and identify where in your code) you calculated the radius of curvature of the lane and the position of the vehicle with respect to center.

I calculated 'radius of curvature' using a function defined as `calc_lane_curvature()` in code cell 15 of IPython notebook. 
Position of vehicle calculated as deviated from the center of the lane which would center of difference between 'center of lane' minus 'center of image'(denote center of vehicle) , 
further this difference scaled by a factor 'meters per pixel' in 'x-direction'(column of image) to convert the distance in meters.
These calculations performed for 'center of lane' in function `draw_lanes()` in code cell 16. And calculation for 'Position of vehicle' `pipeline_image_processing()` in code cell 17 of IPython notebook.


#### 6. Provide an example image of your result plotted back down onto the road such that the lane area is identified clearly.

I implemented this step using function `draw_lanes()` in code cell 16 of IPython notebook. And the entire pipeline is invoked using function `pipeline_image_processing()` in code cell 17. Here is an example of my result on a test image:

![alt text][image6]

---

### Pipeline (video)

#### 1. Provide a link to your final video output.  Your pipeline should perform reasonably well on the entire project video (wobbly lines are ok but no catastrophic failures that would cause the car to drive off the road!).

Here's a [link to my video result](./project_video_output.mp4)

---

### Discussion

#### 1. Briefly discuss any problems / issues you faced in your implementation of this project.  Where will your pipeline likely fail?  What could you do to make it more robust?

Here I'll talk about the approach I took, what techniques I used, 
* what worked was sliding window technique on each image worked better, instead of using the `skip_sliding_window` function due to higher noise.
* and manual selection of src coordinates on image with straight lanes.
* Better thresholding techniques can to used to generalised to code to work on other videos streams. 
* Dynamic generation of source coordinates for warping, would tried in next version of this project. Which will be based on the fact that the width of the road between two lanes should be constant, for any kind of lanes: 
 > straight lanes or  
 > curvered lanes 
  
* For more sharply curved lanes higher degree polynomial could be tried.