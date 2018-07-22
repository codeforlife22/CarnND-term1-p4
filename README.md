

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

[image1]: ./output_images/camera_calibration_test1.png "Undistorted1"
[image2]: ./output_images/camera_calibration_test2.png "Undistorted2"
[image3]: ./output_images/camera_calibration_test3.png "Undistorted3"
[image4]: ./output_images/gradientx_thresh.png "gradientx"
[image5]: ./output_images/color_space_thresh1.png "colorth1"
[image6]: ./output_images/color_space_thresh2.png "colorth2"
[image7]: ./output_images/combine_thresh.png "combineth"
[image8]: ./output_images/perspective_test1.png "pers1"
[image9]: ./output_images/perspective_test2.png "pers2"
[image10]: ./output_images/fit_poly.png "fitpoly"
[image11]: ./output_images/exampe_image.png "example"
[video1]: ./project_video_output.mp4 "Video"


### Camera Calibration

The code for this step is contained in cells [57] and [60] of the IPython notebook located in "./P4.ipynb".

I start by preparing "object points", which will be the (x, y, z) coordinates of the chessboard corners in the world. Here I am assuming the chessboard is fixed on the (x, y) plane at z=0, such that the object points are the same for each calibration image.  Thus, `objp` is just a replicated array of coordinates, and `objpoints` will be appended with a copy of it every time I successfully detect all chessboard corners in a test image.  `imgpoints` will be appended with the (x, y) pixel position of each of the corners in the image plane with each successful chessboard detection.  

I then used the output `objpoints` and `imgpoints` to compute the camera calibration and distortion coefficients using the `cv2.calibrateCamera()` function.  I applied this distortion correction to the test images using the `cv2.undistort()` function and obtained this result: 

![alt text][image1]
![alt text][image2]

### Pipeline (single images)

#### 1. A distortion-corrected image.
Using the camera calibration and distortion coefficients obtained from the previous step, it is easy to get an undistorted image. Here is an example,

![alt text][image3]

#### 2. Describe how (and identify where in your code) you used color transforms, gradients or other methods to create a thresholded binary image.  Provide an example of a binary image result.

I used a combination of color and gradient thresholds to generate a binary image. 
1) Selecting a proper threshold in x-orientation gradient helps greatly in lane line detection. (code cells [66] - [69])
Here is an example of applying gradient thresholding in x-direction. 
![alt text][image4]

In addition to x-gradient, magnitute and directional gradient also help in detecting lane lines, therefore, thresholding in all three gradients were adopted in this project. 

2) Processing images in HLS color space enhances lane line detections, especially for yellow lines. (code cells [70] - [72])
Below is a example that demonstrates the power of proper thresholing in each individual channel of HLS color space,
![alt text][image5]
![alt text][image6]
In this project, color thresholding in L and S channels are used, and L Channel has been proved to be very useful for handling images with shadows (different lighting conditions).

Combining color and gradient thresholds, the output image is shown below,
![alt text][image7]
#### 3. Perspective transfrom

The code for my perspective transform includes two functions called `cal_warp()` and `get_warped_image()`, which appears in cell [63] in jupyter notebook.  The `cal_warp()` function takes as inputs an image (`img`). I chose the following hardcoded source and destination points (clockwise from the top left point, based on the example warpped image provided by Udacity) :

| Source        | Destination   | 
|:-------------:|:-------------:| 
| 590, 450      | 290, 0        | 
| 685, 450      | 1030, 0       |
| 1120, 720     | 1030, 720     |
| 200, 720      | 290, 720      |

I verified that my perspective transform was working as expected by applying perspective transfrom to all test images, 
and I selected two examples for illustration purpose.
![alt text][image8]
![alt text][image9]
#### 4. Lane pixels identification and curve fitting (cell [121])
Firstly, I used histogram peak identification to find the starting points for left lane and right lane, respectively. Then sliding window technique was utilized to find all lane pixels. Finally, I tried to fit the lane lines to a second-order polynomial curve.
An example image is shown below:

![alt text][image10]

#### 5. Curvature and vehicle position calculation (cell [122])
The caculate curvature in real world, I first defined conversions in x and y from pixels space to meters (from lecture)
ym_per_pix = 30/720 # meters per pixel in y dimension
xm_per_pix = 3.7/700 # meters per pixel in x dimension
Then I did a np.polyfit(...) with the transformed real-world coordindates to obtain the polynomial coefficients, and curvature can be calculated readily. 

The offset of the lane center from the center of the image (converted from pixels to meters) is the distance from the center of the lane. 

#### 6. Plot the detected lane area and auxillary info on top of the original image

I implemented this step in cell [123]. Here is an example of my result on a test image:

![alt text][image11]

---

### Pipeline (video)

Here's a [link to my video result](https://github.com/codeforlife22/CarnND-term1-p4/blob/master/project_video_out.mp4)

---

### Discussion


There was some issues when the lane lines are in shadows, the detected area seems to be very unstable. 
To counter this problem, I added color thresholing in L channel of HLS color space, which proves to be quite effective. 

It may be very chanllenging to detection lane line if the images are subject to large lighting variations. Another issue that I can think of is when cars changing lanes, the pipeline may accidently pick up pixels from the cars and mistake them for lane lines.  To make it more robust, techniques such as taking average across different frames, and tracking the previously identified parameters can be adopted.  
