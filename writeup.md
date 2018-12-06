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

[image1]: ./camera_cal/calibration1.jpg "./output_images/calibration1.jpg"
[image2]: ./output_images/calibration1_undistortion.jpg "./output_images/calibration1_undistortion.jpg"
[image3]: ./test_images/test1.jpg "./test_images/test1.jpg"
[image4]: ./output_images/test2_undist.jpg "./output_images/test2_undist.jpg"
[image5]: ./output_images/test2_b_ch.jpg "./output_images/test2_b_ch.jpg"
[image6]: ./output_images/test2_l_ch.jpg "./output_images/test2_l_ch.jpg"
[image7]: ./output_images/test2_thresh.jpg "./output_images/test2_thresh.jpg"
[image8]: ./output_images/test2_persp.jpg "./output_images/test2_persp.jpg"
[image9]: ./output_images/test2_window.jpg "./output_images/test2_window.jpg"
[image10]: ./output_images/test2_caption.jpg "./output_images/test2_caption.jpg"

## [Rubric](https://review.udacity.com/#!/rubrics/571/view) Points

### Here I will consider the rubric points individually and describe how I addressed each point in my implementation.  

---

### Writeup / README

#### 1. Provide a Writeup / README that includes all the rubric points and how you addressed each one.  You can submit your writeup as markdown or pdf.  [Here](https://github.com/udacity/CarND-Advanced-Lane-Lines/blob/master/writeup_template.md) is a template writeup for this project you can use as a guide and a starting point.  

You're reading it!

### Camera Calibration

#### 1. Briefly state how you computed the camera matrix and distortion coefficients. Provide an example of a distortion corrected calibration image.

The code for this step is contained in the first code cell of the IPython notebook located in "./P2.ipynb".  

I start by preparing "object points", which will be the (x, y, z) coordinates of the chessboard corners in the world. Here I am assuming the chessboard is fixed on the (x, y) plane at z=0, such that the object points are the same for each calibration image.  Thus, `objp` is just a replicated array of coordinates, and `objpoints` will be appended with a copy of it every time I successfully detect all chessboard corners in a test image.  `imgpoints` will be appended with the (x, y) pixel position of each of the corners in the image plane with each successful chessboard detection.  

I then used the output `objpoints` and `imgpoints` to compute the camera calibration and distortion coefficients using the `cv2.calibrateCamera()` function.  I applied this distortion correction to the test image using the `cv2.undistort()` function and obtained this result: 

| Original Chessboard |  Undistorted Chessboard |
|:-------------------:|:-----------------------:|
| ![alt text][image1] | ![alt text][image2]     |

### Pipeline (single images)

#### 1. Provide an example of a distortion-corrected image.

The code for this step is contained in the `undistort()` function of the IPython notebook.

To demonstrate this step, I applied the `cv2.undistort()`, as same as last step, to one of the test images like this one:

| Original Image      | Undistorted Image   |
|:-------------------:|:-------------------:|
| ![alt text][image3] | ![alt text][image4] | 

#### 2. Describe how (and identify where in your code) you used color transforms, gradients or other methods to create a thresholded binary image.  Provide an example of a binary image result.

I used combination of B & L channel for creating a thresholded binary image.
* The L Channel from the LUV color space, with a min threshold of 215 and a max threshold of 255, did an almost perfect job of picking up the white lane lines, but completely ignored the yellow lines.
* The B channel from the Lab color space, with a min threshold of 145 and an upper threshold of 200, did a better job than the S channel in identifying the yellow lines, but completely ignored the white lines.

| Operation | Original Image      | L Channel           | B Channel           | Thresholded Image    |
|-----------|:-------------------:|:-------------------:|:-------------------:|:--------------------:|
| Result    | ![alt text][image3] |![alt text][image5]  | ![alt text][image6] | ![alt text][image7]  |

#### 3. Describe how (and identify where in your code) you performed a perspective transform and provide an example of a transformed image.

The code for my perspective transform includes a function called `warper()`, which appears in lines 1 through 8 in the file `example.py` (output_images/examples/example.py) (or, for example, in the 3rd code cell of the IPython notebook).  The `warper()` function takes as inputs an image (`img`), as well as source (`src`) and destination (`dst`) points.  I chose the hardcode the source and destination points in the following manner:

```python
src = np.float32(
        [[(img_size[0] / 2) - 40, img_size[1] / 2 + 100], 
        [((img_size[0] / 6) - 50), img_size[1]],
        [(img_size[0] * 5 / 6) + 80, img_size[1]], 
        [(img_size[0] / 2 + 80), img_size[1] / 2 + 100]])
    
    dst = np.float32(
        [[(img_size[0] / 4), 0],
        [(img_size[0] / 4), img_size[1]],
        [(img_size[0] * 3 / 4), img_size[1]],
        [(img_size[0] * 3 / 4), 0]])
```

This resulted in the following source and destination points:

| Source        | Destination   | 
|:-------------:|:-------------:| 
| 600, 460      | 320, 0        | 
| 163, 720      | 320, 720      |
| 1146, 720     | 960, 720      |
| 720, 460      | 960, 0        |

I verified that my perspective transform was working as expected by drawing the `src` and `dst` points onto a test image and its warped counterpart to verify that the lines appear parallel in the warped image.

![alt text][image8]

#### 4. Describe how (and identify where in your code) you identified lane-line pixels and fit their positions with a polynomial?

Lane-lines pixels are identified using modified Udacity provided function find_window_centroids in laneDetectPipeline. The modifications are done to add a list for y values of corresponding centroids

polyFit is done in laneDetectPipeline as well using np.polyfit.

![alt text][image9]

#### 5. Describe how (and identify where in your code) you calculated the radius of curvature of the lane and the position of the vehicle with respect to center.

I did this in `measure_curvature_real()` function of the IPython notebook.

#### 6. Provide an example image of your result plotted back down onto the road such that the lane area is identified clearly.

I implemented this step in the function `draw_lane()` and `caption()`.  Here is an example of my result on a test image:

![alt text][image10]

---

### Pipeline (video)

#### 1. Provide a link to your final video output.  Your pipeline should perform reasonably well on the entire project video (wobbly lines are ok but no catastrophic failures that would cause the car to drive off the road!).

Here's a [link to my video result](https://youtu.be/Grod-uoh52U)

---

### Discussion

#### 1. Briefly discuss any problems / issues you faced in your implementation of this project.  Where will your pipeline likely fail?  What could you do to make it more robust?

Although I successfully detected the lane of the `project_video.mp4`. But when I apply my pipeline to `challenge_video.mp4` or `harder_challenge_video.mp4`, it can't detect normally. Below are some portions I may need to improve.

* when Changing lanes or due to some lighting conditions, this may not work.
* If the lane have some lane like shadow or cavity, it may affect my lane detection. I need to improve lane detection algorithm.
* If the curvature is too large, my lane detection algorithm may fail to detect lane with the background. This may be fixed using deep learning techniques which I will be learning in the upcoming lessons.
