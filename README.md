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
[image6]: ./examples/example_output.jpg "Output"
[video1]: ./video_output/project_video.mp4 "Video"

## [Rubric](https://review.udacity.com/#!/rubrics/571/view) Points


---

### Camera Calibration

The class Camera is responsible for computing camera calibration. The code for this step is contained in the method compute_camera_calibration.

I start by preparing "object points", which will be the (x, y, z) coordinates of the chessboard corners in the world. Here I am assuming the chessboard is fixed on the (x, y) plane at z=0, such that the object points are the same for each calibration image.  Thus, `objp` is just a replicated array of coordinates, and `objpoints` will be appended with a copy of it every time I successfully detect all chessboard corners in a test image.  `imgpoints` will be appended with the (x, y) pixel position of each of the corners in the image plane with each successful chessboard detection.  

I then used the output `objpoints` and `imgpoints` to compute the camera calibration and distortion coefficients using the `cv2.calibrateCamera()` function.  The computed camera calibration matrix and distortion coefficients are stored in the locatl class variables self.mtx and self.dist. The method undistort_image is responsible for the distortion correction; it passes the precomputed camera calibration matrix and distortion coefficients to cv2.undistort() to obtain undistorted images. The following is the result of calling method undistort_image for a chess image:

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

#### 1. Provide an example of a distortion-corrected image.

The correction of distortion was performed using the method undistort_image of the class Camera. Following is the exmaple of an undistorted image:

Original image             |  Undistorted image
:-------------------------:|:-------------------------:
![alt text][image3]       |  ![alt text][image4]

#### 2. Describe how (and identify where in your code) you used color transforms, gradients or other methods to create a thresholded binary image.  Provide an example of a binary image result.

I used a combination of color and gradient thresholds to generate a binary image. The class ThresholdProcesser is responsible for image thresholding. It has two methods thresholds and color_transform. Thresholds method applies sobel operator to an image in both x and y directions with threshold(10, 150) and filters the image by magnitude and direction of gradient with thresholds (100, 255)  and (0.7, 1.4) respectively. Initially, color_transform method was working only with the saturation channel; values were obtained from converting an image from RGB space to HLS. With this combination of thresholds (gradient and saturation channel), the program could not properly identify the lane lines where there was a sequence of shadow and light frames. Adding thresholding the lightness channel with threshold (150, 255) from LAB color space improved the level of recognition. 

All values for thresholds were obtained manually. Following is the example of a thresholded image:

Original image             |  Thresholded image
:-------------------------:|:-------------------------:
![alt text][image5]       |  ![alt text][image6]

#### 3. Describe how (and identify where in your code) you performed a perspective transform and provide an example of a transformed image.

The code for my perspective transform is in the method perspective_transform of the class Camera. This method takes as inputs an image (`img`), as well as source (`src`) and destination (`dst`) points. I chose the hardcode the source and destination points in the following manner:

This resulted in the following source and destination points:

| Source        | Destination   | 
|:-------------:|:-------------:| 
| 585, 460      | 320, 0        | 
| 203, 720      | 320, 720      |
| 1127, 720     | 960, 720      |
| 695, 460      | 960, 0        |

I verified that my perspective transform was working as expected by drawing the `src` and `dst` points onto a test image and its warped counterpart to verify that the lines appear parallel in the warped image.

straight_lines2.jpg


#### 4. Describe how (and identify where in your code) you identified lane-line pixels and fit their positions with a polynomial?

Then I did some other stuff and fit my lane lines with a 2nd order polynomial kinda like this:

![alt text][image5]

#### 5. Describe how (and identify where in your code) you calculated the radius of curvature of the lane and the position of the vehicle with respect to center.

I did this in lines # through # in my code in `my_other_file.py`

#### 6. Provide an example image of your result plotted back down onto the road such that the lane area is identified clearly.

I implemented this step in lines # through # in my code in `yet_another_file.py` in the function `map_lane()`.  Here is an example of my result on a test image:

![alt text][image6]


```python
src = np.float32(
    [[(img_size[0] / 2) - 55, img_size[1] / 2 + 100],
    [((img_size[0] / 6) - 10), img_size[1]],
    [(img_size[0] * 5 / 6) + 60, img_size[1]],
    [(img_size[0] / 2 + 55), img_size[1] / 2 + 100]])
dst = np.float32(
    [[(img_size[0] / 4), 0],
    [(img_size[0] / 4), img_size[1]],
    [(img_size[0] * 3 / 4), img_size[1]],
    [(img_size[0] * 3 / 4), 0]])
```
---

### Pipeline (video)

#### 1. Provide a link to your final video output.  Your pipeline should perform reasonably well on the entire project video (wobbly lines are ok but no catastrophic failures that would cause the car to drive off the road!).

Here's a [link to my video result](./project_video.mp4)

---

### Discussion

#### 1. Briefly discuss any problems / issues you faced in your implementation of this project.  Where will your pipeline likely fail?  What could you do to make it more robust?

Here I'll talk about the approach I took, what techniques I used, what worked and why, where the pipeline might fail and how I might improve it if I were going to pursue this project further.  
