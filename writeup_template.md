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

[//]: # (Image References)

[image1]: ./output_images/undistorted.png "Undistorted"
[image2]: ./test_images/test5.jpg "Road Transformed"
[image3]: ./output_images/binary_combo_example.png "Binary Example"
[image4]: ./output_images/warped_straight_lines.png "Warp Example"
[image5]: ./output_images/color_fit_lines.png "Fit Visual"
[image6]: ./output_images/example_output.png "Output"
[video1]: ./output_images/project_video.mp4 "Video"

## [Rubric](https://review.udacity.com/#!/rubrics/571/view) Points

### Here I will consider the rubric points individually and describe how I addressed each point in my implementation.  

---

### Writeup / README

#### 1. Provide a Writeup / README that includes all the rubric points and how you addressed each one.  You can submit your writeup as markdown or pdf.  [Here](https://github.com/udacity/CarND-Advanced-Lane-Lines/blob/master/writeup_template.md) is a template writeup for this project you can use as a guide and a starting point.  

You're reading it!

### Camera Calibration

#### 1. Briefly state how you computed the camera matrix and distortion coefficients. Provide an example of a distortion corrected calibration image.

The code for this step is contained in the first code cell of the IPython notebook located in "./examples/example.ipynb" (or in lines # through # of the file called `some_file.py`).  

I start by preparing "object points", which will be the (x, y, z) coordinates of the chessboard corners in the world. Here I am assuming the chessboard is fixed on the (x, y) plane at z=0, such that the object points are the same for each calibration image.  Thus, `objp` is just a replicated array of coordinates, and `objpoints` will be appended with a copy of it every time I successfully detect all chessboard corners in a test image.  `imgpoints` will be appended with the (x, y) pixel position of each of the corners in the image plane with each successful chessboard detection.  

I then used the output `objpoints` and `imgpoints` to compute the camera calibration and distortion coefficients using the `cv2.calibrateCamera()` function.  I applied this distortion correction to the test image using the `cv2.undistort()` function and obtained this result: 

![alt text][image1]

### Pipeline (single images)

#### 1. Provide an example of a distortion-corrected image.

To demonstrate this step, I will describe how I apply the distortion correction to one of the test images like this one:
![alt text][image2]

I wrote a function "undistortImage" which takes original image, object points and image points from previous step at 2nd Cell in example.ipynb. This function returns an undistorted image.

#### 2. Describe how (and identify where in your code) you used color transforms, gradients or other methods to create a thresholded binary image.  Provide an example of a binary image result.

A distortion-corrected image was firstly transformed to grayscale. 

Then I used Sobel Operator to get x-direction gradient. This sobel-x was then preprocessed as absolute. After that it was scaled again. With threshold(20, 100) scaled sobel-x was changed to a binary image.

To use color thresholds, I transfered the distortion-corrected image from RGB space to HLS space. Then I extracted S channel and H channel. Color threshold (170, 255) was used to S channel and (70, 100) to H channel. Then these two binary images were combined to generate a binary image of color. 

After get both binary images of color and gradient, I again combined them to generate a binary image (thresholding steps through 5th code cell to 7th code cell in `example.ipynb`).  Here's an example of my output for this step.  (note: this is not actually from one of the test images)

![alt text][image3]

#### 3. Describe how (and identify where in your code) you performed a perspective transform and provide an example of a transformed image.

The code for my perspective transform is a function called `cv2.getPerspectiveTransform()`, which appears in the 1st code cell of the example.ipynb.  The `cv2.getPerspectiveTransform()` function takes as inputs as source (`src`) and destination (`dst`) points then returns transformation matrix. Manually I chose the source and destination points. This resulted in the following source and destination points:

| Source        | Destination   | 
|:-------------:|:-------------:| 
| 585, 457      | 300, 0        | 
| 260, 690      | 300, 720      |
| 1060, 690     | 980, 720      |
| 700, 457      | 980, 0        |

After calculating out transformation matrix I stored it in pickle object via Pickle module. Whenever I want to use it in later steps I can use pickle module to load this matrix again.

I verified that my perspective transform was working as expected by drawing the `src` and `dst` points onto a test image and its warped counterpart to verify that the lines appear parallel in the warped image.

![alt text][image4]

#### 4. Describe how (and identify where in your code) you identified lane-line pixels and fit their positions with a polynomial?

The warped image was transformed to grayscale. Applying color and gradient thresholds above to this warped grayscale I generated a binary warped image. This warped grayscale was divided into left and right part at middle point. I calculated the max value of each part along x position and got the x position of max value. This x position was used as starting point of lane lines.

Then sliding windows were used to find all nonzero value pixels in the windows. The pixels' x and y values were passed to fit the lane lines using quadratic function: f(x) = A*y*y + B*y + C. After polynomial fitting A, B and C were obtained.

A, B and C coefficients were again used to each y position to calculate corresponding x position. In this way the lane lines were generated.

![alt text][image5]

#### 5. Describe how (and identify where in your code) you calculated the radius of curvature of the lane and the position of the vehicle with respect to center.

I did this in code cell 7 through 8 in my code in `example.ipynb`. 

The function calc_Curverad() takes fit coefficients A, B and chosen max y value(in this project is 719) from above step to calculate curvature using formula: ((1 + (2*A*y +B)**2)**1.5) / |2*A|

In order to converse curve rad to distance(meter) from center function converse_CurveradToMeter() was used, which takes fitting lane lines' x positions and meters per pixel, max y value to get distance.

#### 6. Provide an example image of your result plotted back down onto the road such that the lane area is identified clearly.

I implemented this step in code cell 18 in my code in `example.ipynb` in the function `process_image()`.  Here is an example of my result on a test image:

![alt text][image6]

---

### Pipeline (video)

#### 1. Provide a link to your final video output.  Your pipeline should perform reasonably well on the entire project video (wobbly lines are ok but no catastrophic failures that would cause the car to drive off the road!).

Here's a [link to my video result](./project_video.mp4)

---

### Discussion

#### 1. Briefly discuss any problems / issues you faced in your implementation of this project.  Where will your pipeline likely fail?  What could you do to make it more robust?

Here I'll talk about the approach I took, what techniques I used, what worked and why, where the pipeline might fail and how I might improve it if I were going to pursue this project further.  

At first I just used sobel x combined with S-channel to get a binary image which made obvious failure in some situations that road surface is bad, shadow covers lane lines, unclear lane lines etc. Then I tried to add H channel to extract yellow lane line with appropriate threshold. This addition increase the detection accuracy in facing the situations I mentioned above.
