## Advanced Lane Finding Project

---

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

[image1]: ./output_images/original_calibration2.jpg "Original"
[image2]: ./output_images/undistort_calibration2.jpg "Undistorted"


[image11]: ./output_images/undistort_test3.jpg "Undistorted"
[image21]: ./output_images/undistort_straight_lines1.jpg "Undistorted"
[image31]: ./output_images/undistort_test1.jpg "Undistorted"
[image41]: ./output_images/undistort_test5.jpg "Undistorted"

[image12]: ./output_images/binary_test3.jpg
[image22]: ./output_images/binary_straight_lines1.jpg
[image32]: ./output_images/binary_test1.jpg
[image42]: ./output_images/binary_test5.jpg

[image13]: ./output_images/warped_test3.jpg
[image23]: ./output_images/warped_straight_lines1.jpg
[image33]: ./output_images/warped_test1.jpg
[image43]: ./output_images/warped_test5.jpg

[image14]: ./output_images/final_test3.jpg
[image24]: ./output_images/final_straight_lines1.jpg
[image34]: ./output_images/final_test1.jpg
[image44]: ./output_images/final_test5.jpg

[image3]: ./test_images/test4.jpg
[image4]: ./output_images/undistort_test4.jpg
[image5]: ./test_images/test6.jpg
[image6]: ./output_images/binary_test6.jpg
[image7]: ./output_images/undistort_straight_lines2.jpg
[image8]: ./output_images/warped_straight_lines2.jpg
[image9]: ./examples/formula.png 

[video1]: ./video_output/project_video.mp4 "Video"

## [Rubric](https://review.udacity.com/#!/rubrics/571/view) Points


---

### Camera Calibration

Computing camera calibration is in the class Camera method compute_camera_calibration.

We start by preparing "object points", which will be the (x, y, z) coordinates of the chessboard corners in the world. We assume that the chessboard is fixed on the (x, y) plane at z=0, such that the object points are the same for each calibration image.  Thus, `objp` is just a replicated array of coordinates, and `objpoints` will be appended with a copy of it every time chessboard corners are successfully detected in a test image; with each successful chessboard detection we append (x, y) position of each corner in the image plane to `imgpoints`. At the next stage `objpoints` and `imgpoints` are used to compute the camera calibration and distortion coefficients - `cv2.calibrateCamera()` function.  The computed camera calibration matrix and distortion coefficients are stored in the class variables mtx and dist. The method undistort_image is responsible for the distortion correction; it passes the precomputed camera calibration matrix and distortion coefficients to cv2.undistort() to obtain the undistorted images. The following is the result of calling method undistort_image for a chess image:

Original image             |  Undistorted image
:-------------------------:|:-------------------------:
![alt text][image1]       |  ![alt text][image2]


### Pipeline (single images)

The image pipeline process_image function invokes the following functions:
* remove distortion;
* threshold the image;
* perform a prespective transform;
* find lane pixels;
* perform a reverse transform of the image with drawn lane lines.

The figure below shows the main stages of the pipeline:

Undistorted image          |  Thresholded image       |  Warped image          |  Detected Lines           
:-------------------------:|:------------------------:|:----------------------:|:-------------------------:
![alt text][image11]       |  ![alt text][image12]    |   ![alt text][image13] |  ![alt text][image14]     
![alt text][image21]       |  ![alt text][image22]    |   ![alt text][image23] |  ![alt text][image24]      
![alt text][image31]       |  ![alt text][image32]    |   ![alt text][image33] |  ![alt text][image34]     
![alt text][image41]       |  ![alt text][image42]    |   ![alt text][image43] |  ![alt text][image44]     

#### 1. Example of a distortion-corrected image.

The correction of distortion was performed using the method undistort_image of the class Camera. Following is the exmaple of an undistorted image:

Original image             |  Undistorted image
:-------------------------:|:-------------------------:
![alt text][image3]       |  ![alt text][image4]

#### 2. Creating a thresholded binary image using color transforms and gradients. 

A combination of color and gradient thresholds was used to generate a binary image. The class ThresholdProcesser is responsible for image thresholding. It has two methods thresholds and color_transform. Thresholds method applies sobel operator to an image in both x and y directions with threshold(10, 150) and filters the image by magnitude and direction of gradient with thresholds (100, 255)  and (0.7, 1.4) respectively. Initially, color_transform method was working only with the saturation channel; values were obtained from converting an image from RGB space to HLS. With this combination of thresholds (gradient and saturation channel), the program could not properly identify the lane lines where there was a sequence of shadow and light frames. Adding thresholding the lightness channel with threshold (150, 255) from LAB color space improved the level of recognition. 

All values for thresholds were obtained manually. Following is the example of a thresholded image:

Original image             |  Thresholded image
:-------------------------:|:-------------------------:
![alt text][image5]       |  ![alt text][image6]

#### 3. Perspective transform.

The code for perspective transform is in the method perspective_transform of the class Camera. This method takes as inputs an image (`img`), as well as source (`src`) and destination (`dst`) points. 

Source and destination points:

| Source        | Destination   | 
|:-------------:|:-------------:| 
| 678, 443      | 919, 0        | 
| 605, 443      | 285, 0      |
| 285, 665      | 285, 665      |
| 1019, 665     | 919, 665      |

We verified that my perspective transform was working as expected by drawing the `src` and `dst` points on a test image and its warped counterpart to verify that the lines appear parallel in the warped image:

Undistorted image          |  Warped image
:-------------------------:|:-------------------------:
![alt text][image7]       |  ![alt text][image8]

#### 4. Identify lane-line pixels.

Class LaneDetector contains the code which identifies lane-line pixels using a histogram method, specifically the method detect_lines_images. After the pixels are identified we use a 2nd order polynomial to fit the lane lines pixels and get the coefficients of the curves.

---

### Pipeline (video)

Here's a [link to the video with detected lines](./project_video.mp4)

Video_pipeline invokes all the steps described for the images plus advanced search for lane lines pixels, calculating the curvature and position of a vehicle.

The video pipeline function process_video uses two instances of class Line, left_line, right_line. For identifying the lane lines pixels it uses the class LaneDetector, specifically method detect_lines_video. This method initially apllies the histogram method (to the first frame) using another method of this class find_lanes_w_histogram. For the following frames search of lane lines pixels is performed using a method search_around_poly, which searchs within the borders of the previously detected lines plus margin. For each frame detect_lines_video performs sanity check; it checks that the the difference between first coefficients of the lines is less than 0.1 - by these means we check that lane lines are roughly parallel. If not we plot the values from the previous frame and for the next one we strat again with the histogram method. This method also performs averaging between last n-frames to make the lane lines smother.  

To calculate the radius of curvature we use the following formula:

![alt text][image9]

Before applying the formula, the method measure_curvature of the class LaneDetector recomputes the coefficients using method fit_poly_for_lines in respect to the meters/pixels ration.

```python
ym = 30/720 # meters per pixel in y dimension
xm = 3.7/700 # meters per pixel in x dimension
        
left_fit_cr, right_fit_cr = self.fit_poly_for_lines(self.left_lane.allx*xm, 
                                                    self.left_lane.ally*ym, 
                                                    self.right_lane.allx*xm, 
                                                    self.right_lane.ally*ym)
             
```
To compute the position of the vehicle with respect to center we first take the center of the image in x-axis and compute its coordinate in a warped image using defined src and dst points. For each frame we compute the center of the detected lane lines via using method find_lane_center : 

```python
# h is the height of an image
left_lane_c = left_fit[0]*h**2 + left_fit[1]*h + left_fit[2] # compute x-coordinate for left line
right_lane_c = right_fit[0]*h**2 + right_fit[1]*h + right_fit[2] # compute x-coordinate for right line
return left_lane_c + (right_lane_c - left_lane_c) /2 # find the center between two lines 
             
```
In a pipline we take the difference between this two points. 

---
