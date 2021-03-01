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

All code for this project can be found in the python notebook called Advanced_lane_finding.ipynb

### Camera Calibration

#### 1. Briefly state how you computed the camera matrix and distortion coefficients. Provide an example of a distortion corrected calibration image.

First I get the object points, which are the (x,y,z) coordinates of the chessboard corners. I assume that the chessboard is fixed on the (x,y) plane at z=0 such that the object points are the same for each calibration image. This means that 'objp' is appended to 'objpoints' whenever all chessboard corners are successfully detected in a test image. 'imgpoints' will be appended with the (x,y) pixel position of each of the corners in the image plane with each successful chessboard detection.  

The calibration and distortion coefficients were calculated using 'objpoints' and 'imgpoints'. These coefficients were used to apply distortion correction to a test image. The result can be seen in "output_images/undistorted_calibration1.jpg"

### Pipeline (single images)

#### 1. Provide an example of a distortion-corrected image.

Using the same distortion correction process above I undistorted all of the test images the results of which can be seen in "output_images/Undistorted_images"


#### 2. Describe how (and identify where in your code) you used color transforms, gradients or other methods to create a thresholded binary image.  Provide an example of a binary image result.

Function color_combined is where I applied color thresholding and gradients. I used the HLS color space with x gradients between 20 and 100, and s color channels between 170 and 255. The results of the combination to create a combined_binary image can be seen in "output_images/Combined_binary_images"


#### 3. Describe how (and identify where in your code) you performed a perspective transform and provide an example of a transformed image.

To do my perspective transform I used function warper(). This function takes a binary image, and the src and dst (source and destination) points. In this case the points were hardcoded with points I selected by examining the original image.

```src = np.float32(
        [[568, 468], [715, 468], 
        [1040, 680], [270,680]])
    
dst = np.float32(
       [[200,0], [1000, 0], 
       [1000, 680], [200, 680]])

```

The perspective transform was verified by examining the image after the perpective transform was performed as can be seen in the images in "output_images/Birds_eye_images"


#### 4. Describe how (and identify where in your code) you identified lane-line pixels and fit their positions with a polynomial?

On the first run of the video and for each of the test images, function find_lane_pixels() was used to window the bird_eye image and fit a polynomial. A histogram was used to identify where the lane lines were and windowing was used to fully identify the lanes when the road curves. For each proceeding frame the function search_around_poly was used to identify the lane pixels, this was to reduce repeated computations that were unnecessary. Since we already had an idea of where to look in the image for the lane lines there is no need to search the entire image but only a small subset of the pixels. The result of the windowing can be seen in the test image output in "output_images/Poly_lines_images".

#### 5. Describe how (and identify where in your code) you calculated the radius of curvature of the lane and the position of the vehicle with respect to center.

Function measure_curvature_real() was used to calculate the radius of curvature. This function takes the image output from the previous step as well as the left_fitx and right_fitx (the fitted polynomials). It calculates the curvature and converts the values to meters. To find the position with respect to center, first I found the midpoint between the left and right x-intercept. Then using this value I calculated the pixel offset by finding the difference between the true middle x-intercept for the image and the calculated position. This result was then converted to meters. 

#### 6. Provide an example image of your result plotted back down onto the road such that the lane area is identified clearly.

A new perspective transform was carried out to return the image to its original perspective. First the Minv (inverse of the original perspective transform) was calculated and applied using cv2.warpPerspective. The area between the left and right lanes is filled with the color green using polyfill(). The values from the previous step were then output on the result image. The result images can be seen in "output_images/Result_images"

---

### Pipeline (video)

#### 1. Provide a link to your final video output.  Your pipeline should perform reasonably well on the entire project video (wobbly lines are ok but no catastrophic failures that would cause the car to drive off the road!).

The output video is called "project_video_output.mp4" and is located in the base directory.

---

### Discussion

#### 1. Briefly discuss any problems / issues you faced in your implementation of this project.  Where will your pipeline likely fail?  What could you do to make it more robust?

My pipeline works fairly well on this video, the only major problems it encounters are when the lane lines are very difficult to make out or non-existant. It might help to use the same polynomial for both lane lines when one is of poor quality or non-existant, provided the other lane is not. This may not result in a perfect read but would keep the system from completely losing the lines. I also only run the windowing on the first frame, the result may be improved if windowing was used whenever the result was poor from search_around_poly(). However, in the case that the lane lines are non-existant I am not sure it would make much difference. 
