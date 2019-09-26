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


[image11]: ./output_images/undistort_test3.jpg
[image21]: ./output_images/undistort_straight_lines1.jpg
[image31]: ./output_images/undistort_test1.jpg
[image41]: ./output_images/undistort_test5.jpg

[image12]: ./output_images/binary_test3.jpg
[image22]: ./output_images/binary_straight_lines1.jpg
[image32]: ./output_images/binary_test1.jpg
[image42]: ./output_images/binary_test5.jpg

[image13]: ./output_images/warped_test3.jpg
[image23]: ./output_images/warped_straight_lines1.jpg
[image33]: ./output_images/warped_test1.jpg
[image43]: ./output_images/warped_test5.jpg

[image14]: ./output_images/warped_lines_test3.jpg
[image24]: ./output_images/warped_lines_straight_lines1.jpg
[image34]: ./output_images/warped_lines_test1.jpg
[image44]: ./output_images/warped_lines_test5.jpg

[image15]: ./output_images/final_test3.jpg
[image25]: ./output_images/final_lines1.jpg
[image35]: ./output_images/final_test1.jpg
[image45]: ./output_images/final_test5.jpg



[image2]: ./test_images/test1.jpg "Road Transformed"
[image3]: ./examples/binary_combo_example.jpg "Binary Example"
[image4]: ./examples/warped_straight_lines.jpg "Warp Example"
[image5]: ./examples/color_fit_lines.jpg "Fit Visual"
[image6]: ./examples/example_output.jpg "Output"
[video1]: ./project_video.mp4 "Video"

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

Undistorted image          |  Thresholded image       |  Warped image          |  Detected Lines           |  Final Image
:-------------------------:|:------------------------:|:----------------------:|:-----------------------__:|:-------------------------:
![image11]                 |  ![image12]              |   ![image13]           |  ![image14]               |  ![image15]  

#### 1. Provide an example of a distortion-corrected image.

To demonstrate this step, I will describe how I apply the distortion correction to one of the test images like this one:
![alt text][image2]

#### 2. Describe how (and identify where in your code) you used color transforms, gradients or other methods to create a thresholded binary image.  Provide an example of a binary image result.

I used a combination of color and gradient thresholds to generate a binary image (thresholding steps at lines # through # in `another_file.py`).  Here's an example of my output for this step.  (note: this is not actually from one of the test images)

![alt text][image3]

#### 3. Describe how (and identify where in your code) you performed a perspective transform and provide an example of a transformed image.

The code for my perspective transform includes a function called `warper()`, which appears in lines 1 through 8 in the file `example.py` (output_images/examples/example.py) (or, for example, in the 3rd code cell of the IPython notebook).  The `warper()` function takes as inputs an image (`img`), as well as source (`src`) and destination (`dst`) points.  I chose the hardcode the source and destination points in the following manner:

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

This resulted in the following source and destination points:

| Source        | Destination   | 
|:-------------:|:-------------:| 
| 585, 460      | 320, 0        | 
| 203, 720      | 320, 720      |
| 1127, 720     | 960, 720      |
| 695, 460      | 960, 0        |

I verified that my perspective transform was working as expected by drawing the `src` and `dst` points onto a test image and its warped counterpart to verify that the lines appear parallel in the warped image.

![alt text][image4]

#### 4. Describe how (and identify where in your code) you identified lane-line pixels and fit their positions with a polynomial?

Then I did some other stuff and fit my lane lines with a 2nd order polynomial kinda like this:

![alt text][image5]

#### 5. Describe how (and identify where in your code) you calculated the radius of curvature of the lane and the position of the vehicle with respect to center.

I did this in lines # through # in my code in `my_other_file.py`

#### 6. Provide an example image of your result plotted back down onto the road such that the lane area is identified clearly.

I implemented this step in lines # through # in my code in `yet_another_file.py` in the function `map_lane()`.  Here is an example of my result on a test image:

![alt text][image6]

---

### Pipeline (video)

#### 1. Provide a link to your final video output.  Your pipeline should perform reasonably well on the entire project video (wobbly lines are ok but no catastrophic failures that would cause the car to drive off the road!).

Here's a [link to my video result](./project_video.mp4)

---

### Discussion

#### 1. Briefly discuss any problems / issues you faced in your implementation of this project.  Where will your pipeline likely fail?  What could you do to make it more robust?

Here I'll talk about the approach I took, what techniques I used, what worked and why, where the pipeline might fail and how I might improve it if I were going to pursue this project further.  
