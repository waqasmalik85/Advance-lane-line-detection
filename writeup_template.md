##Writeup Template


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

[image1]: ./examples/undistort_output.png "Undistorted"
[image2]: ./test_images/test1.jpg "Road Transformed"
[image3]: ./examples/binary_combo_example.jpg "Binary Example"
[image4]: ./examples/warped_straight_lines.jpg "Warp Example"
[image5]: ./examples/color_fit_lines.jpg "Fit Visual"
[image6]: ./examples/example_output.jpg "Output"
[image7]: ./examples/chess_comparison.jpg "dist_chess"
[image8]: ./examples/flowchart.png "Undist_chess"
[image9]: ./examples/corners_found11.jpg "Corners"
[image10]: ./examples/test_comparison.jpg "Test"
[image11]: ./examples/binary.jpg "Binary"
[image12]: ./examples/warped.jpg "Warped"
[image13]: ./examples/histogram.jpg "Hist"
[image14]: ./examples/binary7.jpg "Bin"
[image15]: ./examples/curve_lines.jpg "Curve"
[image16]: ./examples/polygon.jpg "Poly"
[image17]: ./examples/final_image_curvature7.jpg "Final"
[video1]: ./project_video.mp4 "Video"

## [Rubric](https://review.udacity.com/#!/rubrics/571/view) Points
###Here I will consider the rubric points individually and describe how I addressed each point in my implementation.  

---
###Writeup / README

####1. Provide a Writeup / README that includes all the rubric points and how you addressed each one.  You can submit your writeup as markdown or pdf.  [Here](https://github.com/udacity/CarND-Advanced-Lane-Lines/blob/master/writeup_template.md) is a template writeup for this project you can use as a guide and a starting point.  

You're reading it!
###Camera Calibration

####1. Briefly state how you computed the camera matrix and distortion coefficients. Provide an example of a distortion corrected calibration image.

The code for this step is contained in the second and third code cell of the IPython notebook `adv-lane-lines.ipynb`.

I start by preparing "object points", which will be the (x, y, z) coordinates of the chessboard corners in the world. Here I am assuming the chessboard is fixed on the (x, y) plane at z=0, such that the object points are the same for each calibration image.  Thus, `objp` is just a replicated array of coordinates, and `objpoints` will be appended with a copy of it every time. Calibration image 0, 14 and 15 failed and could not be used to detect chessboard corners.  `imgpoints` were appended with the (x, y) pixel position of each of the corners in the image plane with each successful chessboard detection. Below is an example of found corners:-
![alt text][image9]  

I then used the output `objpoints` and `imgpoints` to compute the camera calibration and distortion coefficients using the `cv2.calibrateCamera()` function.  I applied this distortion correction to the test image using the `cv2.undistort()` function and obtained this result. Image on the top is calibrated and the improved result can be seen at the image on the bottom.

![alt text][image7]


###Pipeline (single images)

The flow chart below explains the steps taken for an individual image to find the lane markings:-


![alt text][image8]

####1. Provide an example of a distortion-corrected image.

Camera matrix calculated by the chess boards in then applied to the test images in code cell 7 to correct the distortions. Following is the result of correction:

![alt text][image10]
####2. Describe how (and identify where in your code) you used color transforms, gradients or other methods to create a thresholded binary image.  Provide an example of a binary image result.
Function called `lightness_saturation()` is used to transform image from RGB to HLS, Gradiant in the 'x' direction is applied on the Lightness part using Sobelx. Thresholding is applied on the Saturation part to create two binary images. These two binary images are combined to create the final image to search for lane lines. It has been observed that applying histogram equiliazion to balance out shadows to Lighness part has made the images worst so it avoided. Following is the result of the function:-

![alt text][image11]

####3. Describe how (and identify where in your code) you performed a perspective transform and provide an example of a transformed image.

The code for my perspective transform includes a function called `perspective_transform()` the function takes as inputs an image (`img`), as well as source (`src`) and destination (`dst`) points.  I chose the source and destination points by visual inspection of the images and hardcoded them in the following manner:

This resulted in the following source and destination points:

| Source        | Destination   |
|:-------------:|:-------------:|
| 298, 662     | 280, 700      |
| 602, 447      | 280, 0     |
| 683, 447    | 920, 0      |
| 1020, 662    | 920, 700       |

I verified that my perspective transform was working as expected by drawing the `src` and `dst` points onto a test image and its warped counterpart to verify that the lines appear parallel in the warped image.

![alt text][image12]

####4. Describe how (and identify where in your code) you identified lane-line pixels and fit their positions with a polynomial?

Two functions `lane_finder()` and `lane_finder_from_past()` are used to find pixels which belong to lane lines. First to be called is `lane_finder()` it takes the warped binary image as shown below:-

![alt text][image14]

and estimates a histogram of non-zero pixels. Below is how the histogram should look lile:-

![alt text][image13]

Peaks in the histogram should the most likely starting points to find lane pixels. Moving window is applied which identifies the hot pixels as part of the lane from bottom to the top of the image. Following image demonstartes this concept:-

![alt text][image15]

Red marked pixels belong to the left lane and blue represent the line lane. A second order polynomial is regressed through these points and resulting lane curves are displayed as yellow.

Once the lanes are found the coefficentes of the polynomials are buffered in the function `buffering()`in a buffer of size 10. At a given point in time an average of all the coefficents represent lane lines. Moreover `lane_finder_from_past()` is used to quickly find out the relevant lane pixels rather than creating a histogram everytime.

####5. Describe how (and identify where in your code) you calculated the radius of curvature of the lane and the position of the vehicle with respect to center.

Function `find_curvature()` uses the regressed polynomals and converts the pixel coordinate to world coordinates and output the radius of the curvature in meters. Position of the camera is the mean of the two lowest values of the estimated lanes.

If the estimates curvatures are not plausible `sanity_check()` would delete the current coefficent and use the buffered values only to generate lane lines.

####6. Provide an example image of your result plotted back down onto the road such that the lane area is identified clearly.

First of all a polygon is drawn using lane coefficents of both left and right lanes using function  `create_polygon()` Then the polygon is warped back just using the function `perspective_transform()` however this time source (`src`) and destination (`dst`) points are reversed.   Following image demonstrates this process:-

![alt text][image16]

Next steps are to calucate the curvatue radius of the lanes and position of the camera with respect to center of the lane. Super imposes the polygon and text on the undistorted input image using the function  `find_curvature()`. Following is an example of the resulting image:-
![alt text][image17]

---

###Pipeline (video)

####1. Provide a link to your final video output.  Your pipeline should perform reasonably well on the entire project video (wobbly lines are ok but no catastrophic failures that would cause the car to drive off the road!).

Here's a [link to my video result](./project_video.mp4)

---

###Discussion

####1. Briefly discuss any problems / issues you faced in your implementation of this project.  Where will your pipeline likely fail?  What could you do to make it more robust?

First issue was to select pixels from the original image to be used as the source (`src`) points for warping. In future there should be an automated system to roughly dtect the lanes and use those points as source points.

Secondly finding the optimal thresholds to create the binary image needs alot of experimentation. It could be challenging.

 Algorithm would already start wrong if somehow the first histogram peak is not where the lane lines are.

 Further improvements can be brough by comparing the distance between the lanes and to make sure that the distance remains constant 
